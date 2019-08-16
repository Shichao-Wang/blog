---
title: Hexo博客搭建与Github Pages部署
abbrlink: b50f7193
date: 2019-08-16 18:18:24
tags:
---

最近两天用Hexo和Github Pages给自己搭建了一个博客，希望能够记录一些科研方面的东西吧。感觉内容的主要都是论文分析和代码实现。希望自己能够坚持吧。

最后搭建的成果是建立的一个名为`blog`的代码仓库，其中`master`分支存储了我们博客的所有源码，`gh-pages`分支存储了我们发布的Github Pages，将所有的源码存储在`master`分支是为了，方便我们在不同的机器上发布博客。同时，配合`Travis`这个**持续集成工具**，我们的写作流程就变成了：clone`master`分支的代码；撰写；push到github仓库。`Travis`能够自动检测`master`分支上的push操作，并将新的内容发布到`gh-pages`分支。

现在就让我们开始吧！

# `Hexo` 安装与基本使用

Hexo是什么就不多介绍了，相信能够看到这篇文章的都是搜索Hexo进来的

首先我们打开`Hexo`的[官方网站](https://hexo.io/zh-cn/index.html)，滚到下方我们就看到5行代码

```bash
npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo server
```

是的，敲入这五行代码后你就能看到

![image-20190816191622354](http://ww1.sinaimg.cn/large/006tNc79ly1g61qhcivo6j30ze03cgmm.jpg)

打开http://localhost:4000之后你就能看到你的博客首页啦

Hexo的其他操作，还请访问[官方文档](https://hexo.io/zh-cn/docs/)

# 博客页面发布

如果不需要将博客源码一并保存的话，此时可以使用`hexo-deployer-git`工具来部署博客，详见https://hexo.io/zh-cn/docs/deployment.html#Git

而我们需要保存博客源码，并且使用`Travis`将静态页面自动部署到`gh-pages`分支。此时我选择的部署工具是`Travis`自带的Github Pages部署工具。

## 配置Travis

`Travis`与`Github`的集成较好，我们只需要登录`Travis CI`的官网，使用Github登录并将对应`repo`的开关打开即可。

![image-20190816193618791](http://ww2.sinaimg.cn/large/006tNc79ly1g61r22bkawj310g0h275x.jpg)

我们在项目中加入一个名为`.travis.yml`的文件，并将以下内容加入即可

```yaml
language: node_js # 使用的语言为javaScript
node_js: stable # node_js 的版本

branches: # 监视的分支，此处为master
  only:
    - master

cache: # 缓存的目录，目的是为了加快部署速度
  directories:
    - node_modules

before_install:
  - npm install -g hexo-cli # 安装 hexo

install:
  - npm install # 安装 package.json 中的插件
  - hexo generate # 生成静态文件

deploy: # 部署
  provider: pages
  local_dir: public/ # 静态文件目录，hexo的静态文件默认生成在public目录下
  skip_cleanup: true  
  github_token: $GITHUB_TOKEN # github 密钥 部署时需要该密钥来认证身份，下文介绍如何生成
  keep_history: true
  on:
    branch: master
  
```



> **为什么我们要用`blog`作为`repo`的名字，而不是`<username>.github.io`呢？**
>
> 因为`Github`对于一般的`repo`支持在`master`、`gh-pages`、`/docs folder in master`三种发布方式，但是对于以`<username>.github.io`的`repo`只支持从`master`发布，而我们需要修改的是博客源码，因此将其放在`master`分支比较合适，因此我们不适用以用户名为开头的`repo`名
>
> 但是同时我们的博客地址也变成了`<username>.github.io/<repo-name>`，变成了发布在一个页面的子目录下，因此hexo中也要做出相应的修改。见下文[Hexo博客目录修改](#Hexo博客目录修改)
>
> https://help.github.com/en/articles/configuring-a-publishing-source-for-github-pages



# 生成Github密钥（Github Token）

生成Token的页面入口在：左上角头像`settings`——左侧`Develop settings`——左侧`Personal access token`

单击`New personal access token`填入Note备注信息并勾选`repo`下的几项权限即可

![image-20190816195954226](http://ww1.sinaimg.cn/large/006tNc79ly1g61rqlnu9vj31g00t2dld.jpg)

# 将Github Token传递给`Travis`

打开`Travis CI`的页面，进入对应`repo`的设置页面，在下方`Environment Variables`下输入对应的键值对，此处我们的键为`GITHUB_TOKEN`，值为生成的`token`单击ADD后`Travis`就能够使用了

![image-20190816200443727](http://ww2.sinaimg.cn/large/006tNc79ly1g61rvm72wyj322g0ia777.jpg)



# Hexo博客目录修改

因为我们使用非用户名开头的`repo`名，我们的Github页面被部署在了`<username>.github.io/<repo-name>`这个子目录下我们在`Hexo`的配置也需要做一些修改，此处假设我们的`repo`名为`blog`

```yaml
# _config.yml

url: http://<username>.github.io/blog
root: /blog/
```

