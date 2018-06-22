---
title: build blog
tags:
  - null
categories:
  - null
date: 2018-06-22 10:44:35
---

### hexo安装与配置
#### 切换阿里npm源
为加快访问速度, 使用阿里的npm源. 文档: https://npm.taobao.org/, 可以使用阿里定制的cnpm替换默认的npm命令

```bash
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```
或者配置一个`alias`<!-- more -->

```
alias cnpm="npm --registry=https://registry.npm.taobao.org \
--cache=$HOME/.npm/.cache/cnpm \
--disturl=https://npm.taobao.org/dist \
--userconfig=$HOME/.cnpmrc"

# Or alias it in .bashrc or .zshrc
$ echo '\n#alias for cnpm\nalias cnpm="npm --registry=https://registry.npm.taobao.org \
  --cache=$HOME/.npm/.cache/cnpm \
  --disturl=https://npm.taobao.org/dist \
  --userconfig=$HOME/.cnpmrc"' >> ~/.zshrc && source ~/.zshrc
```
后面想安装某个模块时, 使用如下命令安装

```bash
$ cnpm install [name]
```

#### 安装hexo
> 安装hexo之前还需要安装[Node.js](https://nodejs.org/en/) 和 [Git](https://git-scm.com/)

首先在本地新建一个目录, 比如`~/workspace/myBlog`, 然后根据hexo文档https://hexo.io/zh-cn/docs/, 在该目录下执行

```bash
$ cnpm install -g hexo-cli
$ hexo init
```
启动hexo看看

```bash
$ hexo server
```
该命令会启动一个http服务, 访问http://localhost:4000/, 就能看到博客的样子了.
安装Git部署插件

```bash
$ cnpm install hexo-deployer-git --save
```

#### 更换成NexT主题
默认的landscape主题有点丑, 换成好看点的[NexT主题](https://theme-next.iissnan.com/). 
下载next主题文件 https://github.com/theme-next/hexo-theme-next/archive/master.zip, 然后解压到博客根目录下的`themes/next`目录下(next文件夹需要新建)
先`ctrl` + `c`停止http服务, 然后在博客根目录下执行

```bash
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```
编辑博客根目录下的`_config.yml`文件, 将`thmeme`配置改为`next`, 示例:

```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```
然后再启动http服务

```bash
$ hexo server
```
此时就能看的next主题下的样子了.
在首页, 默认所有文章是全文展示的, 可以改成显示摘要, 这样更加友好. 打开主题目录下的`_config.yml`文件, 如`themes/next/_config.yml`, 修改`auto_excerpt`属性为`true`,

```
auto_excerpt:
  enable: true
  length: 500
```
从注释来看, next推荐我们直接在文章里写`<!-- more -->`.

#### 配置
打开博客根目录下的`_config.yml`文件, 首先修改站点相关信息.

```
# Site
title: Tonny's Blog
subtitle: 莫等闲, 白了少年头, 空悲切
description: Coding, Life, Zen
author: Tonny Yi
language: zh-Hans
timezone: Asia/Shanghai
```
其次是站点以及文章url配置

```
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://codertang.com
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
```
其中`permalink`决定了每篇文章的访问地址, 比如博客里有篇文章, 叫`maven`, 且这篇文章是2018年3月30号写的, 根据上面的配置, 这篇文章的完整地址就是`http://codertang.com/2018/03/30/maven/`.
最后是配置站点部署信息, 也就是git仓库信息

```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:tonnyyi/tonnyyi.github.io.git
```

### 创建GitHub Pages
在GitHub上创建一个public仓库, 仓库名格式`你的用户名.github.io`, 如:`tonnyyi.github.io`.
在本地博客根目录执行如下命令,

```bash
$ git init
$ git add ./
$ git commit -m "first commit"
$ git checkout -b source
$ git branch -D master
$ hexo deploy
$ git remote set-url origin https://github.com/tonnyyi/tonnyyi.github.io.git
$ git push
```
上面的命令干了这么几件事: 首先将站点源文件提交到了`source`, 然后将站点页面文件部署到了`master`分支上(在上面的`Deployment`部分配置的), 最后把本地的`source`分支推动到了GitHub上. 把源文件也存到GitHub上之后, 如果要换台电脑写博客, 只需要将源码拉下来, 重新初始化一下环境, 一切数据就都回来了.
这是时候博客就可以在公网访问了, 地址是`你的用户名.github.io`, 如:`https://tonnyyi.github.io/`.

### 撰写文章并发布

### 绑定自定义域名
使用Github Pages不是博客, 默认域名是`你的用户名.github.io`, 有点长, 换成自己的域名. 首先得申请一个域名, 在国内申请的话需要实名认证, 还得备案, 麻烦. 找国外的一个域名服务商, 比如: 口碑最好的Namecheap,史上最便宜的Namesilo, 全球最大的Godaddy. 
在
