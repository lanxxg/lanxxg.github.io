---
title: 使用Gatsby搭建个人博客
date: "2019-04-11"
description: Steps to building a static blog.
---

[Gatsby](https://www.gatsbyjs.org/)是基于React的框架，可以帮助我们构建超快的网站与应用。使用[plugins](https://www.gatsbyjs.org/docs/plugins)可以很方便的实现很多功能，当然官方和社区也提供了很多预设置好的用例模板[starters](https://www.gatsbyjs.org/docs/starters/)，可以快速的构建应用。

> 用[Gatsby blog starter](https://github.com/gatsbyjs/gatsby-starter-blog)模板开始你的博客

## 安装Gatsby CLI
首先，全局安装`Gatsby`脚手架
```
  yarn global add gatsby-cli
  // or
  npm install -g gatsby-cli
```
## 创建项目
安装完之后，通过`gatsby new [rootPath] [starter]`命令新建`gatsby`项目
```
  // create a new Gatsby site using the blog starter
  gatsby new my-blog-starter https://github.com/gatsbyjs/gatsby-starter-blog
```

## 运行开发服务
然后进入`my-blog-starter`目录，运行开发服务器
```
  cd my-blog-starter
  gatsby develop
```
此时在浏览器打开`http://localhost:8000`，你会看到博客首页，编辑content目录下的博客并保存试试吧

## 构建应用
```
  gatsby build
```
Gatsby会打包所有资源，包括静态HTML、JavaScript等
## 部署
如何部署到[GitHub Pages](https://pages.github.com/)？可以通过[gh-pages](https://github.com/tschaub/gh-pages)依赖包把项目发布到`Github Pages`，运行下面命令安装：
```
npm install gh-pages --save-dev
```
### GitHub Repository page
在`package.json`文件中添加部署脚本
```
  {
    "scripts": {
      ...
      "deploy": "gatsby build --prefix-paths && gh-pages -d public"
    }
  }
```
在`gatsby-config.js`配置文件中添加一个路径前缀
```
  {
    pathPrefix: "/your-repo-name",
  }
```
然后运行`npm run deploy`，在浏览器打开`http://username.github.io/repo-name`查看

### GitHub Organization or User page
仓库以`username.github.io`方式命名
```
  {
    "scripts": {
        ...
        "deploy": "gatsby build && gh-pages -d public -b master",
    }
  }
```
然后运行`npm run deploy`，在浏览器打开`http://username.github.io`查看

胡说八道~