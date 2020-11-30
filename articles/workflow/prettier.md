# 如何规范代码风格 - prettier

## Prettier

Prettier 是一个自动格式化代码的工具。

**安装**

```shell
npm i -D prettier
```

**配置**

```json
// .prettierrc.json
{}
```

-   [其他配置文件形式](https://prettier.io/docs/en/configuration.html)
-   [配置选项](https://prettier.io/docs/en/options.html)

定义 `.prettierignore` 告诉 prettier 哪些文件不用管：

```sh
# .prettierignore
# Ignore artifacts:
build
coverage
```

**使用**

```shell
npx prettier --write .
```

-   会格式化所有文件(改写)
-   也可以指定要格式化的文件路径：`prettier --write "app/**/*.test.js"`

```shell
prettier --check .
```

-   只会检查文件格式有没有符合要求，并不会改写文件

**编辑器**

当然，命令行的使用方式并不常用，我们更多是让编辑器帮我们自动格式化。

VSCode 可以安装 `Prettier - Code formatter` 插件。

[配置编辑器](https://prettier.io/docs/en/editors.html)

注意：记得在项目本地安装 prettier。

**ESlint**

如果项目同时也使用 ESlint 的话，需要安装 [eslint-config-prettier](https://github.com/prettier/eslint-config-prettier#installation)，然后在 eslint 配置中加上 `extends: ['prettier']`。它会关掉 ESlint 中不需要的或者跟 prettier 有冲突的规则。

-   prettier 是用来规范代码风格的，比如缩进多少、加不加分号之类的。
-   eslint 更多是用来规范代码质量的，比如能不能有声明了但是不用的变量，能不能定义隐式全局变量这类规则。

## husky

虽然项目中是安装了 prettier，但不能保证每个人都能主动运行命令或者是配置编辑器。

所以，我们还是需要工具来自动检查和格式化。

[这里](./git-hooks.md) 已经介绍过 husky 是干什么的了。

```json
// package.json
{
    "husky": {
        "hooks": {
            "pre-commit": "prettier --write . && git add -A ."
        }
    }
}
```

-   执行 `prettier --write .` 格式化所有代码
-   执行 `git add` 把所有修改的文件加入暂存区，一起提交

## lint-staged

不过 `prettier --write .` 每次都会格式化所有文件，但其实我们只需要检查有修改的文件就好了。

lint-staged 就是用来解决这个问题的。

**安装**

```shell
npm i -D lint-staged
```

**配置**

```json
// package.json
{
    "husky": {
        "hooks": {
            "pre-commit": "lint-staged"
        }
    },
    "lint-staged": {
        "*": "prettier --write"
    }
}
```

**原理**

简单地说，lint-staged 就是把改动文件的路径加到了 `prettier --write` 命令后面。

同时它还把修改文件暂存了，我们也不需要 `git add -A .` 了。
