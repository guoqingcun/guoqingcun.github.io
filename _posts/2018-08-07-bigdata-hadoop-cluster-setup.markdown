---
layout: article-detail
title:  "hadoop 3.0.3集群安装"
author: "郭清存"
date:   2018-08-07 20:06:17 +0800
categories: 大数据
location: 
description: "hadoop 3.0.3在虚拟机上集群安装配置"
---
---

### 环境
linux：CentOS 7  
sun java version "1.8.0_144"  
安装ssh，并设置免密  
hadoop-3.0.3.tar.gz  
用户root  
虚拟机 : VMware
- 192.168.1.135 master 
- 192.168.1.120 slave1  
- 192.168.1.182 slave2
  


**为本台机子命名**  

> vim /etc/hostname

	
如图:
<div>
<img src="/images/bigdata/hadoop/cluster-setup/1.jpg" title=""/>
</div>

**配置三台机子的hosts**

> vim /etc/hosts

如图:
<div align="center">
<img src="/images/bigdata/hadoop/cluster-setup/2.jpg" title=""/>
</div>
	
注意：注释掉127.0.0.0,::1 localhost

如上操作后，reboot三台机子

### 配置本台机子ssh

root用户登录后，如下操作
> ssh-keygen -t rsa -P ''  
> 描述:生成无密码的公钥/私钥对,生成的文件在当前用户的.ssh目录下  

如图
<div align="center">
<img src="/images/bigdata/hadoop/cluster-setup/3.jpg" title=""/>
</div>  

  
接下来将公钥添加到authorzied_keys文件里
> cat /root/.ssh/id_rsa.pub > /root/.ssh/authorized_keys
<div align="center">
<img src="/images/bigdata/hadoop/cluster-setup/4.jpg" title=""/>
</div>
把authorzied_keys复制到其它两台slave机子中
> scp authorzied_keys slave1:/root/.ssh/authorzied_keys  
> scp authorzied_keys slave1:/root/.ssh/authorzied_keys  

第一次slave1和slave2使用scp时,会出现如下字样.
> Are you sure you want to continue connecting(yes/no)?  
输入yes,以示确定。回车。以后就可以快速无需密码连接。


### 三台机子上安装sun java version "1.8.0_144"
> 安装过程略.

> 安装目录:/export/java/jdk1.8.0_144

**配置profile:vim /etc/profile**
>  
#oracle jdk start  
export JAVA_HOME=/export/java/jdk1.8.0_144  
export JRE_HOME=${JAVA_HOME}/jre  
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
export PATH=${JAVA_HOME}/bin:$PATH  
#oracle jdk end


如上配置后,执行:
> source /etc/profile 

### 在master安装hadoop

> 解压:tar -zxvf hadoop-3.0.3.tar.gz  
> 解压目录:/export/hadoop3

### 配置hadoop

三台机子配置profile:vim /etc/profile
>
#hadoop start  
export HADOOP_PREFIX=/export/hadoop3  
export PATH=$PATH:$HADOOP_PREFIX/bin  
export PATH=$PATH:$HADOOP_PREFIX/sbin  
#hadoop end


如上配置后，执行:
> source /etc/profile 

接下来配置master上的hadoop文件.需要配置如下几个文件
- hadoop-env.sh 
- core-site.xml 
- hdfs-site.xml 
- mapred-site.xml 
- yarn-site.xml 
- workers 

#### hadoop-env.sh 
> vim /export/hadoop3/etc/hadoop/hadoop-env.sh

配置内容:
>
export JAVA_HOME=/export/java/jdk1.8.0_144  
export HDFS_NAMENODE_USER=root  
export HDFS_DATANODE_USER=root  
export HDFS_SECONDARYNAMENODE_USER=root  
export YARN_RESOURCEMANAGER_USER=root  
export YARN_NODEMANAGER_USER=root  

#### core-site.xml
> vim /export/hadoop3/etc/hadoop/core-site.xml

配置内容:
``` xml
<configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://master:9000</value>
        </property>
        <!-- 指定hadoop运行时产生文件的存储目录 -->
        <property>
            <name>hadoop.tmp.dir</name>
            <value>/export/hadoop3/tmp</value>
        </property>
</configuration>
```

