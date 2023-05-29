---
layout: '../../layouts/MarkdownPost.astro'
title: '基于fluid主题的hexo博客搭建'
pubDate: 2023-05-27
description: '基于fluid主题的hexo博客搭建'
author: 'ZhJy'
cover:
    url: 'https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305282140339.png'
    square: 'https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305282140339.png'
    alt: 'cover'
tags: ["分享","教程"] 
theme: 'light'
featured: true

meta:
 - name: author
   content: ZhJy
 - name: keywords
   content: key3, key4

keywords: 教程, HEXO, fluid!
---

### 0.前言

![](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305282140339.png)

周末闲着无聊花了一天时间，搭建了一个hexo个人网站，使用的是了fluid主题，感谢作者的无私付出（[fluid-dev/hexo-theme-fluid: 一款 Material Design 风格的 Hexo 主题)](https://github.com/fluid-dev/hexo-theme-fluid)）。

### 1.安装

[Hexo](https://hexo.io/zh-cn/) 是一个快速、简洁且高效的博客框架。Hexo支持 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

#### 1.1 环境准备

安装Hexo其实非常简单，但是需要先在本地电脑中安装好下面几个软件，另外我还安装了Visual Studio Code及Typora，推荐Typora+picgo作为文章写作软件，支持自动上传图片到图床。

- [Node.js](http://nodejs.org/) (Node.js 版本需不低于 8.10，建议使用 Node.js 10.0 及以上版本)
- [Git](http://git-scm.com/)
- [nmp](https://www.npmjs.com/)
- [Visual Studio Code](https://code.visualstudio.com/)
- [Typora](https://typora.io/)

#### 1.2安装

安装好前三个软件后，即可使用npm安装Hexo了，打开一个PowerShell窗口，执行下面命令：

```powershell
npm install -g hexo-cli
```

安装以后，可以使用以下两种方式执行 Hexo：

1. `npx hexo`
2. 将 Hexo 所在的目录下的 `node_modules` 添加到环境变量之中即可直接使用： `hexo`

```powershell
echo 'PATH="$PATH:./node_modules/.bin"' >> ~/.profile
```

#### 1.3升级

如果后期需要升级的话，进入博客的目录，先检查更新:

```text
E:\Blog\hexo> npm outdated
Package  Current  Wanted     Latest  Location           Depended by
hexo       6.3.0   6.3.0  7.0.0-rc1  node_modules/hexo  hexo
```

修改 `package.json` 文件，基于 `Latest` 列内容更新版本号，然后更新并检查版本号：

```powershell
npm install --save
```

### 2.建站

#### 2.1初始化目录

安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件。

```powershell
hexo init <目标文件夹>
cd <目标文件夹>
npm install 
```

#### 2.2启动网页服务

等初始化执行完成后，通过`hexo s`命令即可在本地启动博客站点：

```powershell
>hexo s
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
```

浏览器访问http://localhost:4000就可以看到下面这页面了：

![](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305281920328.png)

### 3.更换主题

#### 3.1安装主题

**方式一**

Hexo 5.0.0 版本以上，推荐通过 npm 直接安装，进入博客目录执行命令：

```bash
npm install --save hexo-theme-fluid
```

然后在博客目录下创建 `_config.fluid.yml`，将主题的 [_config.yml (opens new window)](https://github.com/fluid-dev/hexo-theme-fluid/blob/master/_config.yml)内容复制过去。

**方式二**

下载 [最新 release 版本 (opens new window)](https://github.com/fluid-dev/hexo-theme-fluid/releases)解压到 themes 目录，并将解压出的文件夹重命名为 `fluid`。

#### 3.2指定主题

如下修改 Hexo 博客目录中的 `_config.yml`：

```yaml
theme: fluid  # 指定主题

language: zh-CN  # 指定语言，会影响主题显示的语言，按需修改
```

#### 3.3创建「关于页」

首次使用主题的「关于页」需要手动创建：

```powershell
hexo new page about
```

创建成功后修改 `/source/about/index.md`，添加 `layout` 属性。

修改后的文件示例如下：

```yaml
---
title: 标题
layout: about
---

这里写关于页的正文，支持 Markdown, HTML
```

> ***注意***
>
> `layout: about` 必须存在，并且不能修改成其他值，否则不会显示头像等样式。

完了执行下面两条命令，就可以看到新主题的样式了：

```powershell
hexo clean
hexo s
```

![](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305281955091.png)

#### 3.4更新主题

**方式一**

> 适用于通过 Npm 安装主题。

在博客目录下执行命令：

```bash
npm update --save hexo-theme-fluid
```

**方式二**

> 适用于通过 Release 压缩包安装主题，且没有自行修改任何代码的情况。

1. 先将原文件夹重命名为别的名称，例如 `fluid-bkp`，用于升级失败进行回退；
2. 按照安装步骤，重新下载 [release (opens new window)](https://github.com/fluid-dev/hexo-theme-fluid/releases)并解压重命名为 `fluid`；
3. 如果某些配置发生了变化（改名或弃用），release 的说明里会特别提示，同步修改原配置文件即可。

**方式三**

> 适用于自定义了一些代码，或想体验其他分支的情况，以 dev 分支为例。

1. 确定自己的 fluid 目录已经开启 git，并且所有改动都已 commit；
2. 把 fluid 仓库的 develop 分支拉取到自己当前的分支上（也可新建一个分支再拉取）：

```bash
git pull https://github.com/fluid-dev/hexo-theme-fluid.git develop
```

3. 解决代码冲突，保留自己修改的部分（如何解决冲突可自行搜索）

#### 3.5更多主题配置

更多的主题配置请参考官方配置指南：

[配置指南 | Hexo Fluid 用户手册 (fluid-dev.com)](https://hexo.fluid-dev.com/docs/guide/)

### 4.部署到Github

通过 [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git) 插件可以实现一键将博客部署到github提供的 pages 服务。

#### 4.1新建存储库

建立名为 `<repository的名字>.github.io` 的储存库，这样你的博客网址为 `<你的 GitHub 用户名>.github.io`，例如下面这样：

![](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305282020235.png)



#### 4.1安装hexo-deployer-git

```powershell
npm install hexo-deployer-git 
```

#### 4.2修改`_config.yml`文件

在配置文件中追加下面的内容：

```yaml
deploy:
  type: git
  repo: https://github.com/conscloud/conscloud.github.io
  # example, https://github.com/hexojs/hexojs.github.io
  branch: gh-pages
  ignore_hidden: false
```

#### 4.3部署发布

Commit 并 push 到默认分支上，当部署完成后，在 `gh-pages` 分支可以找到生成的网页，并在 GitHub 储存库中，前往 `Settings > Pages > Source`，并将 branch 改为 `gh-pages`。

部署执行下面的命令：

```powershell
hexo clean && hexo deploy
```

前往 `https://<你的 GitHub 用户名>.github.io/` 查看网站。

### 5.部署到vercel

部署到github一个是速度不够快，另一个是国内的网络环境有时可能无法访问，推荐部署到[Vercel](https://vercel.com/),配合自定义域名，可以提高访问速度。

#### 5.1新建工程

首先先注册一个vercel账号后，在页面中点击`New Project`，创建工程。

![](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305282033030.png)

然后通过绑定的 `github` 或者导入需要部署的项目。

![](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305282036266.png)

因为导入的项目是打包好的静态页，`FRAMEWORK PRESET` 选择 `Other` 。

![](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305282036659.png)

点击 `deployed` 进行部署，如果部署失败可以查看报错信息是不是上一步的某些选项没有覆盖。部署成功后会进入如图所示的界面：

![](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305282038893.png)

#### 5.2自定义域名

**域名可以购买或者去[Freenom](https://www.freenom.com/zh/index.html?lang=zh)申请免费的域名，但是Freenom域名听说有可能存在被收回的风险。**

- 默认情况下部署成功后 `vercel` 会给你生成一个默认的域名，如果想要修改成自己的域名可将域名名称修改成自己的。
- 当选择修改成自己的域名名称后，`vercel` 会检查域名指向的 `DNS` 对不对，如果不对的话会提示你域名的 DNS 应该如何配置，按照 `vercel` 提示的 `DNS` 信息

在自己的域名的 `DNS` 配置中进行配置，如图在 setting 中配置自定义域名：

![](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305282042641.png)

![](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305282046481.png)

待生效后，就可以用自定义的域名访问了。

#### 5.4修改Vrecel服务器区域

在Settings-Functions中修改区域为香港。

![](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305282049763.png)

我用的是freenom的免费域名，通过cloudfare管理，服务器区域修改为香港后，在晚上高峰期测试的延迟情况：

![](https://cdn.jsdelivr.net/gh/conscloud/picgotemp/imgplus/202305282051918.png)

### 6.总结

好像就是这样就完事了。

哦！感谢下列文章作者：

[文档 | Hexo](https://hexo.io/zh-cn/docs/)

[配置指南 | Hexo Fluid 用户手册 (fluid-dev.com)](https://hexo.fluid-dev.com/docs/guide/)

[三万字教程\]基于Hexo的matery主题搭建博客并深度优化一站式完全教程 | 夜法之书 (17lai.site)](https://blog.17lai.site/posts/40300608/#!)

[Vercel部署高级用法教程 - Hexo Theme Fluid (fluid-dev.com)](https://hexo.fluid-dev.com/posts/hexo-vercel/#)

