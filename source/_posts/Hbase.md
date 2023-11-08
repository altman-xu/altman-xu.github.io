---
title: Hbase
tags:
  - Hbase
toc: true
date: 2021-06-23 11:13:25
categories:
---

## MacOS 本机安装运行 hbase 单机

```zsh
## 安装命令
brew install hbase
## 查看版本
brew info hbase
## 运行命令
brew services start hbase
## 查看运行状态
brew services list
## 进入 hbase shell 终端
hbase shell  ## 此命令如果无法执行，则进入 /usr/local/Cellar/hbase/2.4.3/bin 目录 执行 sh ./hbase shell
```

查看版本信息截图如下:

![](https://raw.githubusercontent.com/altman-xu/altman-xu.github.io/source/ImageForBlog/20210623111930.png)

## MacOS 连接远程 hbase 集群

```zsh
## 通过配置 ZK 地址 来连接对应远程的 hbase 集群
cd /usr/local/Cellar/hbase/2.4.3/libexec/conf
vim hbase-site.xml
## 在这个文件的 <configuration> 节点下添加如下子节点
## zk:2181,zk:2181,zk:2181 为集群的 zk 连接地址
<property>
    <name>hbase.zookeeper.quorum</name>
    <value>zk:2181,zk:2181,zk:2181</value>
</property>

## 然后执行命令连接远程 hbase, 可执行 list 命令查看所有表，验证是否连接到远程 hbase
hbase shell
```



## Hbase 命令

```zsh
## 查看所有表
list

## 查看某个表结构，例如 scores
describe "scores"

## 删除表 scores , 删除需要先 disable 再 drop
disable "scores"
drop "scores"

## 删除 匹配表
drop_all 't.*'

## 创建表 scores 表，包含两个列族 grade 、 course
create 'scores','grade','course'

## put 命令往表添加数据  
## 格式: put 't1', 'r1', 'c1', 'value', ts1   
## t1指表名，r1指行键名，c1指列名，value指单元格值。ts1指时间戳，一般都省略掉了
## 一次 put 只能往表里面 put 一个单元格数据
put 'scores', 'Altman', 'grade:', '5'
put 'scores', 'Altman', 'course:math', '99'
put 'scores', 'Altman', 'course:english', '98' ## course 列族包含多个列，需要 列族:列 指定对应的列名
put 'scores', 'Tom', 'grade', '5'              ## grade 列族就只有一列 故 : 可以省略

## get 命令查询表数据
get 'scores', 'Altman'

get 'scores', 'Altman', 'grade'
get 'scores', 'Altman', 'grade', 'course:math', 'course:english'
get 'scores', 'Altman', ['grade', 'course:math', 'course:english']

get 'scores', 'Altman', {COLUMN => 'grade'}
get 'scores', 'Altman', {COLUMN => ['grade','course:math', 'course:english']}

get 'scores', 'Altman', {COLUMN => 'grade', TIMESTAMP => 1624436235582}   ## 时间错为毫秒格式
get 'scores', 'Altman', {COLUMN => 'grade', TIMESTAMP => 1624436235582, VERSIONS => 4}

## scan 命令扫描所有数据, 可以加限定词 TIMERANGE, FILTER, LIMIT, STARTROW, STOPROW, TIMESTAMP, MAXLENGTH, COLUMNS
scan 'scores'
scan 'scores', {COLUMNS => ['grade', 'course:math'], LIMIT => 10, STARTROW => 'Altman'}
scan 'scores', {COLUMNS => ['grade'], TIMERANGE => [1624436235582, 1624436335582]}
scan 'scores', {FILTER => "(PrefixFilter('Al') AND (QualifierFilter(>=, 'binary:xzy'))) AND (TimestampsFilter(123,456))"}

## delete 删除指定数据
delete 't1', 'r1', 'c1', ts1
delete 'scores', 'Altman', 'grade'
## deleteall 删除整行数据 慎用
deleteall 'scores', 'Altman'
## truncate 命令删除全表 , 这个命令其实是 disable drop create 三个命令组合出来的
truncate 'scores'

## alter 修改表结构  若 alter 命令执行报错，则先 disable 'scores', 再执行 alter 语句, 最后执行 enable 'scores'
### a. 添加一个列族 info
alter 'scores', NAME=>'info'

### b. 修改一个列族 info
alter 'scores', NAME=>'info', VERSIONS => 5

### c. 删除一个列族 info
alter 'scores', NAME => 'info', METHOD => 'delete' 
alter 'scores', 'delete' => 'info'

### d. 修改表属性 MAX_FILESIZE, MEMSTORE_FLUSHSIZE, READONLY, DEFERRED_LOG_FLUSH
alter 'scores', METHOD => 'table_att', MAX_FILESIZE => '134217728'

### e. 添加一个表协同处理器   一个表上可以配置多个协同处理器，一个序列会自动增长进行标识。加载协同处理器（可以说是过滤程序）需要符合以下规则：[coprocessor jar file location] | class name | [priority] | [arguments]
alter 'scores', METHOD => 'table_att', 'coprocessor' => ‘hdfs:///foo.jar|com.foo.FooRegionObserver|1001|arg1=1,arg2=2′

### f. 移除 coprocessor
alter 'scores', METHOD => 'table_att_unset', NAME => 'coprocessor$1'

### g. 可以一次执行多个 alter 命令
alter 'scores', {NAME => 'new_coloumn'}, {NAME => 'info', METHOD => 'delete'}

## count 命令统计行数  count一般会比较耗时，使用mapreduce进行统计，统计结果会缓存，默认是10行。统计间隔默认的是1000行（INTERVAL）
count 'scores'
count 'scores', INTERVAL => 100000
count 'scores', CACHE => 1000
count 'scores', INTERVAL => 100000, CACHE => 1000

## disable 表
disable 'scores'

## disable_all 正则匹配
disable_all 't.*'

## is_disabled 
is_disabled 'scores'

## help 查看帮助文档
help               
help 'alter'        ## 查看 alter 命令的帮助
help 'create'       ## 查看 create 命令的帮助

## hbase shell 脚本  将一些hbase shell 命令写到一个文件 cmd.txt ，然后顺序执行
hbase shell cmd.txt
```


## 参考资料
[使用 Shell 访问](https://help.aliyun.com/document_detail/52056.html)
[HBase shell的基本用法](https://www.cnblogs.com/ggjucheng/p/3379607.html)
[Apache HBase ™ Reference Guide v2.4](https://hbase.apache.org/2.4/book.html)
[Learn HBase](https://learnhbase.wordpress.com/)