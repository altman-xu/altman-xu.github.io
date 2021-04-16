---
title: '本博客搭建:Hexo+GithubPages+TraviCI'
date: 2021-04-16 17:57:26
tags: Hexo, TraviCI
---

# 本博客搭建:Hexo+GithubPages+TraviCI


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
