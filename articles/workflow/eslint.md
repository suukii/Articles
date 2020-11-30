# 如何规范代码质量 - eslint

## eslint 可以干嘛

帮我们做静态语法分析，在代码执行之前发现错误。

1. 帮我们发现语法错误：使用未声明变量、缺少右括号等等。
2. 提示我们要遵循最佳实践：使用 `===` 而不是 `==`，不要使用 eval 等。
3. 提示我们删除多余代码：`return` 后面的语句、永远也不会执行的 `else` 分支等。
4. 统一代码风格：用单引号还是双引号、缩进几个空格等。不过这部分不是 eslint 专注的内容，统一代码风格还是用 [prettier](./prettier.md) 更好。

## 在项目中引入 eslint

**安装**

```shell
npm i -D eslint
```

**配置**

```shell
npx eslint --init
```

-   执行这条命令之后，做完一系列选择之后，就会生成一个配置文件 `.eslintrc.js`。

**使用**

```shell
npx eslint src/*.js
```

**修复**

```shell
npx eslint src/*.js --fix
```

-   不过有些错误 eslint 不能自动修复，比如使用没有声明的变量等，这些还是要靠我们自己修复。

**也可以将命令写成 script**

```json
// package.json
{
    "scripts": {
        "lint": "eslint src/*.js",
        "lint:fix": "eslint src/*.js --fix"
    }
}
```

如果项目中有测试的话，还可以定义 eslint 在测试前或者测试后进行检查：

```json
// package.json
{
    "scripts": {
        "pretest": "eslint --ignore-path .gitignore ."
    }
}
```

-   当我们运行 `npm test` 命令时，`pretest` 命令会自动在 test 之前执行，如果想要在 test 之后执行一些任务，可以定义 `posttest` 命令。

## `.eslintrc.js` 配置文件

`.eslintrc.js` 大概长这样：

```js
module.exports = {
    env: {
        browser: true,
        es2021: true,
        node: true,
    },
    extends: ['eslint:recommended', 'plugin:vue/essential'],
    parser: 'esprima',
    parserOptions: {
        ecmaVersion: 12,
        sourceType: 'module',
    },
    plugins: ['vue'],
    rules: {},
};
```

-   `env`: 表示你的代码是在哪个环境运行的。比如说你的代码只在 Node 中运行，那 eslint 就会检查代码中不能出现 document 或者 window 等。
-   `extends`: `eslint:recommended` 表示使用 [eslint 推荐的规则](https://eslint.org/docs/rules/)(打勾那些)。
-   `parser`：指定要用什么解析器来将源码解析成 AST，默认是 Espree。
-   `parserOptions`：跟解析器相关的一些配置。
-   `plugins`：比如使用 Vue 或者 React 来进行开发的话，有些语法跟 JS 不一样，就需要使用 plugins 来让 eslint 支持这些语法。好用的 plugin 看[这里](https://github.com/dustinspecker/awesome-eslint#plugins)
-   `rules`: 在这里可以自定义规则，会覆盖掉 extends 和 plugins 中的相同规则。全部规则在[这里](https://eslint.org/docs/rules/)。

**parserOptions 一些常见的选项**

-   `ecmaVersion`：表示源码使用的 JS 版本，比如 ecmaVersion 是 5，但源码中使用了 let，那 eslint 就没办法解析了。
-   `sourceType`：表示源码的类型，现在我们都写模块了，所以一般这个值是 `module`，它的默认值是 `script`。
-   `ecmaFeatures.jsx`：表示是否开启 JSX 语法支持，不过 eslint 支持的 JSX 跟 React 的不一样，如果要支持 React，需要使用 eslint-plugin-react。

## config 和 plugin

**config**

关于代码规范，我们可以选择官方的 `eslint:recommended` 或者 `eslint:all`，也可以选择[社区热门的](https://www.npmjs.com/search?q=eslint-config-)， 比如 Google, Airbnb, Standard 等，当然也可以完全自定义规则。

**plugin**

eslint 本身已经提供了很多规则，但这些还不够，比如我们要用 React 的话，也需要制定一些规则来检查 React 特有的语法。不过这些不用自己写，社区有[很多](https://www.npmjs.com/search?q=keywords:eslint-plugin)。plugin 制定了新的规则(rules)，还自带了 config，比如 eslint-plugin-react 就提供了 `eslint-plugin-react:recommended` 和 `eslint-plugin-react:all` 两种 config。要使用的话，我们只需要在 plugin 中增加 react，然后在 extends 中增加想要用的 config 就好了。

```json
{
    "plugins": ["react"],
    "extends": "plugin:react/recommended"
}
```

-   plugins 中，`eslint-plugin-` 前缀可省(同样 config 的前缀 `eslint-config-` 也是可以省略的)。
-   注意 extends 中用的是 `plugin:react` 前缀。

## 配合 prettier 使用

跟单独使用 prettier 不同的是，eslint-plugin-prettier 相当于将 prettier 当成 eslint 的一个规则来用。

**安装**

```shell
npm i -D prettier eslint-plugin-prettier eslint-config-prettier
```

-   `eslint-plugin-prettier` 自带了 `eslint-plugin-prettier:recommended`，不过 `eslint-plugin-prettier:recommended` 是继承了 `eslint-config-prettier` 的，所以要安装 `eslint-config-prettier`。

**配置**

```json
{
    "plugins": ["prettier"],
    "extends": "plugin:prettier/recommended"
}
```

## 配合 git 使用

如果是团队开发，那一定一定会有人忘记 lint 的。所以，这种事情还是需要强制监控一下。我们可以配置在 commit 之前检查代码，如果不符合规范就不能 commit。

[这里](./git-hooks.md) 已经介绍过 husky 是干什么的了。

```json
// package.json
{
    "husky": {
        "hooks": {
            "pre-commit": "npm run lint:fix"
        }
    }
}
```

如果项目中也用 prettier 的话，可以这样：

```json
// package.json
{
    "husky": {
        "hooks": {
            "pre-commit": "npm run lint:fix && lint-staged"
        }
    }
}
```

-   lint-staged 是啥可以看[这里](./prettier.md)

## eslint 原理

简单来说：

1. 首先将源码转换成 AST
2. 然后递归遍历 AST
3. AST 中每个选择器都会被访问两次，一次“自顶向下”(递)，一次“自底向上”(归)，默认是监听第一次的访问
4. 每条规则都会监听一个或者多个选择器
5. 在遍历到某个选择器的时候，如果它有被监听，就触发相应的回调
