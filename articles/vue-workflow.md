# Vue 项目开发流程

## 规范 Git 提交

可以使用插件来规范

#### 安装插件

`npm i -D commitizen conventional-changelog cz-conventional-changelog`

- `comiitizen` 会提供一个交互界面来规范 commit 信息的填写
- `conventional-changelog` 和 `cz-conventional-changelog` 用来生成提交记录

#### 配置依赖路径

package.json

```json
{
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
}
```

#### git commit

- 方法 1：

`npx git-cz`

- 方法 2：

在 `package.json` 中配置命令

```json
{
  "script": {
    "commit": "npx git-cz"
  }
}
```

#### 生成 changelog

先在全局安装 cli：
`npm i -g conventional-changelog-cli`

增加一个 npm 命令：
`"genlog": "conventional-changelog -p angular -i .github/CHANGELOG.md -s"`

执行 `npm run genlog` 就会在 `.github` 文件夹中看到一个 `CHANGELOG.md` 文件了。

## 规范代码风格

使用 eslint 来规范

#### eslint

- 安装依赖
  `npm i -D eslint`

- 初始化
  `./node_modules/.bin/eslint --init`

根据提示一路选择配置，然后会自动生成一个 `.eslintrc` 文件

- 配置规则

[doc](https://eslint.org/docs/user-guide/configuring)

#### prettier

可以全局安装，也可以在项目上安装，建议在项目中安装固定的版本

- 安装
  `npm i -D --save-exact prettier`

- 配置
  新建 `.prettierrc.js` 文件，[doc](https://prettier.io/docs/en/configuration.html)
  配置文件还支持其他格式，具体看文档

- 开启 vscode 自动格式化
  安装 prettier 插件，修改配置文件

```json
{
  "prettier.singleQuote": true,
  "prettier.semi": false,
  "prettier.tabWidth": 2,
  "[javascript]": {
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

# 目录结构

```shell
├── build                      // 构建相关  
├── config                     // 配置相关
├── src                        // 源代码
│   ├── api                    // 所有请求
│   ├── assets                 // 主题 字体等静态资源
│   ├── components             // 全局公用组件
│   ├── directive              // 全局指令
│   ├── filtres                // 全局 filter
│   ├── icons                  // 项目所有 svg icons
│   ├── lang                   // 国际化 language
│   ├── mock                   // 项目mock 模拟数据
│   ├── router                 // 路由
│   ├── store                  // 全局 store管理
│   ├── styles                 // 全局样式
│   ├── utils                  // 全局公用方法
│   ├── vendor                 // 公用vendor
│   ├── views                   // view
│   ├── App.vue                // 入口页面
│   ├── main.js                // 入口 加载组件 初始化等
│   └── permission.js          // 权限管理
├── static                     // 第三方不打包资源
│   └── Tinymce                // 富文本
├── .babelrc                   // babel-loader 配置
├── eslintrc.js                // eslint 配置项
├── .gitignore                 // git 忽略项
├── favicon.ico                // favicon图标
├── index.html                 // html模板
└── package.json               // package.json
```

### api 和 views

建议：根据业务模块来划分 views，并将 views 和 api 两个部分的模块一一对应，公用的 api 模块单独放置。

### components

`src/components` 中放全局公用的组件，比如上传组件、富文本等，页面级的组件建议放到各自的 views 文件下方便维护。

### store

不要为了用 vuex 而用，如果业务之间的耦合度不高，也没必要使用 vuex 来存储数据。

# ESLint

- 安装依赖
  建议本地安装而非全局安装，因为每个项目的 ESLint 规则可能会不一样。
  `npm i -D eslint`

- 初始化
  `./node_modules/.bin/eslint --init`

也可以使用 `npx eslint --init`

根据提示一路选择配置，然后会自动生成一个 `.eslintrc` 文件

> npx is a npm package runner. 它会去 ./node_modules/.bin 中去找要执行的命令。

# webpack

### ProvidePlugin

webpack 的一个内置插件

配置例子：

```js
new webpack.ProvidePlugin({
  $: 'jquery',
  jQuery: 'jquery'
})
```

这样当 webpack 在第三方库中出现全局的 `$`, `jQuery`, `window.jQuery` 时，就会使用 node_modules 下的 jquery 包 export 出来的东西。

# 前后端交互

**跨域**

- CORS
- dev 环境: webpack-dev-server 的 proxy
- prod 环境: nginx 反向代理

**接口文档**

- 文档生成工具[https://swagger.io/](https://swagger.io/)
- 自行 mock: mock server 或者 [mockjs](https://github.com/badoo/MockJS) + [rap](https://github.com/thx/RAP) 或者 [easy-mock](https://easy-mock.com/)

**iconfont**
[教程](https://juejin.im/post/59bb864b5188257e7a427c09)

# 路由鉴权

**流程**

1. 登录获取 token -> 根据 token 获取用户权限 -> 根据用户权限计算出可访问的路由 -> 动态添加路由
2. 路由表分为通用路由表和异步挂载路由表两个部分，挂载 vueRouter 时只添加通用路由表，登录后根据用户权限计算出异步挂载路由表通过 addRoutes 来添加
3. '\*' 404 页面需要放在异步挂载路由表的最后，不能放到通用路由表中，不然它会拦截所有异步添加的路由
4. 在 vuex 中根据权限 filter 出可以访问的异步挂载路由表
