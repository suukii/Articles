# 如何监听 git hooks

## git hooks

首先来简单了解下 git hooks 是什么。

在 git 的生命周期中，会有很多事件发生，比如 `commit`, `push` 等等。在这些事件发生时，我们可以指定执行一些脚本，这些脚本就是 git hooks。

要用比喻来说的话，有点像 Vue 组件的生命周期函数，但也不完全一样，Vue 的生命周期函数不能阻止组件进入下一个生命周期，而 git hooks 却可以阻止 git 事件的发生。

我们可以自己写 git hook 脚本，不过有更简单的办法，就是用 husky(git hooks for Node.js)！

// 谁要自己写啊 (⊙ˍ⊙)？

## husky

[https://typicode.github.io/husky/#/](https://typicode.github.io/husky/#/)

**安装**

```shell
npm i -D husky
```

**安装注意**

-   在初始化 git 仓库之后再安装 husky。
-   或者在安装 husky 之后运行 `npx husky install` 命令。

这样 husky 才能起作用。

**package.json 配置**

```json
// package.json
{
    "husky": {
        "hooks": {
            // 要监听的 git hook: 要执行的命令
            "pre-commit": "echo hello suukii"
        }
    }
}
```

**.huskyrc 配置**

配置也可以单独放在一个文件中：

```json
// .huskyrc
{
    "hooks": {
        "pre-commit": "echo hello suukii"
    }
}
```

**绕过 git hooks**

如果想要绕过 git hooks，可以使用 `--no-verify` 或者 `-n` 参数：

```shell
git commit -m 'yoho' --no-verify
```

如果是没有 `--no-verify` 参数的 git 命令，可以使用 HUSKY 环境变量：

```shell
HUSKY=0 git push
```

**常用的 hooks**

-   `commit-msg`: `git commit` 带参数的时候触发
-   `pre-commit`: `git commit` 不带参数参数时也会触发
-   `push`
-   husky 几乎支持 [git 提供的所有 hooks](https://git-scm.com/docs/githooks)

## git hooks 可以做什么

-   规范 commit 信息
-   规范代码格式
-   运行测试
-   ......
