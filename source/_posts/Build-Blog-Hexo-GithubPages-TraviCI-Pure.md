---
title: '本博客搭建:Hexo+GithubPages+TraviCI+pure'
date: 2021-04-19 11:50:43
tags: 
    - Hexo
    - TraviCI
hexo new page "postArticleName" # 新建页面
categories: # 测试使用 主要使用 tag区分
  - 博客
toc: true # 是否启用内容索引 (每个文字页面下面的 Catalogue/文章目录)
---

## 参考链接
> [Hexo 文档](https://hexo.io/zh-cn/docs/)  
> [Travis CI 加 Hexo 实现自动构建部署 Github Pages 博客](https://segmentfault.com/a/1190000021987832)  
> [将 Hexo 部署到 GitHub Pages](https://hexo.io/zh-cn/docs/github-pages)  
> [Hexo 集成 Disqus 评论](https://www.cylong.com/blog/2017/03/26/hexo-next-disqus/)

## 最终目的流程

1. 在本地hexo仓库新建博客文件 `Hexo new "new-blog-post-name"`
2. 在本地疯狂编辑博客内容 `new-blog-post-name.md`
3. 在本地部署查看效果 `hexo s` 然后访问 http://localhost:4000/ 查看最新文章在本地效果是否满意(此步骤可省略)
4. push 本地 hexo 仓库到 github 上
5. 远程 Travis CI 自动触发 CI, 将源码 md 文件通过 hexo 编译为 html文件, 更新 GitHub Pages 内容
6. 访问浏览器查看到新内容

## 本地环境说明

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

## Hexo 本地初始化站点

```zsh
cd /Users/zhihua.xu/Documents/       ## 笔者将站点放在 Documents 目录下， 这个可以每个人自定义
hexo init altman-xu.github.io       ## hexo 站点名字， 起名为这个是因为后面 github 仓库名也是这个
```

>修改 站点的 _config.yml
>最主要就修改下面这些数据, 注意使用的主题改为 pure

```yml
title: Altman's Blog
subtitle: ''
description: ''
keywords:
author: Altman
language: en     ## zh-CN en
timezone: 'Asia/Shanghai'

url: https://altman-xu.github.io/

theme: pure
```


## pure 站点主题
> [theme-suka](https://theme-suka.skk.moe/docs/)    [Sukka's Blog](https://blog.skk.moe/)  
> [theme-next](https://theme-next.iissnan.com/)     [展示](https://notes.iissnan.com/)  
> [theme-pure](https://github.com/cofess/hexo-theme-pure)  [展示](https://blog.cofess.com/)  

对比了这三个主题，最终还是选择 pure 个人比较喜欢

```zsh
# 当前目录为 hexo 站点根目录: /Users/zhihua.xu/Documents/altman-xu.github.io
git clone https://github.com/cofess/hexo-theme-pure.git themes/pure
cd themes/pure
git pull


## 启动本地 hexo 服务
hexo s --debug   ## 部署站点，然后访问 http://localhost:4000/ 查看站点效果
```

>接着 参照 [Hexo博客主题pure使用说明](https://blog.cofess.com/2017/11/01/hexo-blog-theme-pure-usage-description.html#collapseToc) 博客内容修改 pure 目录下的 _config.yml 文件
>pure 目录下的 /source/images 部分图片也替换成自己的
>在修改过程，有些修改重刷页面即可看到效果，有些需要重启服务


### 本地安装插件说明

1. [hexo-wordcount](https://github.com/willin/hexo-wordcount)  
    作用: 字数统计、阅读时长预计
    ```zsh
    npm install hexo-wordcount --save
    ```
2.  [hexo-generator-json-content](https://github.com/alexbruno/hexo-generator-json-content)  
    作用: 站内搜索
    ```zsh
    npm install hexo-generator-json-content --save
    ```
3. [hexo-generator-feed](https://github.com/hexojs/hexo-generator-feed)  
    作用: Generate Atom 1.0 or RSS 2.0 feed
    ```zsh
    npm install hexo-generator-feed --save
    ```
4. [hexo-generator-sitemap](https://github.com/hexojs/hexo-generator-sitemap)  
    作用: Generate sitemap -- 针对谷歌  
    使用站图的初衷是为自己的博客添加站内搜索,如果想更好的发挥站图的作用
    ```zsh
    npm install hexo-generator-sitemap --save
    ```
5. [hexo-generator-sitemap](https://github.com/hexojs/hexo-generator-sitemap)  
    作用: Generate sitemap -- 针对百度
    使用站图的初衷是为自己的博客添加站内搜索,如果想更好的发挥站图的作用，建议手动提交baidusitemap给百度.
    ```zsh
    npm install hexo-generator-baidu-sitemap --save
    ```

### 本地pure主题修改
>pure 目录下的 /source/images 部分图片也替换成自己的
其余修改参照 [Hexo博客主题pure使用说明](https://blog.cofess.com/2017/11/01/hexo-blog-theme-pure-usage-description.html#collapseToc)

### 评论设置
之前想的是使用 [gitalk](https://github.com/gitalk/gitalk) 作为评论方案，可以用github登录更 programer, 但是遇到太多问题，故还是使用国外的 Disqus  

> [gitalk Error: Not Found.](https://blog.csdn.net/qing_gee/article/details/100133060)
> [gitalk 解决配置gitalk插件后初始化登录时跳转回首页](https://blog.csdn.net/w47_csdn/article/details/88858343)
> [gitalk 评论登录 403 问题解决](https://cuiqingcai.com/30010.html)

1. 现在 Disqus[https://disqus.com/] 上新建一个站点名称为 altman-xu.github.io  可参照这个 [Hexo 集成 Disqus 评论](https://www.cylong.com/blog/2017/03/26/hexo-next-disqus/) 
2. 修改 pure 下的 _config.yml 即可
    ```yml
    comment:
    type: disqus  # 启用哪种评论系统        ## gitalk proxy 频频报错，故改用 disqus
    disqus: altman-xu-github-io # enter disqus shortname here
    ```

## GitHub Pages

在 github 上新建仓库, 仓库名为 `username.github.io`

```zsh
# 当前目录为 hexo 站点根目录
# 添加 Github 仓库到本地
git remote add origin https://github.com/altman-xu/altman-xu.github.io.git
# 编辑 .gitignore 文件, 如果没有，则手动创建这个文件， 文件内容如下， 注 一定不要有 themes/ 即我们要把这个文件夹下所有内容添加到 git 上(因为我们使用的 pure 主题，修改了源文件，也不想在 ci 过程中重新去 github 拉取)

.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/

## 去除 themes/pure 的 git 关联，改内容是从 git clone 过来的，现在我们改了内容，要 push 到我们自己的仓库中, 先将整个 themes/pure 文件备份一份，避免接下来操作出现问题
rm -rf themes/pure/.git
rm -rf themes/pure/.gitignore
git rm --cached themes/pure/ -f     ## 干掉 submodule 关联，不执行这一步， 下面 push 时候会提示 fatal: in unpopulated submodule 这个错误

# 新建一个名为 source 的分支
git checkout -b source
# 将所有文件添加到 git
git add .
# 添加 commit
git commit -m "initial"
# 将本地的文件推送到 Github 上的 source 分支
git push -u origin source

## 注意去远程仓库看 themes/pure/ 下有没有内容
```

## Travis CI 自动化部署方案

1. 将 [Travis CI](https://github.com/marketplace/travis-ci) 添加到你的 Github 账户 , 添加 Open Source, 其他都要收费
2. 去 [Applications settings](https://github.com/settings/installations) 设置让 Travis CI 能够访问你的 repo
3. 去 [Personal access tokens](https://github.com/settings/tokens) 为 Travis CI 新建一个 token ( 只需要 repo 这个 scopes )，然后把 token 的值记录下来
4. 去 [Travis CI](https://travis-ci.com/)，在你的 repo 页面下点击 More Options 找到 Settings， 然后在 Environment Variables 下新建一个环境变量，Name 为 GH_TOKEN，Value 为刚才你在 GitHub 生成的 Token。确保 DISPLAY VALUE IN BUILD LOG 保持 不被勾选 避免你的 Token 泄漏。点击 Add 保存
5. 在你的 Github 的项目 source 分支内新建一个名为 .travis.yml 的文件，参考以下内容进行填入

    ```yml
    os: linux
    language: node_js 
    node_js:
    - 10  # 使用 nodejs LTS v10
    branches:
    only:
        - source # 只监控 source 的 branch
    cache:
    directories:
        - node_modules # 缓存 node_modules 加快构建速度
    before_script: ## 根据你所用的主题和自定义的不同，这里会有所不同
    # 使用 themes/suka 是使用的 ci 脚本 begin
    # - npm install -g hexo-cli # 在 CI 环境内安装 Hexo
    # - mkdir themes # 由于我们没有将 themes/ 上传，所以我们需要新建一个
    # - cd themes 
    # - git clone https://github.com/SukkaW/hexo-theme-suka.git suka #从 Github 上拉取 Suka 主题
    # - cd suka
    # - npm install --production # 安装 Suka 主题的依赖
    # - cd ../.. # 返回站点根目录
    # - cp _config.theme.yml themes/suka/_config.yml # 将主题的配置文件放回原处    
    # - npm install # 在根目录安装站点需要的依赖 
    # 使用 themes/suka 是使用的 ci 脚本 end

    # 使用 themes/pure 是使用的 ci 脚本 begin       ## 注: 与 suka 最大的区别是 themes 不从 Github 上拉取，而是使用自己仓库里的 原因是修改了一些源码和图片
    - npm install -g hexo-cli # 在 CI 环境内安装 Hexo
    - npm install hexo-wordcount --save                 # 安装插件 hexo-wordcount
    - npm install hexo-generator-json-content --save    # 安装插件 hexo-generator-json-content
    - npm install hexo-generator-feed --save            # 安装插件 hexo-generator-feed
    - npm install hexo-generator-sitemap --save         # 安装插件 hexo-generator-sitemap
    - npm install hexo-generator-baidu-sitemap --save   # 安装插件 hexo-generator-baidu-sitemap
    - npm install # 在根目录安装站点需要的依赖 
    # 使用 themes/pure 是使用的 ci 脚本 end
    script: 
    - hexo generate # generate static files
    deploy: # 根据个人情况，这里会有所不同
    provider: pages
    skip_cleanup: true # 构建完成后不清除
    token: $GH_TOKEN # 你刚刚设置的 token
    keep_history: true # 保存历史
    # fqdn: blog.ne0ng.page # 自定义域名，使用 username.github.io 可删除
    on:
        branch: source # hexo 站点源文件所在的 branch
    local_dir: public 
    target_branch: master # 存放生成站点文件的 branch，使用 username.github.io 必须是 master
    ```
6. 将 .travis.yml 推送到 repository 中。Travis CI 应该会自动开始运行，并将生成的文件推送到同一 repository 下的 master 分支下
7. 在 GitHub 中前往你的 repository 的 Settings-Pages ，修改 GitHub Pages 的部署分支为 master
8. 在 GitHub 中前往你的 repository 的 Branches, 将 Default Branch 修改为 master 分支
9. 前往 https://altman-xu.github.io 查看你的站点是否可以访问。这可能需要一些时间。 也可以本地修改内容或 hexo new postArticle 然后推送git 重新触发 ci， 然后去 [Travis CI](https://travis-ci.com/) 上看构建情况 是否报错

## 部署方案另外的选择 hexo d
参照 [mac下搭建hexo+github](https://houxuefeng.com/2019/07/27/mac%E4%B8%8B%E6%90%AD%E5%BB%BAhexo-github/) 示例
通过 `hexo g && hexo d` 将生成的静态html文件部署上git上  
注: 这种方式没有将 源码文件也放到git上  
我们上面的 通过 Travis CI 方式，将源码文件和生成的静态html文件都放到git上,更优雅; 在另外的机器上重新搭建写hexo也更方便


## 问题说明

1. 文章名字建议用全英文  
    hexo new "postArticleName"      新建文章时候，命令里的 postArticleName 文章名字使用 全英文  
    然后在 _post 里面对应的 md 里面的 title， 可以使用想用的中文，避免gittalk转链接和长度限制错误


## hexo 常用命令

```zsh
hexo new "postArticleName"      # 新建文章
hexo new page "postArticleName" # 新建页面
hexo s                          # 简写 hexo server # 开启预览访问端口（默认端口4000，'ctrl + c'关闭server）,可边修改配置边预览
hexo g                          # 简写 hexo generate # 生成静态页面至public目录
hexo d                          # 简写 hexo deploy # 将.deploy目录部署到GitHub
hexo help                       # 查看帮助
hexo clean                      ## 清楚缓存文件(db.json)和已生成的静态文件(public)
hexo version                    # 查看Hexo的版本
```

## hexo 模板修改
修改站点目录下的 /scaffolds/post.md , 改为如下内容, 优化 `hexo new "postArticleName"` 新创建文章的初始化内容

```markdown
---
title: {{ title }}
date: {{ date }}
tags:       ## tags 多个的话, 分多行显示
    - tag1_PleaseDelete
    - tag2_PleaseDelete
categories: # 目录暂不启用, 现主要使用 tag区分
toc: true   # 是否启用内容索引 (每个文字页面下面的 Catalogue/文章目录)
---

## 

##
```