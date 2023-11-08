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
>[GitHub Actions 自动部署 Hexo 博客](https://blog.zhanganzhi.com/zh-CN/2022/06/0800d76d306e/)
>[GitHub Actions 入门教程](https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)
>[GitHub Actions 官方文档](https://docs.github.com/zh/actions/using-workflows/events-that-trigger-workflows#running-your-workflow-only-when-a-push-affects-specific-files)

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

### 本地生成 ssh 密钥对

在本地生成一对 SSH 密钥，注意更改文件名避免将正在使用的密钥覆盖。

```sh
ssh-keygen -t ed25519 -C "xuzhihua1107@gmail.com"
```

在博客仓库的 Settings -> Secrets -> Actions 中添加 SSH 私钥, 内容为刚刚生成的 id_ed25519 文件的秘钥值, 命名为 SSH_DEPLOY_KEY

![20231108-gitaction-1](https://raw.githubusercontent.com/altman-xu/Image/master/20231108-gitaction-1.png)

在部署仓库的 Settings -> Deploy keys 中添加 SSH 公钥, 内容为刚刚生成的 id_ed25519.pub 文件的公钥值, 命名为 public key of SSH_DEPLOY_KEY 注意勾选 Allow write access。

![20231108-gitaction-2](https://raw.githubusercontent.com/altman-xu/Image/master/20231108-gitaction-2.png)

> 注: 后续的 workflow 会使用 SSH_DEPLOY_KEY 公钥值来部署

### 在 _config.yml 添加内容

[_config.yml](https://github.com/altman-xu/altman-xu.github.io/blob/source/_config.yml)

```sh
deploy:
  type: git
  repo: git@github.com:altman-xu/altman-xu.github.io.git
  branch: master
```

### 编写 workflow

创建 [.github/workflows/githubactions.yml](https://github.com/altman-xu/altman-xu.github.io/blob/source/.github/workflows/githubactions.yaml)

写入以下内容，注意修改仓库地址和 Git 配置。此时当 push 到博客仓库时，GitHub Actions 将会自动部署

```sh
name: githubactions # yml文件名

env:
  GIT_USER: altman-xu
  GIT_EMAIL: xuzhihua1107@gmail.com
  DEPLOY_REPO: altman-xu/altman-xu.github.io
  DEPLOY_BRANCH: master 

on:
  push:
    branches:
      - source    # 仅当推送 source 分支时才运行工作流程

jobs:
  build-and-deploy: # 任务名
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source               # 下载 source 分支代码
        uses: actions/checkout@v2
        with: 
          ref: source
      - name: Setup Nodejs                  # 安装 node.js
        uses: actions/setup-node@v2
        with:
          node-version: '10'
      - name: Cache node modules            # 缓存 node
        id: cache-npm
        uses: actions/cache@v3
        env:
          cache-name: cache-node-modules
        with:
          # npm cache files are stored in `~/.npm` on Linux/macOS
          path: ~/.npm
          key: ${{ runner.os }}-build-${{ env.cache-name }}-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-build-${{ env.cache-name }}-
            ${{ runner.os }}-build-
            ${{ runner.os }}-

      - name: Debug Cache Hit
        run: echo "${{ steps.cache-npm.outputs.cache-hit }}"  # true 代表缓存命中 ~/.npm 没有变化; 而''代表没有命中, ~/.npm 发生变化

      - name: Install npm                                   # 只有结果是''时才执行
        if: ${{ steps.cache-npm.outputs.cache-hit != 'true' }}
        continue-on-error: true
        run: npm install                                    # 在根目录安装站点需要的依赖

      - name: Install npm hexo                              # 安装依赖
        # if: ${{ steps.cache-npm.outputs.cache-hit != 'true' }}  # 缓存暂时不能对 hexo 相关生效, 奇怪
        run: |
          npm install hexo-cli -g                           # 在 CI 环境内安装 Hexo
          npm install hexo-wordcount --save                 # 安装插件 hexo-wordcount
          npm install hexo-generator-json-content --save    # 安装插件 hexo-generator-json-content
          npm install hexo-generator-feed --save            # 安装插件 hexo-generator-feed
          npm install hexo-generator-sitemap --save         # 安装插件 hexo-generator-sitemap
          npm install hexo-generator-baidu-sitemap --save   # 安装插件 hexo-generator-baidu-sitemap
          npm install hexo-deployer-git --save              # 安装插件 用来执行 hexo deploy
      - name: Setup Git                   # 设置 git 环境变量
        run: |
          git config --global user.name $GIT_USER
          git config --global user.email $GIT_EMAIL
      - name: Setup SSH Key               # 设置 sshkey
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_DEPLOY_KEY }}" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          ssh-keyscan github.com >> ~/.ssh/known_hosts
      - name: hexo generate
        run: |
          hexo clean
          hexo generate
      - name: hexo deploy
        run: |
          hexo deploy
```

## cicd 执行

每次在 source 分支, 编写新博客 md 文件, 提交到 git 远程后，都会触发 cicd 流程, 如下图所示, 可以点击进去查看 cicd 执行的每个步骤

![20231108-gitaction-3](https://raw.githubusercontent.com/altman-xu/Image/master/20231108-gitaction-3.png)