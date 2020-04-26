# Redis-HA-Windows-Config
Windows系统配置Redis集群高可用（开发）

一 所需软件:Redis、Ruby语言运行环境、Redis的Ruby驱动redis-xxxx.gem、创建Redis集群的工具redis-trib.rb

二 安装配置redis 

redis下载地址   https://github.com/MSOpenTech/redis/releases

集群规划有三个节点的集群，每个节点有一主一备。需要6台虚拟机。

把 redis 解压后，再复制出 5 份，配置 三主三从集群。 由于 redis 默认端口号为 6379，那么其它5份的端口可以为6380，6381，6382，6383，6384。 并且把目录使用端口号命名

 打开目录6379下有一个文件 redis.windows.conf，修改里面的端口号，以及集群支持配置。

修改其他配置支持集群

cluster-enabled yes

cluster-config-file nodes-6379.conf

cluster-node-timeout 15000

appendonly yes

如果cluster-enabled 不为yes， 那么在使用JedisCluster集群代码获取的时候，会报错。

cluster-node-timeout 调整为  15000，那么在创建集群的时候，不会超时。

cluster-config-file nodes-6379.conf 是为该节点的配置信息，这里使用 nodes-端口.conf命名方法。服务启动后会在目录生成该文件。

编写一个 bat 来启动 redis，在每个节点目录下建立 start.bat，内容如下：

title redis-6380

redis-server.exe redis.windows.conf

 

三 安装Ruby

redis的集群使用  ruby脚本编写，所以系统需要有 Ruby 环境 ,下载地址 http://dl.bintray.com/oneclick/rubyinstaller/:rubyinstaller-2.3.3-x64.exe

安装时3个选项都勾选。

 

四 安装Redis的Ruby驱动redis-xxxx.gem

下载地址 https://rubygems.org/pages/download， 下载后解压，当前目录切换到解压目录中，如 D:\Program Files\redis\rubygems-2.6.12 然后在命令行执行  ruby setup.rb。

 

然后GEM 安装 Redis ：切换到redis安装目录，需要在命令行中，执行 gem install redis

 

五 安装集群脚本redis-trib

下载地址  https://raw.githubusercontent.com/antirez/redis/unstable/src/redis-trib.rb

 打开该链接如果没有下载，而是打开一个页面，那么将该页面保存为redis-trib.rb，建议保存到一个Redis的目录下，例如放到6379目录下。

集群的命令为 

redis-trib.rb create --replicas 1 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384

--replicas 1 表示每个主数据库拥有从数据库个数为1。master节点不能少于3个，所以我们用了6个redis

 

六 启动每个节点并且执行集群构建脚本

把每个节点下的 start.bat双击启动， 在切换到redis目录在命令行中执行   redis-trib.rb create --replicas 1 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384

备注：有朋友反应上面的语句执行不成功。可以在前面加上ruby再运行。

 

ruby redis-trib.rb create --replicas 1 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384

 


在出现 Can I set the above configuration? (type 'yes' to accept):   请确定并输入 yes 。成功后的结果如下：

 

七测试

使用Redis客户端Redis-cli.exe来查看数据记录数，以及集群相关信息

命令 redis-cli –c –h ”地址” –p "端口号" ;  c 表示集群

输入dbsize查询 记录总数

输入cluster info可以从客户端的查看集群的信息

 

=====================================

注意：如果出现创建集群不成功：

dos命令窗口执行创建集群命令，出现以下提示：
复制代码

D:\redis\6379>redis-trib.rb create --replicas 1 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384
WARNING: redis-trib.rb is not longer available!
You should use redis-cli instead.

All commands and features belonging to redis-trib.rb have been moved
to redis-cli.
In order to use them you should call redis-cli with the --cluster
option followed by the subcommand name, arguments and options.

Use the following syntax:
redis-cli --cluster SUBCOMMAND [ARGUMENTS] [OPTIONS]

Example:
redis-cli --cluster create 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 --cluster-replicas 1

To get help about all subcommands, type:
redis-cli --cluster help

复制代码

原因是redis-trib.rb的链接指向官网最新的版本。从对应版本（redis3.2.0即可）的源码压缩包中src文件夹下找到对应的redis-trib.rb文件使用，即可解决问题。

下载redis源码版压缩包：http://download.redis.io/releases/

至此，redis集群搭建成功，可在项目中测试redis集群
