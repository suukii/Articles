# 用 Node 写一个 cli

## hello world cli

首先我们先来写一个简易版本的 cli，看下一个 cli 项目长什么样。

**1. 创建一个 npm 项目**

```shell
mkdir sue-cli && cd sue-cli && npm init -y
```

-   sue-cli 是本项目的名称，可以换成你喜欢的名字

**2. 在 `package.json` 中设置命令名称**

```json
{
    "bin": {
        "sue": "./bin/cli.js"
    }
}
```

-   `sue` 是使用这个 cli 时要在命令行输入的指令
-   `./bin/cli.js` 是要执行的脚本文件
-   指令名称是自定义的，而且脚本也不是一定要放在 bin 文件夹下的

**3. 创建要执行的脚本文件**

创建 `./bin/cli.js` 文件，输入以下内容：

```shell
#! /usr/bin/env node

console.log('hello world');
```

-   `#! /usr/bin/env node` 这一行是用来告诉系统，当前文件需要用 node 来执行。一般如果我们要在 node 环境中执行 js 文件，是通过 `node index.js` 命令的，但如果在 js 文件开头加了这句 shebang(指 `#!` 开头的语句)，我们就可以直接通过 `index.js` 命令来执行文件了。
-   `console.log('hello world');` 就是这个 cli 目前所做的所有事情。

**4. 调试 cli**

在 sue-cli 目录下运行 `npm link`，就相当于全局安装了 `sue-cli` 这个依赖，然后我们就可以在本机其他地方使用 `sue` 命令了。

调试结束之后可以在 sue-cli 目录下运行 `npm unlink` 删除依赖。

**5. 发布 cli**

关于如何发布一个 npm package 看[这里](./npm-publish.md)。

发布成功之后，就可以通过 `npm install --global sue-cli` 下载啦。

然后运行 `sue` 我们就可以在命令行看到 `hello world` 被输出了。

当然，没有人会写一个 cli 只是用来打印 `hello world` 的 ( ╯□╰ )。

## 简单脚手架

如果是前端，应该都有用过 `vue-cli` 或者 `create-react-app` 吧，那我们就来写一个简单的脚手架。

在开始敲代码之前，先来定义一下用户是怎么使用这个脚手架的：

1. `sue create [name]`: 创建一个名为 `[name]` 的项目
2. `sue create`: 在当前文件夹下创建项目

就这么简单的两种用法。然后第一个需求产生了：我们需要获取用户输入，看他有没有提供 `[name]`。

用 Node 获取用户输入的方法有很多种，其中之一就是通过 `process.argv`，不过 `process.argv` 获取的信息比较原始，如果不想自己进一步处理，还可以使用 `minimist` 来简化操作。

第二个需求，使用模板项目生成一个新的项目，这是脚手架的关键任务啦。

模板项目可以直接放在 cli 项目中，也可以是一个单独的项目。如果是一个单独的项目，我们还需要把项目从 Github(或者 GitLab 之类的) 上面下载下来，可以使用 [download-git-repo](https://www.npmjs.com/package/download-git-repo) 来实现这个功能。

我们先来实现这两个核心的功能吧。首先安装依赖：

```shell
npm i -D download-git-repo minimist
```

首先是获取输入：

```js
const minimist = require('minimist');
const argv = minimist(process.argv.slice(2));

const create = argv._[0] === 'create';
const dirname = argv._[1] || '';

if (!create) return;
```

-   没带 `--` 和 `-` 的参数 minimist 都会把它们放在 `_` 数组中。

然后是下载 GitHub 项目：

```js
const path = require('path');
const download = require('download-git-repo');

download(
    'suukii/91-days-algorithm',
    path.resolve('./', dirname),
    {},
    function (err) {
        console.log(err ? 'Error' : 'Success');
    },
);
```

-   `download-git-repo` 的简单使用只需要指定要下载的仓库 `username/repo` 和要下载到的目标文件夹就可以了，更多选项可以去看文档。

至此一个简单的脚手架就完成啦，虽然比较简陋，以下是 `sue-cli.js` 的完整代码：

```js
'use strict';

const path = require('path');
const minimist = require('minimist');
const download = require('download-git-repo');
const argv = minimist(process.argv.slice(2));

const create = argv._[0] === 'create';
const dirname = argv._[1] || '';

if (argv.help || !create) {
    printHelp();
    return;
}

downloadRepo(dirname);

// ********************************

function printHelp() {
    console.log('Usage: sue <commane> [options]');
    console.log('');
    console.log('Options:');
    console.log('--help                             output usage information');
    console.log('');
    console.log('Commands:');
    console.log('create <app-name>                  create a new project');
}

function downloadRepo(dirname) {
    download(
        'suukii/91-days-algorithm',
        path.resolve('./', dirname),
        {},
        function (err) {
            console.log(err ? 'Error' : 'Success');
        },
    );
}
```

## TODO

对于一个脚手架来说，还有很多功能可以加呢，比如提供配置选项让用户选择，然后下载不同的模板项目，或者在创建项目的时候加一个进度条，提升用户体验。

晚点再继续写。
