---
title: Elasticsearch-Install
tags:
  - Elasticsearch
toc: true
date: 2021-07-21 10:50:09
categories:
---

## brew 单机安装

```zsh
brew install elasticsearch
brew services start elasticsearch
brew services list
```

## 单机安装 + 插件

1. 下载 https://www.elastic.co/cn/downloads/elasticsearch 得到 elasticsearch-7.12.0-darwin-x86_64.tar.gz
2. 解压压缩包，得到 elasticsearch-7.12.0
3. cd /Users/zhihua.xu/ALTMAN_WORKSPACE/distributed_workspace/elasticsearch-7.12.0/bin   以下命令都是在此目录下执行
4. 执行 ./elasticsearch 即可启动本机 elasticsearch 服务， 通过 web 页面查看服务 http://localhost:9200
5. 查看本机已安装的elasticsearch插件  ./elasticsearch-plugin list
6. 安装插件:   ./elasticsearch-plugin install analysis-icu
7. 重启 elasticsearch 服务， 通过web页面查看插件:  http://localhost:9200/_cat/plugins


## 集群安装

1. 下载 https://www.elastic.co/cn/downloads/elasticsearch 得到 elasticsearch-7.12.0-darwin-x86_64.tar.gz
2. 解压压缩包，得到 elasticsearch-7.12.0
3. cd /Users/zhihua.xu/ALTMAN_WORKSPACE/distributed_workspace/elasticsearch-7.12.0/bin   以下命令都是在此目录下执行

启动命令:
./elasticsearch -E node.name=node1 -E cluster.name=altman_cluster_test -E path.data=node1_data -d
./elasticsearch -E node.name=node2 -E cluster.name=altman_cluster_test -E path.data=node2_data -d
./elasticsearch -E node.name=node3 -E cluster.name=altman_cluster_test -E path.data=node3_data -d

命令说明:  node.name 指定节点名字； cluster.name 指定集群名字； path.data 指定文件名字； -d 后台运行
打开浏览器验证 http://localhost:9200/ 里面可以看到 cluster_name 信息
http://localhost:9200/_cat/nodes 查看集群节点信息

删除进程 
ps | grep elasticsearch
kill pid

## 参考资料
[Elasticsearch 安装](https://www.qikegu.com/docs/3049)