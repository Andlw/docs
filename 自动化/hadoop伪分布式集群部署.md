一、设置好JAVA_HOME变量，而后解压安装包到指定目录
    ]# mkdir -pv /hdapps
    ]# tar xvf hadoop-2.6.2.tar.gz -C /hdapps/
    ]# ln -sv /hdapps/hadoop-2.6.2 /hdapps/hadoop

    编辑环境配置文件/etc/profile.d/hadoop.sh
    export HADOOP_PREFIX="/hdapps/hadoop"
    export PATH=$HADOOP_PREFIX/bin:$HADOOP_PREFIX/sbin:$PATH

二、创建运行hadoop进程的用户和相关目录,创建用户和组，出于安全的目的，通常需要用特定的用
    户来运行hadoop不同的守护进程，例如，以hadoop为组，分别用三个用户yarn、hdfs和mapred来运行相应的进程
    ]# groupadd hadoop
    ]# useradd -g hadoop yarn
    ]# useradd -g hadoop hdfs
    ]# useradd -g hadoop mapred

    创建数据和日志目录，hadoop需要不同权限的数据和日志目录，这里以/data/hadoop/hdfs为hdfs数据存储目录
    ]# mkdir -pv /data/hadoop/hdfs/{nn,dn,snn}
    ]# chown -R hdfs:hadoop /data/hadoop/hdfs
    ]# cd /hdapps/hadoop
    ]# mkdir logs
    ]# chmod g+w logs
    ]# chown -R yarn:hadoop ./*

三、配置hadoop
    ]# cd /hdapps/hadoop/etc/hadoop
    ]# vim core-site.xml
    <configuration>
    <property>
      <name>fs.defaultFS</name>
      <value>hdfs://hadoopdev:8020</value>
    </property>
    <property>
     <name>fs.trash.interval</name>
     <value>360</value>
    </property>
    <property>
      <name>hadoop.http.authentication.simple.anonymous.allowed</name>
      <value>true</value>
    </property>
    <property>
      <name>io.file.buffer.size</name>
      <value>131072</value>
    </property>
    <property>
      <name>hadoop.tmp.dir</name>
      <value>/data/hadoop</value>
    </property>
    </configuration>

    hdfs-site.xml主要用于配置HDFS相关属性，例如复制因子(即数据块的副本数)，NN和DN用于存储数据的目录等，数据块的副本数对于伪分布式的hadoop应用为1，而NN和DN用于存储的数据的目录为前面的步骤中专门为其创建的路径。另外，前面的步骤中也为SNN创建了相关的目录，这里也一并配置为启用状态
    ]# vim hdfs-site.xml
      <configuration>
        <property>
          <name>dfs.replication</name>
          <value>1</value>
        </property>
        <property>
          <name>dfs.namenode.name.dir</name>
          <value>file:///data/hadoop/hdfs/nn</value>
        </property>
        <property>
          <name>dfs.datanode.data.dir</name>
          <value>file:///data/hadoop/hdfs/dn</value>
        </property>
        <property>
          <name>fs.checkpoint.dir</name>
          <value>file:///data/hadoop/hdfs/snn</value>
        </property>
        <property>
          <name>fs.checkpoint.edits.dir</name>
          <value>file:///data/hadoop/hdfs/snn</value>
        </property>
        <property>
          <name>dfs.permissions</name>
          <value>false</value>
        </property>
      </configuration>

      mapred-site.xml文件用于配置集群的MapReduce framework，此处应该指定使用yarn，另外的可用值还有local和classic。mapred-site.xml默认不存在，但有模块文件mapred-site.xml.template,只需要将其复制为mapred-site.xml就可以
      ]# vim mapred-site.xml
      <configuration>
        <property>
        <name>mapreduce.map.log.level</name>
        <value>INFO</value>
        <description>The logging level for the map task. The allowed levels are:
        OFF, FATAL, ERROR, WARN, INFO, DEBUG, TRACE and ALL.
        The setting here could be overridden if "mapreduce.job.log4j-properties-file"
        is set.
        </description>
        </property>
      </configuration>

      yarn-site.xml用于适配YARN进程以及YARN的相关属性。首先需要指定ResourceManager守护进程的主机和监听的端口，对于伪分布式模型来讲，其主机为localhost，默认的端口为8032，其次需要指定ResourceManager使用的scheduler，以及NodeManager的辅助服务
      ]# vim yarn-site.xml
        <configuration>
          <property>
            <name>yarn.resourcemanager.address</name>
            <value>hadoopdev:8032</value>
          </property>
          <property>
            <name>yarn.resourcemanager.scheduler.address</name>
            <value>hadoopdev:8030</value>
          </property>
          <property>
            <name>yarn.resourcemanager.resource-tracker.address</name>
            <value>hadoopdev:8031</value>
          </property>
          <property>
            <name>yarn.resourcemanager.admin.address</name>
            <value>hadoopdev:8033</value>
          </property>
          <property>
            <name>yarn.resourcemanager.webapp.address</name>
            <value>hadoopdev:8088</value>
          </property>
          <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
          </property>
          <property>
            <name>yarn.nodemanager.auxservices.mapreduce_shuffle.class</name>
            <value>org.apache.hadoop.mapred.ShuffleHandler</value>
          </property>
          <property>
            <name>yarn.resourcemanager.scheduler.class</name>
            <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.Scheduler</value>
          </property>
        </configuration>

        Hadoop的各守护进程依赖于JAVA_HOME环境变量，需要在hadoop-env.sh中设置JAVA_HOME环境变量
        ]# vim hadoop-env.sh
          export JAVA_HOME=/usr/java/latest

四、格式化HDFS
    在HDFS的NN启动之前需要先初始化其用于存储数据的目录，如果hdfs-site.xml中dfs.namenode.name.dir属性指定的目录不存在，格式化命令会自动创建，如果事先存在，请确保其权限设置正确，此时格式操作会清楚其内容的所有数据并重新建立一个新的文件系统
    ]# su - hdfs
    ]$  hdfs namenode -format

五、启动Hadoop
    启动HDFS服务
    ]$ start-dfs.sh
    启动YARN服务
    ]$ start-yarn.sh

六、运行测试程序
    Hadoop-YARN自带了许多样例程序，它们位于Hadoop安装路径下的share/hadoop/mapreduce目录里，其中的hadoop-mapreduce-examples可用作mapreduce程序测试
    ]# su - hdfs
    ]$ yarn jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.2.jar

    ]$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep input output 'dfs[a-z.]+'
