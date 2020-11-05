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

Nuxt 会为 `pages` 目录下的所有 `.vue` 文件自动生成路由，开发者无需自己配置路由。

**导航**

使用 `NuxtLink` 组件来进行页面间导航，这个组件是由 Nuxt 提供的，不用额外引入，可以看成是 Vue 的 `RouterLink` 的替代品。

## 目录结构

### pages 文件夹

Nuxt 会为这个文件夹下的 `.vue` 文件生成页面路由。

[learn more](https://nuxtjs.org/guides/directory-structure/pages)

### components 文件夹

放 Vue 组件的地方，组件会由 Nuxt 自动引入，开发者无需手动引入。（前提是 `nuxt.config.js` 中的 `component` 字段设为 `true`）

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