#### hdfs-site.xml
> vim /export/hadoop3/etc/hadoop/hdfs-site.xml

配置内容:
``` xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.name.dir</name>
        <value>/export/hadoop3/hdfs/name</value>
    </property>
    <property>
        <name>dfs.data.dir</name>
        <value>/export/hadoop3/hdfs/data</value>
    </property>
</configuration>
```
#### mapred-site.xml 
> vim /export/hadoop3/etc/hadoop/mapred-site.xml 

配置内容:
``` xml
<configuration>
   <property>
        <name>mapred.job.tracker</name>
        <value>http://master:9001</value>
    </property>

   <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

    <property>
         <name>mapreduce.application.classpath</name>
         <value>
             /export/hadoop3/etc/hadoop,
             /export/hadoop3/share/hadoop/common/*,
             /export/hadoop3/share/hadoop/common/lib/*,
             /export/hadoop3/share/hadoop/hdfs/*,
             /export/hadoop3/share/hadoop/hdfs/lib/*,
             /export/hadoop3/share/hadoop/mapreduce/*,
             /export/hadoop3/share/hadoop/mapreduce/lib/*,
             /export/hadoop3/share/hadoop/yarn/*,
             /export/hadoop3/share/hadoop/yarn/lib/*
         </value>
    </property>

</configuration>
```
#### yarn-site.xml
> vim /export/hadoop3/etc/hadoop/yarn-site.xml

配置内容:
``` xml
<configuration>

<!-- Site specific YARN configuration properties -->

     <property>
          <name>yarn.nodemanager.aux-services</name>
          <value>mapreduce_shuffle</value>
     </property>
     <property>
           <name>yarn.resourcemanager.address</name>
           <value>master:8032</value>
     </property>
     <property>
          <name>yarn.resourcemanager.scheduler.address</name>
          <value>master:8030</value>
      </property>
     <property>
         <name>yarn.resourcemanager.resource-tracker.address</name>
         <value>master:8031</value>
     </property>
     <property>
         <name>yarn.resourcemanager.admin.address</name>
         <value>master:8033</value>
     </property>
     <property>
         <name>yarn.resourcemanager.webapp.address</name>
         <value>master:8088</value>
     </property>

</configuration>
```
#### workers    
> vim /export/hadoop3/etc/hadoop/workers

配置内容:
``` xml
slave1
slave2
```

#### 同步master上的hadoop环境
>
scp /export/hadoop3 slave1:/export/  
scp /export/hadoop3 slave2:/export/	

等上述命令执行完后，hadoop的环境就完安装和配置完成

### 格式化namenode
在启动hadoop前，先要格式化namenode
> hadoop namenode -format

### 启动hadoop
> start-all.sh

执行上述命令完成后，使用jps命令在三台机子上查看启动的进程.
```
master上需要启动如下进程
	- NameNode
	- SecondaryNameNode
	- ResourceManager
slave1上需要启动如下进程
	- NodeManager
	- DataNode
slave2上需要启动如下进程
	- NodeManager
	- DataNode
```
如果进程启动无误，到此hadoop集群安装成功

### hadoop上的hello world

创建两个文件,操作如下
```
mkdir /export/file
echo "hello world hello hadoop" > /export/file/file1.txt
```
创建hadoop的输入目录且把file1.txt文件上传到此
```
hadoop dfs -mkdir /input
hadoop dfs -put /export/file/file1.txt /input
```
环境已经准备OK，那就可以跑第一个应用(单词计数功能)喽.
```
hadoop jar /export/hadoop3/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.0.3.jar wordcount hdfs://master:9000/input/file1.txt   hdfs://master:9000/output/
```
如果执行命令无误，可以通过如下命令查看输出结果
```
hadoop dfs -ls /output/
```
输出结果：
```
-rw-r--r--   2 root supergroup          0 2018-08-07 17:43 /output/_SUCCESS
-rw-r--r--   2 root supergroup         42 2018-08-07 17:43 /output/part-r-00000
```
查看输出内容
```
hadoop dfs -cat /output/part-r-00000  
world   1
Hello   2
hadoop  1
```
>
  <small>本文总阅读量<span id="busuanzi_value_page_pv"></span>次</small>
