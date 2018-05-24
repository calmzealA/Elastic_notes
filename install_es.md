### Elastic Search 安装

**目录 (Table of Contents)**

[TOCM]

[TOC]

#Linux版本：

##1. 安装JDK。
###1.1 下載 jdk解压。
###1.2 配置环境变量。
vi /etc/profile

export JAVA_HOME=/home/jdk1.8.0_172
export PATH=$JAVA_HOME/bin:$PATH 

source /etc/profile使改变生效。

##2. 安装es
cd /home
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.4.zip
unzip elasticsearch-6.2.4.zip
mv elasticsearch-6.2.4 elasticsearch

##3.创建用户
groupadd elsearch 
useradd elsearch -g elsearch -p elasticsearch
chown -R elsearch:elsearch  elasticsearch
chown -R elsearch:elsearch  /bigdata  

##4. 修改Elasticsearch 配置。
cd /home/elasticsearch/config
vi jvm.options
设置内存大小
-Xms8g
-Xmx8g
完整配置见附件

vi elasticsearch.yml
```yml
##集群的名称
cluster.name: warz-app   
##节点名称,其余三个节点分别为node-2,node-3,node-4
node.name: node-3
##指定该节点是否有资格被选举成为master节点，默认是true，es是默认集群中的第一台机器为master，如果这台机挂了就会重新选举master  
node.master: true
##允许该节点存储数据(默认开启)
node.data: true
##索引数据的存储路径  
path.data: /home/elasticsearch/data,/bigdata/esdata1,/bigdata/esdata2,/bigdata/esdata3,/bigdata/esdata4
##日志文件的存储路径  
path.logs: /home/elasticsearch/logs  
#绑定的ip地址  
network.host: 0.0.0.0  
#设置对外服务的http端口，默认为9200  
http.port: 9200
# 设置节点间交互的tcp端口,默认是9300   
transport.tcp.port: 9300

discovery.zen.ping.unicast.hosts: ["10.155.114.239:9300","10.153.116.100:9300"]  
##如果没有这种设置,遭受网络故障的集群就有可能将集群分成两个独立的集群 - 分裂的大脑 - 这将导致数据丢失  
discovery.zen.minimum_master_nodes: 2

bootstrap.memory_lock: false
bootstrap.system_call_filter: false
## elastichead插件的跨域设置
http.cors.enabled: true
http.cors.allow-origin: "*"
```

##5.开放端口

打开配置文件加入如下语句:

vi /etc/sysconfig/iptables
添加

-A INPUT -p tcp -m state --state NEW -m tcp --dport 9300 -j ACCEPT
重启防火墙，修改生效

service iptables restart

##6. 启动es。
su elsearch #切换账户
cd elasticsearch/bin #进入你的elasticsearch目录下的bin目录
./elasticsearch



##7.安装 es 遇到的问题
https://www.jianshu.com/p/4c6f9361565b

###1 ERROR: bootstrap checks failed
memory locking requested for elasticsearch process but memory is not locked
原因：锁定内存失败

解决方案： 
切换到root用户，编辑limits.conf配置文件， 添加类似如下内容： 
sudo vim /etc/security/limits.conf

添加如下内容: 
* soft memlock unlimited 
* hard memlock unlimited 
备注：* 代表Linux所有用户名称

###2 max number of threads [1024] for user [es] is too low, increase to at least [2048] 
原因：无法创建本地线程问题,用户最大可创建线程数太小 
解决方案：切换到root用户，进入limits.d目录下，修改90-nproc.conf 配置文件。

sudo vim /etc/security/limits.d/90-nproc.conf

找到如下内容： 
* soft nproc 1024

修改为 
* soft nproc 4096



###3  max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
原因：最大虚拟内存太小 
解决方案：切换到root用户下，修改配置文件sysctl.conf

sudo vim /etc/sysctl.conf

添加下面配置： 
vm.max_map_count=655360
fs.file-max=655360
vm.swappiness=1

并执行命令： 
sysctl -p

###4.max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]


###5. 安装es的head插件需要nodejs环境。
export NODE_HOME=/home/node
export PATH=$NODE_HOME/bin:$PATH

#Windows版本安装 TODO
