---
title: 本博客搭建:PicGo+GitHub作为图床
tags:
  - Blog
toc: true
date: 2021-04-20 10:32:37
categories:
  - Blog
---

博客的图床，又在白嫖 GitHub 了

## 参考链接

>[如何使用 Github 作为自己的免费图床](https://learnku.com/articles/48574)  
>[PicGo 文档:配置手册](https://picgo.github.io/PicGo-Doc/zh/guide/config.html)

## GitHub 仓库 Image  
在 GitHub 上新建仓库，仓库名为 Image (或自定义)，然后无需其他操作

## GitHub 配置 Personal access tokens  
在 GitHub 右上角头像单击，然后 `Settings -> Developer settings -> Personal access tokens -> Generate new token` 进入编辑页面  
Note: 名字可以随便填写，如 Image、Photo   
Select scopes: 选择 repo 即可  
然后点击按钮 Generate token 在跳转出来的页面， 将 Token 值 保存起来，后面 配置 PicGo 时用到  
<img src="https://raw.githubusercontent.com/altman-xu/altman-xu.github.io/source/ImageForBlog/GitHub_New_personal_access_token_1.png" width="50%" height="50%">

## PicGo 使用

1. 安装 PicGo
    ```zsh
    brew install --cask picgo
    ```

2. 配置 PicGo  
  打开 PicGo 软件， Mac 屏幕上方状态栏会出现 PicGo 小图标， `右击图标->打开详细窗口->图床设置->GitHub图床` (如下截图所示界面)  
  设定Token 一栏中填入 上面步骤生成的 Token 值
  <img src="https://raw.githubusercontent.com/altman-xu/altman-xu.github.io/source/ImageForBlog/PicGo%20setting.png" width="50%" height="50%">



3. 使用 PicGo  
  将待上传的图片拖动到 Mac 屏幕上方状态栏的 PicGo 小图标(会出现 + 添加图片的标志)，在 + 标志处松开鼠标即为上传  
  然后单击 PicGo 小图标，出现刚刚上传图片的 缩略图， 单击缩略图，即会复制图片链接   
  链接格式可以是 markdown/HTML/URL/UBB/Custom 等 在 `右击图标->打开详细窗口->上传区可设置`