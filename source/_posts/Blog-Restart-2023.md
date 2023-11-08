---
title: 博客重启
tags:
  - Blog
toc: true
date: 2023-10-25 16:10:10
categories:
  - Blog
---

## 重新开始

重新打开我的 [博客git仓库](https://github.com/altman-xu/altman-xu.github.io), 发现上一次编辑是 2021.07.22, 已经两年多了. 重新开始



|         | blog 网页地址 | git 仓库地址                          |
|---------|---------------|---------------------------------------|
| old     |     https://altman-xu.github.io/xuzhihua/         | https://github.com/altman-xu/xuzhihua |
| new     |     https://altman-xu.github.io/          |         https://github.com/altman-xu/altman-xu.github.io                              |


## 操作

```sh
## 1. 安装 hexo
brew install hexo
## 2. 在 ~/Documents 目录下 clone 博客仓库
cd ~/Documents
git clone git@github.com:altman-xu/altman-xu.github.io.git
## 3. 进入 altman-xu.github.io 目录, 切换到 source 分支
cd altman-xu.github.io
git checkout source
## 4. 启动 hexo
hexo s
## 5. 提示启动失败
ERROR Cannot find module 'hexo' from '/Users/zhihua.xu/Documents/altman-xu.github.io'
ERROR Local hexo loading failed in ~/Documents/altman-xu.github.io
ERROR Try running: 'rm -rf node_modules && npm install --force'

## 执行提示的命令
rm -rf node_modules && npm install --force
## 6. 再次启动 hexo
hexo s
## 成功, 然后退出

## 7. 创建新文档编辑
hexo new "My New Post"

## 8. 查看效果
hexo s
## 点击 http://localhost:4000 进入效果页面

## 9. 提交 source 分支改动, 远程会 TraviCI 自动编译到 master 分支
```