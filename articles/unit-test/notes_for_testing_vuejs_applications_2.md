# Test Vuejs Application - Chapter 2

-   打包工具
-   Linting
-   Jest
-   Vue Test Utils
-   Chrome Debugger

## 打包工具

在用 Vue 开发时，我们写的 SFC 和一些比较新的 JS 特性都不能在浏览器中正常运行，所以需要一些程序来讲我们写的源码编译打包成可以在浏览器运行的代码，这些程序就叫做打包工具。

现在应用最广泛的打包工具就是 webpack。webpack 的工作就是把分成模块的各个 JS 文件打包到一个文件中。

## Linting

Linting 意在检查代码中的**潜在错误**和**格式问题**。

ESLint 是一个 linting 库，可以自己配置检查的规则，也可以使用别人发布的规则。

linting 也是测试中重要的一环。

## Jest

Jest 是一个测试框架。

**健全测试(sanity test)**

健全测试一般指一个初始化的测试工作。这个测试是保证会通过的，如果这个测试没有通过，那就是测试配置出了问题。我们可以用健全测试来判断到底是测试配置还是源码导致的测试结果不通过，所以写真正的测试代码之前，可以先写一个健全测试。

**glob pattern**

```regexp
**/__tests__/**/*.js?(x),**/?(*.)(spec|test).js?
```

这是 Jest 用来寻找测试文件的模式。它会寻找 `__tests__` 文件夹下的 `js` 或者 `jsx` 文件，以及所有的 `.spec.js` 和 `.test.js` 文件

-   [ ] [more about glob](www.npmjs.com/package/glob#glob-primer)

**watch mode**

在命令行中传入 `--watch` 选项可以开启监听模式，文件发生修改时 Jest 会自动执行一次测试代码。

**lint error**

因为 ESLint 并不知道我们的测试代码是在 Jest 中执行的，所以当我们使用 `test` `describe` 等方法时，ESLint 会报错，告诉我们这些全局方法不存在。

我们需要在 ESLint 的配置项中指定 `env`，这个配置告诉 ESLint 我们的代码运行在什么环境中。

```json
{
    "eslintConfig": {
        "env": {
            "jest": true
        }
    }
}
```

**false positive**

在测试中，false positive 指的是不管源码如何都总会通过的测试代码。

常见的例子就是异步代码：把断言写在异步代码中，整个测试会在断言执行前就结束，该断言永远不会被执行，因此也不会影响测试结果。

避免 false positive 的最好办法就是使用 TDD。

**用 describe 来给测试代码分块**

tips: 不要过多嵌套 describe，要不然当你想增加测试代码时，会发现很难确定到底要在哪个 describe 中增加。

**编译源码**

在 Vue 项目中使用 Jest 时，由于 SFC 并不是可执行的 JS 代码，我们还需要使用 Jest Transformer(thoughts: 应该是像 webpack loader 的东西。) 来对源码进行编译。

-   vue-jest: 将 SFC 编译成 JS。
-   babel-jest: 将 modern JS 编译成能在 Node 中运行的 JS。

配置：

```json
{
    "jest": {
        "transform": {
            "^.+\\.js$": "babel-jest",
            "^.+\\.vue$": "vue-jest"
        }
    }
}
```

## Vue Test Utils

Vue Test Utils 是一个官方的测试库，提供了很多工具方法来测试 Vue 组件，常用 API：

-   mount: 挂载一个组件并返回一个包装对象，这个包装对象中有已挂载的组件实例以及一系列工具方法。
-   shallowMount: 跟 mount 相似，不同在于 shallowMount 不会渲染子孙组件，而是对它们进行存根。(存根就是指用一个没有内容的空标签来替换子组件)

## Chrome Debugger

```json
{
    "scripts": {
        "test:unit:debug': "node --inspect-brk ./node_modules/jest/bin/jest.js --no- cache --runInBand"
    }
}
```

-   运行 debug 命令后打开浏览器 `chrome://inspect`
-   在 Remote Targets 下面找到 `node_modules/jest/bin/jest`
