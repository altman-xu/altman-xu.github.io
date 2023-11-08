---
title: 本博客搭建(v2):Hexo-GithubPages-GithubAction
tags:
  - Blog
toc: true
date: 2023-11-08 16:29:26
categories:
  - Blog
---

>[本博客搭建:Hexo+GithubPages+TraviCI+pure](https://altman-xu.github.io/2021/04/19/Build-Blog-Hexo-GithubPages-TraviCI-Pure/)

## 背景
原有的 blog 搭建流程[本博客搭建:Hexo+GithubPages+TraviCI+pure](https://altman-xu.github.io/2021/04/19/Build-Blog-Hexo-GithubPages-TraviCI-Pure/) 随着 Travi CI 在 2020 年开始收费, 已经不可用. 

所以改用新的方式搭建博客

## 目标
1. 博客用一个仓库地址
  1. source 分支用来存放 项目源代码
  2. master 分支用来存放 hexo 编译 source 分支的文件后，生成的文件
2. 用户只需要编写 md 博客文档, 编译生成 html 的动作交由 cicd 流程
  1. 之前的 cicd 使用 travi ci
  2. 新方式改用为 GithubAction

## 说明

本文调整之前搭建流程的 ci 模块 [本博客搭建:Hexo+GithubPages+TraviCI+pure](https://altman-xu.github.io/2021/04/19/Build-Blog-Hexo-GithubPages-TraviCI-Pure/#Travis-CI-%E8%87%AA%E5%8A%A8%E5%8C%96%E9%83%A8%E7%BD%B2%E6%96%B9%E6%A1%88) 

## 配置

### 1. 本地生成 ssh 密钥对

在本地生成一对 SSH 密钥，注意更改文件名避免将正在使用的密钥覆盖。

```sh
ssh-keygen -t ed25519 -C "your_email@example.com"
```

在博客仓库的 Settings -> Secrets -> Actions 中添加 SSH 私钥，命名为 SSH_DEPLOY_KEY

![20231108-gitaction-1](https://raw.githubusercontent.com/altman-xu/Image/master/20231108-gitaction-1.png)