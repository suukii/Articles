# 如何优化图片资源

## 使用 imagemin 来压缩图片

imagemin 是一个图片压缩工具，它提供 [CLI](https://www.npmjs.com/package/imagemin-cli) 和 [npm module](https://www.npmjs.com/package/imagemin) 两种使用方式，一般推荐使用 npm module 形式，因为它能提供更多配置选项，也可以嵌入到 `webpack` 等工具的打包流程中，另外还有一系列配套的插件来处理不同格式的图片文件。

### CLI

#### 安装

`npm install --global imagemin-cli`

#### 使用

- `imagemin images/* --out-dir=dist`: 将 `images` 文件夹中的所有图片进行压缩并输出到 `dist` 文件夹中。

- `imagemin --help`: 查看所有配置选项。

```
Usage
  $ imagemin <path|glob> ... --out-dir=build [--plugin=<name> ...]
  $ imagemin <file> > <output>
  $ cat <file> | imagemin > <output>

Options
  --plugin, -p   Override the default plugins
  --out-dir, -o  Output directory

Examples
  $ imagemin images/* --out-dir=build
  $ imagemin foo.png > foo-optimized.png
  $ cat foo.png | imagemin > foo-optimized.png
  $ imagemin foo.png --plugin=pngquant > foo-optimized.png
  $ imagemin foo.png --plugin.pngquant.quality={0.1,0.2} > foo-optimized.png
  $ imagemin foo.png --plugin.webp.quality=95 --plugin.webp.preset=icon > foo-icon.webp
```

### npm module

#### 安装

`npm i -D imagemin`

#### 使用

- Node script
  e.g. 使用 `imagemin-mozjpeg` 插件来压缩 `jpeg` 图片

```js
const imagemin = require('imagemin')
const imageminMozjpeg = require('imagemin-mozjpeg')

imagemin(
  // 指定需要压缩的图片文件路径
  ['images/*.jpg'],
  {
    // 指定压缩后图片文件的输出路径
    destination: 'dist/images',
    // 指定负责处理图片的插件
    plugins: [imageminMozjpeg({ quality: 50 })]
  }
)
```

- 打包工具
  - [x] [教程 - 如何在 webpack 中使用 imagemin](https://web.dev/codelab-imagemin-webpack/)
  - [ ] [教程 - 如何在 gulp 中使用 imagemin](https://web.dev/codelab-imagemin-gulp/)
  - [ ] [教程 - 如何在 grunt 中使用 imagemin](https://web.dev/codelab-imagemin-grunt/)

### 在 webpack 中使用 imagemin

e.g. webpack.config.js

```js
// 引入 imagemin-webpack-plugin
const ImageminPlugin = require('imagemin-webpack-plugin')

// 如果想要自己指定处理图片的插件，也需要另外引入
const imageminMozjpeg = require('imagemin-mozjpeg')

module.exports = {
  plugins: [
    ImageminPlugin({
      // pngquant 是 imagemin-webpack-plugin 使用的默认插件之一
      // 所以可以直接指定配置选项
      pngquant: { quality: [0.5, 0.5] },

      // imagemin-mozjpeg 不是 imagemin-webpack-plugin 的默认插件
      // 所以需要额外引入并加入到 plugins 数组中
      plugins: [imageminMozjpeg({ quality: 50 })]
    })
  ]
}
```

### 常用的插件

| 图片格式 | 有损                                                                 | 无损                                                                 |
| -------- | -------------------------------------------------------------------- | -------------------------------------------------------------------- |
| JPEG     | [imagemin-mozjpeg](https://www.npmjs.com/package/imagemin-mozjpeg)   | [imagemin-jpegtran](https://www.npmjs.com/package/imagemin-jpegtran) |
| PNG      | [imagemin-pngquant](https://www.npmjs.com/package/imagemin-pngquant) | [imagemin-optipng](https://www.npmjs.com/package/imagemin-optipng)   |
| GIF      | [imagemin-giflossy](https://www.npmjs.com/package/imagemin-giflossy) | [imagemin-gifsicle](https://www.npmjs.com/package/imagemin-gifsicle) |
| SVG      | [imagemin-svgo](https://www.npmjs.com/package/imagemin-svgo)         |                                                                      |
| WebP     | [imagemin-webp](https://www.npmjs.com/package/imagemin-webp)         |                                                                      |

> tips: 如果一个插件提供订制图片质量的配置选项，那它就是一个有损压缩的插件。

## 用视频替换 GIF

> 有些 GIF 的体积可能会很大，将它们转换成视频格式可以节省用户带宽。

FFmpeg 是一个音频/视频转换工具，使用方法如下：

#### 安装

- 去[官网](https://www.ffmpeg.org/)上下载，官网也提供了很详细的[使用文档](https://www.ffmpeg.org/documentation.html)。
- 使用 [npm module](https://www.npmjs.com/package/ffmpeg)，这种方法也需要先安装 FFmpeg。

#### 使用

- [x] [博客 - 将 GIF 转换成视频](https://web.dev/replace-gifs-with-videos/)
- [ ] [教程 - 将 GIF 转换成视频](https://gif-to-video.glitch.me/)

#### 替换

将 GIF 文件转换成视频之后，还需要在文档中进行替换。

GIF 有三个特点：

- 自动播放
- 无限循环
- 静音

这些都可以通过 `video` 标签的属性来设置：

```html
<video autoplay loop muted playsinline></video>
```

还可以通过 `<source>` 元素来提供 fallback：

```html
<video autoplay loop muted playsinline>
  <source src="my-animation.webm" type="video/webm" />
  <source src="my-animation.mp4" type="video/mp4" />
</video>
```

## 提供响应式图片

给不同设备提供不同尺寸的图片，比较常用的两个用来生成不同尺寸图片的工具是 sharp 和 ImageMagick CLI：

### sharp

sharp 是一个 [npm 包](https://www.npmjs.com/package/sharp)

#### 安装

`npm i -D sharp`

#### 使用

Node script

```js
const sharp = require('sharp')
const fs = require('fs')
const directory = './images'

fs.readdirSync(directory).forEach((file) => {
  const format = /.*\.([a-zA-Z]+)/.exec(file)[1]
  sharp(`${directory}/${file}`)
    .resize(200, 100) // width, height
    .toFile(`${directory}/${file}-small.${format}`)
})
```

### ImageMagick CLI

ImageMagick 是一个 CLI 工具

#### 安装

去[官网](https://www.imagemagick.org/script/index.php)下载安装包。

#### 使用

- 命令行，不过有平台兼容性问题。

例子 1 `convert -resize 33% logo.jpg logo-small.jpg`：将 `logo.jpg` 转换成原图的三分一大小。

例子 2 `convert logo.jpg -resize 300x200 logo-small.jpg`：将图片尺寸调整为 300x200

- 也可以下载 [imagemagick-cli](https://www.npmjs.com/package/imagemagick-cli) npm 包，然后通过 Node 来调用，这个包解决了操作系统兼容性的问题。

Node script

```js
const imagemagickCli = require('imagemagick-cli')
imagemagickCli.exec('convert -version')
```

> 每张图片应该有多少种尺寸？这个问题没有统一答案，不过比较常见的是 3-5 种。

### 在文档中设置

将图片转换成不同的尺寸后，还需要在 HTML 中为不同屏幕尺寸指定不同版本的图片，然后浏览器就会根据实际情况来选择加载哪张图片。

```html
<img src="logo.jpg" srcset="logo-small.jpg 480w, logo-large.jpg 1080w" sizes="50vw" />
```

- `src` 是为了兼容那些不支持 `srcset` 和 `sizes` 属性的浏览器。
- `logo-small.jpg 480w` 表示 `logo-small.jpg` 这张图的宽度是 `480px`，这样浏览器就不用等把图片下载完后才知道图片的尺寸。注意到图片宽度的单位是 `px`，但在设置时要写成 `w`，关于 [width descriptor](https://www.w3.org/TR/html5/semantics-embedded-content.html#width-descriptor)。
- `sizes` 属性告诉浏览器图片在页面上显示时的宽度，但是设置这个属性对图片的显示效果并没有影响(我们还是得设置 CSS)，这个值只是用来帮助浏览器决定应该加载哪张图片，如果没有这个属性，浏览器就会加载 `src` 属性中指定的图片。`sizes` 的有效值包括 `100px`, `33vw`, `20em`, `calc(50vw-10px)`，但是百分比 `25%` 是不合法的。

> srcset 和 sizes 属性是配合使用的。

- [ ] [教程 - 如何设置多个 sizes 值](https://web.dev/codelab-specifying-multiple-slot-widths/)
- [ ] [概念 - Art Direction](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images#Art_direction)
- [ ] [教程 - Art Direction](https://web.dev/codelab-art-direction)

> art direction: 在不同大小的屏幕中显示**内容不同**的图片，而不仅仅是**尺寸不同**的图片。

## 使用 WebP 图片

两个常用的将图片转换成 WebP 的工具是：[cwebp CLI 工具](https://developers.google.com/speed/webp/docs/using) 和 [Imagemin WebP 插件](https://github.com/imagemin/imagemin-webp)，一般推荐使用 Imagemin WebP 插件，因为可以整合到打包工具中。

### cwebp CLI 工具

#### 安装

[下载地址](https://storage.googleapis.com/downloads.webmproject.org/releases/webp/index.html)

#### 使用

转换单张图片，使用默认配置：

`cwebp images/flower.jpg -o images/flower.webp`

转换单张图片，设置 quality 为 50：

`cwebp -q 50 images/flower.jpg -o images/flower.webp`

转换指定文件夹中的所有图片：

`for file in images/*; do cwebp "$file" -o "${file%.*}.webp"; done`

### Imagemin WebP 插件

#### 安装

`npm install imagemin-webp`

#### 使用

- Node script:

```js
const imagemin = require('imagemin')
const imageminWebp = require('imagemin-webp)

imagemin(
  ['images/*.{jpg,png}'],
  {
    destination: 'build/images',
    plugins: [
      imageminWebp({quality: 50})
    ]
  }
)
```

- 打包工具
  - [ ] [例子 - 在 webpack 中使用 Imagemin WebP](https://glitch.com/~webp-webpack/)
  - [ ] [例子 - 在 gulp 中使用 Imagemin WebP](https://glitch.com/~webp-gulp/)
  - [ ] [例子 - 在 grunt 中使用 Imagemin WebP](https://glitch.com/~webp-grunt/)

### 浏览器兼容

如果需要兼容不支持 WebP 的[浏览器](https://caniuse.com/#search=webp)的话，还需要提供 fallback 文件。

```html
<picture>
  <source type="image/webp" srcset="flower.webp" />
  <source type="image/jpeg" srcset="flower.jpg" />
  <img src="flower.jpg" alt="" />
</picture>
```

- `<picture>` 中可以指定多个 `<source>`，按优先级排列。
- `<img>` 是为了兼容不支持 `<picture>` 标签的浏览器。

## 使用 CDN

- [Use image CDNs to optimize images](https://web.dev/image-cdns/)
