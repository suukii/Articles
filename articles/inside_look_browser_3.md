# 深入理解现代浏览器 - 渲染器进程

渲染器进程负责一个标签页里的所有工作，它的主要工作就是把 HTML, CSS, JS 结合起来变成一个可交互的网页。在渲染器进程中，主要是以下几个线程在工作着：

- 主线程(main thread)：负责运行客户端的大部分代码。
- 工作线程(worker thread)：如果网页应用中有 web worker 或者 service worker 的话，这些 JS 代码会在工作线程中运行。
- 栅格化线程(raster thread)：负责栅格化图层，也就是把网页图层转换成屏幕上的像素。
- 合成器线程(compositor thread)：负责把已经完成栅格化的图层合成页面。

页面渲染过程包含了以下几个步骤。

## 第一步：解析 HTML

- 构建 DOM：当渲染器进程从浏览器进程那里收到提交导航的信号并开始接收 HTML 数据流时，渲染器进程中的主线程就开始了解析 HTML 字符串并把它转化成一个 DOM 树的工作。

- 加载子资源：在解析 HTML 文档时，解析器会不时遇到 `<img>` `<link>` `<script>` 等标签，这些外部资源需要通过网络下载或者可以从缓存中获取。当然，主线程可以在解析到这些标签的时候停下来去发送请求，不过这样就太浪费时间了。为了提高效率，一个“预加载扫描器”(preload scanner)会同时运行，当主线程解析到以上标签时，预加载扫描器就负责给浏览器进程中的网络线程发送请求。

- JS 会阻塞 HTML 解析：当 HTML 解析器碰到 `<script>` 标签时，它会停下来，等到 JS 代码加载、解析、运行结束后，再继续解析 HTML。这是因为在 JS 代码中开发者可以使用 `document.write()` 之类的来修改 DOM 树的结构，所以在 JS 代码执行完成之前，继续往下解析 HTML 是没有意义的。

- 修改资源加载的时机：如果你的 JS 代码里面没有修改 DOM 的操作，那我们可以通过给 `<script>` 标签加上 `async` 或者 `defer` 属性，告诉浏览器这些资源可以异步加载，这样解析器就可以不被打断继续解析 HTML。如果必要的话，还可以使用 JS 模块(JS 模块默认是异步加载的)。

## 第二步：计算样式

生成 DOM 树之后，主线程会去解析 CSS，并根据选择器、优先级等来计算出每一个 DOM 节点的最终样式(computed style)。

就算你的代码里没有任何 CSS，这一步也是不可省的，因为浏览器本身会提供一份默认的 CSS 样式表。

## 第三步：布局

接下来主线程会遍历整个 DOM 树和计算样式，生成一个布局树(layout tree)，布局树上会记录每个节点在页面上的横/纵坐标和盒子大小这些信息。

布局树和 DOM 树在结构上很相似，不过布局树只关心在页面上显示的内容：

- `display: none;` 的元素不会在布局树上(不过 `visibility: hidden;` 的元素却在布局树上)。
- 伪元素如 `::before` 虽然不在 DOM 树上，却在布局树上。

## 第四步：绘制

在这一步主线程会遍历布局树并生成一个绘制记录(paint record)，根据 z-index 属性等来决定并记录元素绘制的先后顺序，如果两个元素有重合的部分，先绘制的元素就会被后绘制的元素覆盖。

## 第五步：划分图层

现在我们知道了整个 HTML 文档的结构，每个元素的样式、横/纵坐标、盒子大小，以及绘制的顺序，接下来就需要把这些信息转化成屏幕上的像素了，这个转化的过程叫栅格化(Rastering)。

栅格化的实现方式有两种：

1. Chrome 以前用到的老办法，先把视口(viewport)中的一部分网页栅格化显示出来，当页面发生滚动时，把已经栅格化的部分向上/下移动，然后对空白的部分进行栅格化填充页面。

