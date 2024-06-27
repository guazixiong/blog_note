# Git提交规范

## Commit message

```bash
# 提交格式
<type>(<scope>): <subject>

<body>

<footer>

# 新增登录表单组件,包括邮箱和密码字段,关闭issue 123问题
feat(login): add login form component

Add a new component for the login form, including email and password fields.

Closes #123
```

目前，社区有多种 Commit message 的[写法规范](https://github.com/ajoslin/conventional-changelog/blob/master/conventions)。本文介绍[Angular 规范](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.greljkmo14y0)（见下图），这是目前使用最广的写法，比较合理和系统化，并且有配套的工具。

![](Git%20%E4%BB%A3%E7%A0%81%E6%8F%90%E4%BA%A4%E8%A7%84%E8%8C%83.assets/202302251938830.png)

## Angular  Git 代码提交规范

Angular 的 Git 代码提交规范遵循了一定的格式和规范，以确保提交信息的清晰度和可读性。下面是 Angular 的 Git 代码提交规范的主要要点：

1. 类型（Type）：表示本次提交的类型，是必填项，可以选择以下类型之一：

- feat：新功能（feature）
- fix：修复问题（bug fix）
- docs：文档（documentation）
- style：代码格式（formatting, missing semi colons, …）
- refactor：重构（refactoring code）
- test：增加测试（adding missing tests）
- chore：构建过程或辅助工具的变动（maintain）

1. 范围（Scope）：表示本次提交影响的范围，是可选项。

2. 主题（Subject）：本次提交的简短描述，是必填项。

3. 正文（Body）：本次提交的详细描述，是可选项。

4. 脚注（Footer）：本次提交的备注信息，是可选项，可以用于关闭 issue 或引用相关文档等。

5. 撤销（Revert）：如果当前 commit 用于撤销以前的 commit，则必须以`revert:`开头，后面跟着

   被撤销 Commit 的 Header。

下面是一个示例提交信息：

```bash
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>
```

```bash
feat(login): add login form component

Add a new component for the login form, including email and password fields.

Closes #123
```

> 在这个示例中，
>
> - "feat" 表示这是一个新功能，
> - "login" 是范围，
> - "add login form component" 是主题，
> - "Closes #123" 是脚注，表示本次提交关闭了 Issue #123。

Angular 的 Git 代码提交规范有助于提高团队合作的效率，提高代码质量，也有助于更好地维护代码库。

**特殊的撤销**

还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，则必须以`revert:`开头，后面跟着被撤销 Commit 的 Header。

```bash
revert: feat(pencil): add 'graphiteWidth' option

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
```

Body部分的格式是固定的，必须写成`This reverts commit <hash>.`，其中的`hash`是被撤销 commit 的 SHA 标识符。

如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的`Reverts`小标题下面。

## Commit message 的作用

格式化的Commit message，有几个好处。

**（1）提供更多的历史信息，方便快速浏览。**

比如，下面的命令显示上次发布后的变动，每个commit占据一行。你只看行首，就知道某次 commit 的目的。

```bash
$ git log <last tag> HEAD --pretty=format:%s
```

![](Git%20%E4%BB%A3%E7%A0%81%E6%8F%90%E4%BA%A4%E8%A7%84%E8%8C%83.assets/202302251942384.png)

**（2）可以过滤某些commit（比如文档改动），便于快速查找信息。**

比如，下面的命令仅仅显示本次发布新增加的功能。

```bash
$ git log <last release> HEAD --grep feature
```

**（3）可以直接从commit生成Change log。**

Change Log 是发布新版本时，用来说明与上一个版本差异的文档，详见后文。

![](Git%20%E4%BB%A3%E7%A0%81%E6%8F%90%E4%BA%A4%E8%A7%84%E8%8C%83.assets/202302251943710.png)

## Git 规范方式

为了实现规范，我们使用[commitlint](https://link.juejin.cn?target=https%3A%2F%2Fmarionebl.github.io%2Fcommitlint%2F%23%2F)和[husky](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Ftypicode%2Fhusky) 来进行提交检查，当执行`git commit`时会在对应的git钩子上做校验，只有符合格式的Commit message才能提交成功。

为了方便使用，增加了[commitizen](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcommitizen%2Fcz-cli)支持，使用[cz-customizable](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fleonardoanalista%2Fcz-customizable)进行配置。支持使用`git cz`替代`git commit`。

有兴趣的小伙伴可以查阅相关文档。

**PS: 之后如果要推行代码规范，也可以使用[husky](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Ftypicode%2Fhusky)来在其他的Git钩子(如pre-push等)上进行eslint等校验。**

**如何加入项目**

### commitlint & husky

```bash
// 项目目录下安装
npm i commitlint --save-dev
npm i @commitlint/config-conventional --save-dev

// 在项目目录下，新建配置文件 commitlint.config.js， 内容如下
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    // type 类型定义
    'type-enum': [2, 'always', [
      "feat", // 新功能 feature
      "fix", // 修复 bug
      "docs", // 文档注释
      "style", // 代码格式(不影响代码运行的变动)
      "refactor", // 重构(既不增加新功能，也不是修复bug)
      "perf", // 性能优化
      "test", // 增加测试
      "chore", // 构建过程或辅助工具的变动
      "revert", // 回退
      "build" // 打包
    ]],
    // subject 大小写不做校验
    // 自动部署的BUILD ROBOT的commit信息大写，以作区别
    'subject-case': [0]
  }
};
```

```bash
// husky
// 项目目录下安装
npm i husky --save-dev
// 在package.json文件中增加相关配置
"husky": {
  "hooks": {
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
  }
}
```

### commitizen

```bash
// 全局安装
npm install commitizen -g
// 项目目录下安装
npm install commitizen --save-dev
commitizen init cz-customizable --save --save-exact

// 在package.json文件中增加相关配置
// 注意这里的path可能要根据实际情况进行修改，如nAdmin项目
"config": {
  "commitizen": {
    "path": "./node_modules/cz-customizable"
  }
}

// 在项目目录下，新建配置文件 .cz-config.js
'use strict';

module.exports = {
  types: [
    {value: 'feat',     name: 'feat:     新功能'},
    {value: 'fix',      name: 'fix:      修复'},
    {value: 'docs',     name: 'docs:     文档变更'},
    {value: 'style',    name: 'style:    代码格式(不影响代码运行的变动)'},
    {value: 'refactor', name: 'refactor: 重构(既不是增加feature，也不是修复bug)'},
    {value: 'perf',     name: 'perf:     性能优化'},
    {value: 'test',     name: 'test:     增加测试'},
    {value: 'chore',    name: 'chore:    构建过程或辅助工具的变动'},
    {value: 'revert',   name: 'revert:   回退'},
    {value: 'build',    name: 'build:    打包'}
  ],
  // override the messages, defaults are as follows
  messages: {
    type: '请选择提交类型:',
    // scope: '请输入文件修改范围(可选):',
    // used if allowCustomScopes is true
    customScope: '请输入修改范围(可选):',
    subject: '请简要描述提交(必填):',
    body: '请输入详细描述(可选，待优化去除，跳过即可):',
    // breaking: 'List any BREAKING CHANGES (optional):\n',
    footer: '请输入要关闭的issue(待优化去除，跳过即可):',
    confirmCommit: '确认使用以上信息提交？(y/n/e/h)'
  },
  allowCustomScopes: true,
  // allowBreakingChanges: ['feat', 'fix'],
  skipQuestions: ['body', 'footer'],
  // limit subject length, commitlint默认是72
  subjectLimit: 72
};
```

### 自动生成Change log

使用`npm run changelog`自动生成CHANGELOG.md文件。

规范了Commit格式之后，发布新版本时，使用[Conventional Changelog](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fconventional-changelog%2Fconventional-changelog%23conventional-changelog)就能够自动生成Change log，生成的文档包括以下3个部分：

- New features
- Bug fixes
- Breaking changes(不向上兼容的部分，我们的规范不要求footer，所以这一项不会出现)

每个部分都会罗列相关的 commit ，并且有指向这些 commit 的链接。当然，生成的文档允许手动修改，所以发布前，你还可以添加其他内容。

**如何加入项目**

```awk
// 安装
npm i conventional-changelog-cli --save-dev
// 在package.json中加入配置方便使用
"scripts": {
  "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s"
}
// 项目目录下生成 CHANGELOG.md 文件
npm run changelog复制代码
```

### 加入项目之后的工作

1. 在readme中增加文档
2. 提交package-lock.json(如果因为安装依赖导致变化)

### 配置完成后其他人如何使用

1. 将配置测试完成的代码合并至主分支。
2. 其他开发者需要重新执行`sudo npm i`安装相关依赖。
3. 如果要使用`commitizen`，需要全局安装`sudo npm i commitizen -g`

## 问题汇总

### Git commit 中文乱码

文件提交编码格式

```bash
git config --global i18n.commitencoding utf-8
```

log输出的编码格式

```bash
git config --global i18n.logoutputencoding utf-8
```

界面编码格式

```bash
git config --global gui.encoding utf-8
```

## 参考链接

1. [Commit message 和 Change log 编写指南 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)
2. [超详细的Git提交规范引入指南 - 掘金](https://juejin.cn/post/6844903793033756680)
3. ChatGPT