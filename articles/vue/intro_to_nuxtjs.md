# Nuxt.js 入门

## Nuxt.js 是什么

TODO

是基于 Vue 的一个开发框架。

Nuxt.js 之于 Vue，相当于 Next.js 之于 React，可以这样分：`nUxt` 和 `vUe`，`nExt` 和 `rEact`。

## 安装

**第一步：创建项目**

```shell
npm init nuxt-app <project-name>
```

> 或者[手动创建](https://nuxtjs.org/guides/get-started/installation#manual-installation)

**第二步：安装 Nuxt**

```shell
npm install nuxt
```

## 路由

Nuxt 会为 `pages` 目录下的所有 `.vue` 文件自动生成路由，开发者不需要自己配置路由。

而且 Nuxt.js 还会为路由文件进行自动 code splitting，开发者不需要进行额外的配置。

### 普通路由

文件结构：

```
pages/
--| user/
-----| index.vue
-----| one.vue
--| index.vue
```

自动创建的路由结构：

```js
router: {
    routes: [
        {
            name: 'index',
            path: '/',
            component: 'pages/index.vue',
        },
        {
            name: 'user',
            path: '/user',
            component: 'pages/user/index.vue',
        },
        {
            name: 'user-one',
            path: '/user/one',
            component: 'pages/user/one.vue',
        },
    ];
}
```

### 动态路由

创建动态路由只需要在文件名前面加 `下划线` 就好了。

文件结构：

```
pages/
--| _slug/
-----| comments.vue
-----| index.vue
--| users/
-----| _id.vue
--| index.vue
```

> \_slug/index.vue 这种结构表示 `:slug` 参数是必须的
> \_id.vue 这种结构表示 `:id?` 参数是可选的
> 如果要将 `:id` 参数变成必选，只需要修改文件结构为 `_id/index.vue`

自动创建的路由结构：

```js
router: {
    routes: [
        {
            name: 'index',
            path: '/',
            component: 'pages/index.vue',
        },
        {
            name: 'users-id',
            path: '/users/:id?',
            component: 'pages/users/_id.vue',
        },
        {
            name: 'slug',
            path: '/:slug',
            component: 'pages/_slug/index.vue',
        },
        {
            name: 'slug-comments',
            path: '/:slug/comments',
            component: 'pages/_slug/comments.vue',
        },
    ];
}
```

### 嵌套路由

需要创建一个和父级页面同名的文件夹。还需要在父级路由的页面组件中加入 `NuxtChild` 组件。

文件结构：

```
pages/
--| users/
-----| _id.vue
-----| index.vue
--| users.vue
```

自动创建的路由结构：

```js
router: {
    routes: [
        {
            path: '/users',
            component: 'pages/users.vue',
            children: [
                {
                    path: '',
                    component: 'pages/users/index.vue',
                    name: 'users',
                },
                {
                    path: ':id',
                    component: 'pages/users/_id.vue',
                    name: 'users-id',
                },
            ],
        },
    ];
}
```

### 不常见的路由需求

-   [动态嵌套路由](https://nuxtjs.org/docs/2.x/features/file-system-routing#dynamic-nested-routes)
-   [不确定嵌套层级的动态路由](https://nuxtjs.org/docs/2.x/features/file-system-routing#unknown-dynamic-nested-routes)

### 路由拓展

-   用 [router-extras-module](https://github.com/nuxt-community/router-extras-module) 自定义页面的路由参数。
-   用 [@nuxtjs/router](https://github.com/nuxt-community/router-module) 来自己写 `router.js` 覆盖 Nuxt 的路由。
-   在 `nuxt.config.js` 中使用 [router.extendRoutes](https://nuxtjs.org/docs/2.x/configuration-glossary/configuration-router#extendroutes) 属性进行配置。

**导航**

使用 `NuxtLink` 组件来进行页面间导航，这个组件是由 Nuxt 提供的，不用额外引入，可以看成是 Vue 的 `RouterLink` 的替代品。

## 目录结构

### pages 文件夹

Nuxt 会为这个文件夹下的 `.vue` 文件生成页面路由。

[learn more](https://nuxtjs.org/guides/directory-structure/pages)

### components 文件夹

放 Vue 组件的地方，组件会由 Nuxt 自动引入，开发者不需要手动引入。（前提是 `nuxt.config.js` 中的 `component` 字段设为 `true`）

[learn more](https://nuxtjs.org/guides/directory-structure/components)

### assets 文件夹

放不用编译的资源，比如样式文件、图片、字体。

[learn more](https://nuxtjs.org/guides/directory-structure/assets)

### static 文件夹

会映射到服务器根路由。

[learn more](https://nuxtjs.org/guides/directory-structure/static)

### `nuxt.config.js` 文件

[配置文件](https://nuxtjs.org/guides/directory-structure/nuxt-config)

[learn more about directories](https://nuxtjs.org/guides/directory-structure/nuxt)

## 渲染

-   服务端渲染(SSR)：就是每次用户请求的时候都在服务端动态获取数据，组合好 HTML 文档返回给浏览器。
-   静态网站(static sites)：HTML 文档是在打包构建阶段就已经生成了，用户请求页面的时候服务器只要直接返回文档。

要开启 SSR 或者静态渲染的话，在 `nuxt.config.js` 中设置：

```js
export default {
    ssr: true, // 不过这是默认值，所以其实不用手动设置
};
```

-   客户端渲染：指的是用户请求页面时，服务端返回一个基础的 HTML 文档，里面是没有数据的，浏览器接收到文档后再使用 JS 进行数据请求，然后将数据渲染到页面中。

在 `nuxt.config.js` 中设置：

```js
export default {
    ssr: false,
};
```

## 部署

### 静态网址

在 `nuxt.config.js` 中设置：

```js
export default {
    target: 'static', // 默认值是 'server'
};
```

使用 `nuxt generate` 命令打包生成静态文件，Nuxt.js 会为每一个页面路由生成一个 HTML 文件，放到 `dist/` 文件夹下。

### 服务端渲染

在 `nuxt.config.js` 中设置：

```js
export default {
    target: 'server', // 这是默认值
};
```

使用 `nuxt build` 命令打包构建项目。

## 引入 element-ui

安装 element-ui 之后在 `nuxt.config.js` 中设置：

```js
export default {
    css: ['element-ui/lib/theme-chalk/index.css'],
    plugins: [{ src: '~plugins/element-ui', ssr: false }],
    build: {
        vendor: ['element-ui'],
    },
};
```

新增 `plugins/element-ui.js`：

```js
import Vue from 'vue';
import Element from 'element-ui';
Vue.use(Element);
```
