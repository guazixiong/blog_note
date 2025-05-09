# 团队项目的 Git 分支管理规范

## 分支管理

### 分支说明

创建项目时，会针对不同环境创建几个常用分支：

- **master**：主分支，负责记录上线版本的迭代，该分支代码与线上代码是完全一致的。
- **develop**：开发分支，该分支记录相对稳定的版本，所有的 `feature` 分支和 `bugfix` 分支都从该分支创建。
- **release**：发布分支，用于代码上线准备，该分支从 `develop` 分支创建，创建之后由测试人员发布到测试环境进行测试，测试过程中发现 bug 需要开发人员在该 `release` 分支上进行 bug 修复，修复完成自测没问题后，在上线之前，需要合并该 `release` 分支到 `master` 分支，同时需要再合并该分支到 `develop` 分支。

平时开发工作中，会根据需要由开发人员创建几类临时分支：

- **feature**：特性（功能）分支，用于开发某个特定的功能，该分支从 `develop` 分支创建，不同的功能创建不同的功能分支，开发完成自测没问题后，需要合并该分支到 `develop` 分支，之后删除该分支。
- **bugfix**：bug 修复分支，用于修复某个普通的 bug，该分支从 `develop` 分支创建，修复完成自测没问题后，需要合并该分支到 `develop` 分支，之后删除该分支。
- **hotfix**：热修复分支，用于修复某个紧急的 bug，该分支只有在紧急情况下使用，该分支从 `master` 分支创建，用于紧急修复线上的 bug，修复完成自测没问题后，需要合并该分支到 `master` 分支，以便上线，同时需要再合并该分支到 `develop` 分支，之后删除该分支。

### 分支命名规范

#### 特性（功能）分支

该分支的分支名称应该为描述该功能的英文简称：

```javascript
feature/分支名称
```

例如，开发的功能为“新增商品到物料库”，则可以创建一个名为 `feature/material-add` 的分支。

#### bug 修复分支、热修复分支

该分支的名称应该为 Java中 bug 代码或者是描述该 bug 的英文简称：

```javascript
bugfix/分支名称
hotfix/分支名称
```

比如，修复的 bug 在 java中代号为 MATERIAL-1，则可以创建一个名为 `bugfix/MATERIAL-1` 的分支。

#### 发布分支

该分支的名称应该为本次发布的主要功能英文简称：

```javascript
release/分支名称
```

比如，本次上线的功能为“新增商品到物料库”，则可以创建一个名为 `release/material-add` 的分支。

#### 流程规范

##### 正常开发流程

1. 从 `develop` 分支切出一个新分支，根据是功能还是 bug，命名为 feature/* 或 bugfix/* 分支。
2. 开发者完成开发，提交该分支到远程仓库。
3. 开发者发起 `merge request` 请求（可在 `gitlab` 页面 `New merge request`），将新分支请求合并到 `develop` 分支，并提醒组长或同事进行 `code reviewer`。
4. `code reviewer`后，若无问题，则接受 `merge request`，并将新分支合并到 `develop` 分支，同时可以删除新建分支；若有问题，则不能进行合并，可 `close` 该请求，同时通知开发者在新分支上进行相应调整，调整完后提交代码，重复 `code reviewer` 流程。
5. 转测时，直接从当前 `develop` 分支合并到 `release` 分支，重新构建测试环境，完成转测。
6. 测试完成后，从 `release` 分支合并到 `master` 分支，基于 `master` 分支构建生产环境完成上线，并对 `master` 分支打 `tag`， `tag` 名可为 `v1.0.0_2019032115`（即：版本号_上线时间）。

流程示意图如下：

![img](%E5%9B%A2%E9%98%9F%E9%A1%B9%E7%9B%AE%E7%9A%84%20Git%20%E5%88%86%E6%94%AF%E7%AE%A1%E7%90%86%E8%A7%84%E8%8C%83.assets/857e63c57afb10906f0cee12604039c2.png)

##### 并行开发测试环境 Bug 修复流程

并行开发（即前一个版本已经转测但未上线，后一个版本又已在开发中并部分合并到了 `develop` 分支）过程中，转测后测试环境发现的 bug 需要修复，但是 `develop` 分支此时又有新内容且该部分内容目前不计划转测，可以从 `release` 切出一个 bug 修复分支，完成之后需要同时合并到 `release` 分支与 `develop` 分支，合并时参考“正常开发流程”。

流程示意图如下：

![img](%E5%9B%A2%E9%98%9F%E9%A1%B9%E7%9B%AE%E7%9A%84%20Git%20%E5%88%86%E6%94%AF%E7%AE%A1%E7%90%86%E8%A7%84%E8%8C%83.assets/4aac6bf4e435e8865064ab99a101cf8e.png)

##### 生产环境 Bug 修复流程

生产环境的 Bug 分两种情况：

1. 紧急 Bug：严重影响用户使用的为紧急 Bug，需立即进行修复,如关键业务流程存在问题，影响用户正常的业务行为。
2. 非紧急 Bug 或优化：非关键业务流程问题，仅影响用户使用体验，或出现频率较小等，为非紧急 Bug，可规划到后续版本进行修复。

非紧急 Bug 修复参考“正常开发流程”。 紧急 Bug 修复，需要从 `master` 分支切出一个紧急 bug 修复分支，完成之后需要同时合并到 `master` 分支与 `develop` 分支（如果需要测试介入验证，则可先合并到 `release` 分支，验证通过后再合并到 `master` 分支上线），合并时参考“正常开发流程”。

流程示意图如下：

![img](%E5%9B%A2%E9%98%9F%E9%A1%B9%E7%9B%AE%E7%9A%84%20Git%20%E5%88%86%E6%94%AF%E7%AE%A1%E7%90%86%E8%A7%84%E8%8C%83.assets/cf9d3929ebb9ba7f2153484acb5b1855.png)

## 拓展阅读

以上的说明仅是针对同一个仓库进行git分支管理, 当分支过多时,维护起来就很麻烦,因此可以仿照github上开源项目的协作方式进行开发.  保证主仓库的分支唯一性,被fork的仓库由开发者自我维护.

### github开源项目开发流程

1. 管理员创建组织，并创建项目主仓库,创建`develop`、`release`等不同的分支，并初始化项目代码。
2. 开发者加入组织，远程fork仓库到自己仓库中去。
3. 开发者在自己的仓库进行功能开发，提交该分支到自己的远程仓库。
4. 开发者发起`pull`请求,获取最新的代码到本地分支.
5. 开发者发起 `merge request` 请求（可在 `gitlab` 页面 `New merge request`），将新分支请求合并到远程仓库指定分支，并提醒组长或同事进行 `code reviewer`。
6. `code reviewer`后，若无问题，则接受 `merge request`，并将新分支合并到指定分支.
7. 转测时,可直接通过主仓库的分支进行构建测试环境,完成转测.
8. 测试完成后,从`release`分支合并到主仓库的`master`分支,基于`master`分支构建生产环境完成上线,并对`master`分支打`tag`, `tag` 名可为 `v1.0.0_2019032115`（即：版本号_上线时间）。