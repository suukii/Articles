# 如何规范 git commit 信息

## commit message

每次 git commit 我们都要写 commit message(提交说明)。

如果一行信息足够了，可以执行 `git commit -m 'hello world`。

如果想要输入更多信息，可以执行 `git commit` 调出编辑器填入多行信息。

理论上 commit message 写什么都行，不过为了项目的可维护性，规范还是要有的。

## commit message 规范

社区有很多规范，这里只列举 commit 的类型：

1. feat: 新特性
2. fix: 修复 bug
3. style: 关于代码风格的修改(注意不是 CSS style)
4. refactor: 重构
5. test: 测试相关的代码
6. docs: 文档相关
7. chore: 构建过程或者辅助工具的变动

## commitlint

当然，靠自觉来遵守规范是不太现实的，所以，还是用工具吧。commitlint 就是一个用来检查 commit message 是否符合规范的工具。

**安装**

第一步：安装 commitlint cli

```shell
npm i -D @commitlint/cli
```

第二步：安装规范，这里安装的是 `config-conventional`，也可以选择[其他规范](https://github.com/conventional-changelog/commitlint#shared-configuration)

```shell
npm i -D @commitlint/config-conventional
```

**配置**

```js
// commitlint.config.js
module.exports = {
    // 指定要用的规范
    extends: ['@commitlint/config-conventional'],
};
```

**使用**

```shell
npx commitlint --from HEAD~1 --to HEAD --verbose
```

-   检查上一次 commit 的信息是否符合规范。

当然，这没什么 🥚 用。我们要的是在 commit 前就检查信息是否符合规范，如果不符合就阻止他提交。

## husky + commitlint

[这里](./git-hooks.md) 已经介绍过 husky 是干什么的了。

**配置**

```json
{
    "hooks": {
        // 在提交 commit 信息的时候运行 commitlint 命令
        "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
}
```

-   https://dev.to/talohana/husky-and-commitlint-for-clean-git-log-44be
-   https://www.freecodecamp.org/news/writing-good-commit-messages-a-practical-guide/
