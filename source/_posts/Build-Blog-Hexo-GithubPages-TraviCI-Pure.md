---
title: '本博客搭建:Hexo+GithubPages+TraviCI+pure'
date: 2021-04-19 11:50:43
tags: 
    - Hexo
    - TraviCI
categories: # 测试使用 主要使用 tag区分
  - 博客
toc: true # 是否启用内容索引 (每个文字页面下面的 Catalogue/文章目录)
---

## 最终目的流程

1. 在本地hexo仓库新建博客文件 `Hexo new "new-blog-post-name"`
2. 在本地疯狂编辑博客内容 `new-blog-post-name.md`
3. push 本地 hexo 仓库到 github 上
4. 远程 Travis CI 自动触发 CI, 更新 GitHub Pages 内容
5. 访问浏览器查看到新内容


## 本地环境

>笔者本机电脑: macOS Catalina  Version 10.15.7

1. Git
    ```shell
    ## 安装: Mac电脑自带 Git,若无请手动安装  
    ## 版本
    ➜  git --version
    git version 2.24.3 (Apple Git-128)
    ```

2. node
    ```shell
    ## 安装: 
    ➜  brew install node
    ## 版本
    ➜  git --version
    git version 2.24.3 (Apple Git-128)
    ```

3. npm
    ```shell
    ## 安装: 
    ➜  brew install node
    ## 版本
    ➜  npm -v
    7.7.6
    ```

4. hexo
    ```shell
    ## 安装
    ➜  npm install -g hexo-cli
    ➜  npm install -g hexo
    ## 版本
    ➜  ~ hexo -version
    hexo-cli: 4.2.0
    ```

## 初始化 Hexo 站点



## 插件

### [hexo-wordcount](https://github.com/willin/hexo-wordcount)
作用: 字数统计、阅读时长预计

```zsh
npm install hexo-wordcount --save
```

### [hexo-generator-json-content](https://github.com/alexbruno/hexo-generator-json-content)
作用: 站内搜索

```zsh
npm install hexo-generator-json-content --save
```

### [hexo-generator-feed](https://github.com/hexojs/hexo-generator-feed)
作用: Generate Atom 1.0 or RSS 2.0 feed

```zsh
npm install hexo-generator-feed --save
```

### [hexo-generator-sitemap](https://github.com/hexojs/hexo-generator-sitemap)
作用: Generate sitemap -- 针对谷歌
使用站图的初衷是为自己的博客添加站内搜索,如果想更好的发挥站图的作用
```zsh
npm install hexo-generator-sitemap --save
```

### [hexo-generator-sitemap](https://github.com/hexojs/hexo-generator-sitemap)
作用: Generate sitemap -- 针对百度
使用站图的初衷是为自己的博客添加站内搜索,如果想更好的发挥站图的作用，建议手动提交baidusitemap给百度.

```zsh
npm install hexo-generator-baidu-sitemap --save
```



## 问题说明

### 新建博客文章
hexo new "postName"       新建文章时候，命令里面使用 全英文
然后在 _post 里面对应的 md 里面的 title， 可以使用想用的中文，避免gittalk转链接和长度限制错误


> [[gitalk] 解决配置gitalk插件后初始化登录时跳转回首页](https://blog.csdn.net/w47_csdn/article/details/88858343)