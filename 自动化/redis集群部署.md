一、在官方下载redis源码包，安装编译工具，进行编译安装。本次集群6个节点，分别在两台虚拟
  机上实现。每台机器上3个redis节点
  ]# wget http://download.redis.io/releases/redis-3.2.4.tar.gz
  ]# cd /opt && tar xvf  redis-3.2.4.tar.gz
  ]#  yum install gcc gcc-c++ -y
  ]#  cd redis-3.2.4
  ]#  make && make install

二、将redis集群工具拷贝到/usr/local/bin目录下
  ]#  cd src
  ]#  cp redis-trib.rb /usr/local/bin/

三、创建Redis节点，在两台虚拟机的redis-3.2.4的目录下，分别创建一个redis_cluster目录，
  并在该目录下分别创建三个目录,并把配置文件分别拷到这些目录下面
  ]#  mkdir -pv redis_cluster/{7000,7001,7002}
  ]#  cp redis.conf redis_cluster/7000
  ]#  cp redis.conf redis_cluster/7001
  ]#  cp redis.conf redis_cluster/7002
  ]#  cd redis_cluster/7000
  ]#  vim redis.conf
    bind 172.17.60.201    //修改为其他节点可以访问的IP地址
    port 7000  
    daemonize yes
    pidfile  /var/run/redis_7000.pid
    cluster-enabled yes
    cluster-config-file  nodes_7000.conf            
    cluster-node-timeout  15000
    appendonly  yes
  剩下5个节点做相应的配置

四、启动每个节点
  ]#  redis-server redis_cluster/7000/redis.conf
  ]#  redis-server redis_cluster/7001/redis.conf
  ]#  redis-server redis_cluster/7002/redis.conf
  ]#  redis-server redis_cluster/7003/redis.conf
  ]#  redis-server redis_cluster/7004/redis.conf
  ]#  redis-server redis_cluster/7005/redis.conf

  ]#  ps -ef |grep redis
  root      14034      1  0 00:41 ?        00:00:04 redis-server 172.17.60.201:7000 [cluster]
  root      14047      1  0 00:58 ?        00:00:03 redis-server 172.17.60.201:7001 [cluster]
  root      14051      1  0 00:58 ?        00:00:03 redis-server 172.17.60.201:7002 [cluster]

  ]# ps -ef |grep redis
  root      13223      1  0 00:59 ?        00:00:03 redis-server 172.17.60.208:7003 [cluster]
  root      13227      1  0 00:59 ?        00:00:03 redis-server 172.17.60.208:7004 [cluster]
  root      13231      1  0 00:59 ?        00:00:03 redis-server 172.17.60.208:7005 [cluster]

五、创建集群
  ]#  yum -y install ruby ruby-devel rubygems rpm-build
  ]#  gem install redis
  ]#  redis-trib.rb create --replicas 1 172.17.60.201:7000 172.17.60.201:7001 172.17.60.201:7002 172.17.60.208:7003 172.17.60.208:7004 172.17.60.208:7005


六、集群验证，连接第一台虚拟机上7002和第二台虚拟机的7005节点，在7005上创建一个键值对，
  在7002上去验证
  ]#  redis-cli -h 172.17.60.201 -c -p 7002
  ]#  redis-cli -h 172.17.60.201 -c -p 7005
  172.17.60.208:7005> SET hi haha
  -> Redirected to slot [16140] located at 172.17.60.201:7001
  OK
  172.17.60.201:7002> get hi
  -> Redirected to slot [16140] located at 172.17.60.201:7001
  "haha"
  通过验证，集群运行正常


  ```

  ```
