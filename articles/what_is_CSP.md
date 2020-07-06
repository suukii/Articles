# CSP 内容安全策略

### 什么是 CSP

一个网页可以跟各种网络资源打交道，比如请求 JS 或者 CSS 文件，发送 AJAX 请求，加载图片/字体文件等，在这些过程中很有可能会发生 XSS 或者代码注入攻击等。

为了防御攻击，就有了 CSP(Content Security Policy)，它是一层安全保障，用来约束浏览器只能从白名单站点下载 scripts(包括 inline scripts)、styles、媒体文件等资源。

P.S. 虽然 CSP 可以减少 XSS 攻击的风险，但它不能完全替代校验用户输入以及编码输出这些安全性检查操作

### 如何开启 CSP

开启 CSP 有两个方法:

- 通过设置 HTTP 响应头部(Content-Security-Policy)
- 通过在 HTML 文档中使用 `<meta>` 标签并设置属性 `http-equiv="Content-Security-Policy"`

CSP 设置内容：

不管使用以上哪种方法，CSP 的内容设置都是一样的，一个 CSP 设置包含了一系列允许和禁止的操作，写法上，它是由一系列“指令+源”(directive + resource)组成的。指令和源中间以空格分隔，一个指令可以指定多个源，每个源也以空格分隔。每个“指令+源”组合中间使用 `;` 隔开。

通过指令我们可以控制网站的一些行为，以下是一些例子

- `script-src` 指定 js 文件的可信任源

`script-src 'self' *.mozilla.com`: `'self'` 是一个关键字，表示允许从当前域(不包括子域)下载 js 文件，`*.mozilla.com` 表示来自 mozilla.com 的 js 文件也可以下载

- `img-src` 指定 img 和 favicon 的可信任源

- `default-src` fallback, 如果要请求的文件类型没有相应的指令，那就使用这个指令的设置

`default-src 'none'`: `'none'` 表示不信任来自任何站点的资源

[w3.org 完整指令列表](https://www.w3.org/TR/CSP/#csp-directives)

### CSP 的两种模式

以上所说的是 enforce 模式的 CSP，也就是 `Content-Security-Policy` 头部里面指定的的指令都会被执行。CSP 还有另外一个模式 report 模式，在这种模式下，指令不会被执行，也就是，网站该下载什么文件照样下载，即使下载源不在白名单里，不过如果网站发生了违反 CSP 设置的行为的话，比如向非白名单源请求 js 文件，浏览器就会向指定的服务端发送一份报告。

开启 report 模式只需要使用 `Content-Security-Policy-Report-Only` 头部即可，内容设置的规则和 `Content-Security-Policy` 是一样的，而且，这两个头部是可以同时存在，互不干扰的。

另外，在 enforce 模式下，如果 CSP 设置有被违反的话，默认是不会向服务端发送报告的，我们可以通过 `report-uri example.com` 指令来开启这个功能。
