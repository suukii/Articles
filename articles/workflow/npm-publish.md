# 如何发布一个 npm package

## 简单发布

1. 注册一个 npm 账号
2. 创建一个 npm package
3. 运行 `npm login` 登录
4. 运行 `npm publish` 发布

完事，发布一个 npm package 就这么简单。

## 解决 package 命名冲突

不过，如果你要发布的 package 名字已经有人用了，npm 可能会提示 `You do not have permission to publish "package-name". Are you logged in as the correct user?`，要解决这个问题，你有以下两种做法可以选择：

1. 换一个名字继续发布
2. 在命名空间下发布

### 换一个名字发布

1. 通过 `npm search [name]` 命令搜索 package 名称是否已经存在
2. 在 `package.json` 中将 `name` 替换成没有人使用过的名字
3. 重新运行 `npm publish` 发布
4. 一般我们也会相应地修改项目文件名(tips: `mv [old-name] [new-name]` 命令可以用来做这件事情哦)

### 在命名空间下发布

要找到一个没有人用过的 package 名称不是易事，为了解决这个问题，npm 给我们提供了命名空间。我们的用户名或者组织名就是一个命名空间，如果要在命名空间下发布 package，只需要保证 package 名称在这个命名空间下是唯一就行。

要将 package 发布到命名空间下，要做如下改动：

1. 修改 package 名称，有两种做法：
    1. 手动在 `package.json` 中将 `name` 改成 `@username/package-name`
    2. 或者在创建 package 的时候使用 `npm init --scope=username` 而不是 `npm init`
2. 使用 `npm publish --access public` 命令进行发布(因为 private package 是收费服务)

## 专业发布

正经情况下，发布 package 不止这么简单，一般我们可能需要完成以下步骤：

-   运行测试
-   更新版本号
-   创建 git tag
-   推送最新代码到 github
-   发布 package 到 npm
-   创建 release notes

这么规律又繁复的工程，还是交给工具来吧，`np` 就是这个工具。

**前置工作**

1. 你的项目得是一个 git 仓库，而且还需要一个远程仓库
2. 起码得 push 过一次到远程仓库
3. 工作区目前是没有改动的

如果前置条件不满足就会发布失败。

**安装 np**

全局安装：

```shell
npm install --global np
```

**使用 np**

直接运行 `np` 就可以了，np 会弹出一个交互 UI 让你做各种选择，做完选择后剩下的发布工作就由 np 来完成了。

**np 文档**

https://github.com/sindresorhus/np

## Reading

-   https://medium.com/@bretcameron/how-to-publish-your-first-npm-package-b224296fc57b
-   https://zellwk.com/blog/publish-to-npm/
-   [adding badges to readme](https://shields.io/)