2. 更成熟的解决方法是合成(compositoring)，也是 Chrome 现在采用的方式。先把页面的各个部分分成不同的图层(layer)，然后分别栅格化这些图层，接着在一个独立的线程中(合成器线程)把这些栅格化的图层组合成页面。当页面发生滚动时，因为各个图层都是已经栅格化了的，要做的工作就只是用图层重新合成一个新的页面视图而已。

- 在这一步，主线程会遍历整个布局树，决定哪些元素放到哪个图层，然后合成图层树(layer tree)。

- 开发者还可以通过 CSS 的 `will-change` 属性来告诉浏览器哪些元素应该放在独立的图层中。

- 不要想着把所有元素都放在单独的图层中，因为合成过多数目的图层可以会比老的栅格化方法还要慢呢。

P.S. 可以在 DevTools 的 Layers panel 看到当前网页的所有图层，如果 Layers panel 没有打开，可以通过 `Ctrl+P` 搜索打开

## 最后一步：合成

- 当图层树创建完成，而且元素的绘制顺序也已确定了，主线程就会向合成器线程传送这些信息。

- 接着合成器线程就会把图层分发到不同的栅格化线程中去，如果图层太大，合成器线程还会先把图层分成小片(tile)，然后分给不同的栅格化线程去处理。

- 合成器线程还可以安排不同栅格化线程的优先级，所以视口(viewport)及其附近的图层会先进行栅格化处理。

- 栅格化线程把接收的图层或者图层小片栅格化并存储到 GPU 的内存中，这些图层小片保存着它们在内存中的位置以及它们在页面上应该在什么位置。

- 完成图层栅格化后，合成器线程就会根据图层小片的信息，也叫绘制方块(draw quads)，来创建一个合成器帧(compositor frame)，合成器帧只是多个绘制方块拼成的页面中的一帧。

- 创建好的合成器帧会通过 IPC 传送到浏览器进程，接着再传送到 GPU 进程，由 GPU 进程把它画到屏幕上。

- 如果发生滚动事件，合成器线程就会再创建新的合成器帧并发送给 GPU 进程。

## 渲染是一个耗时的过程

从 HTML 解析到绘制的这个过程中，我们注意到了每一步的操作都需要依赖上一步的结果，所以如果布局树发生了变化，那绘制顺序也需要重新计算了。

如果页面上有动画的话，那浏览器就需要在每一帧都完成一次上述过程，否则动画看起来就不太流畅，而现在的显示屏一般是每秒刷新 60 次(60fps)，也就是浏览器每秒要重复 60 次这个过程。

不过，即使浏览器的渲染速度能跟上屏幕的刷新速度，动画的流畅度也不能保证。因为这个渲染过程的所有计算都是在主线程中完成的，而主线程同时也负责 JS 代码的执行，JS 代码可能会阻塞这些渲染计算，导致动画卡帧。

想要避免 JS 阻塞渲染这个问题的话，我们可以使用 `requestAnimationFrame()` 来把 JS 代码分成小块并分到每个帧中去运行，或者把 JS 从主线程中抽出来，放到 web worker 中去运行。

当然，还有更好的实现动画的方式，注意在完整的渲染过程中，合成操作是在独立的合成器线程中完成的，所以，如果动画只改变了图层相关的 CSS 属性，也就是 `transform: translate(x, y);`, `transform: scale(n);`, `transform: rotate(ndeg)`, `opacity: 1;` 之类的，页面就可以在不需要主线程参与的情况下更新。

但是，如果动画涉及了绘制相关的属性，如 `color`, `visibility` 等，页面更新需要从绘制开始往下走流程，更麻烦的是，如果动画涉及了布局相关的属性，如 `width`, `float`, `display` 等，更新就得从重新生成布局树开始了。

## 小结

这一节拆解渲染流水线(rendering pipeline)，详细介绍了渲染过程从解析到合成的各个步骤。