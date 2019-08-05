# HADOOP

# 01

* 빅데이터의 3요소 : 크기(Volume), 속도(Velocity), 다양성(Variety)



하둡 : 대용량 데이터를 분산 처리할 수 있는 자바 기반의 오픈소스 프레임워크

(BI, OLAP)

* NoSQL

  :  RDBMS처럼 완벽한 데이터 무결성과 정합성을 제공하지 않음

  핵심 데이터에는 RDBMS를 이용하고, 핵심이 아니지만 데이터를 보관하고 처리해야하는 경우에 이용

  MongoDB가 가장 많이 사용됨

* 과제

  고가용성 지원(High Availability), 99.999%상태의 가용


---------------



### Hadoop 설치



![Hadoop_01-1](https://user-images.githubusercontent.com/50862497/62451798-e4f30a80-b7a9-11e9-90f1-799ee2d0f096.png)



```
1. 필요 파일 다운 로드
1) java
2) hadoop
3) eclipse
4) tomcat
5) mysql
	MariaDB 설치
    [maria]# pwd
    /root/file/maria
    # yum -y remove mariadb-libs
    # yum -y localinstall Maria*
    Installed:
      MariaDB-client.x86_64 0:10.0.15-1.el7.centos
      MariaDB-common.x86_64 0:10.0.15-1.el7.centos
      MariaDB-server.x86_64 0:10.0.15-1.el7.centos
    Complete!
    # systemctl restart mysql
    # systemctl status mysql
    # chkconfig mysql on => 서비스 상시 가동
    # firewall-config
    public - 서비스 - mysql 체크(런타임, 영구적)
    # mysql
    # mysqladmin -u root password '111111'
    # mysql -u root -p 

# systemctl disable firewalld
firewalld를 영구적으로 사용하지 않겠다

https://archive.apache.org/dist/hadoop/common/hadoop-1.2.1/ -> tar xvf hadoop-1.2.1.tar.gz

2. etc/hosts
70.12.114.211 hadoopserver1

3. etc/hostname
hostname : hadoopserver1
hostnamectl set-hostname hadoopserver1


4. etc/sysconfig/network-scripts
TYPE="Ethernet"
BOOTPROTO=none
IPADDR=70.12.114.211
NETMASK=255.255.255.0
GATEWAY=70.12.116.2
DNS1=70.12.116.2

5. etc/profile
JAVA_HOME=/usr/java/jdk1.7.0_80
HADOOP_HOME=/usr/local/hadoop
export PATH=HADOOP_HOME/bin:JAVA_HOME/bin:$PATH
export PATH
CLASSPATH=$CLASSPATH:JAVA_HOME/lib
export CLASSPATH
or
JAVA_HOME=/etc/jdk1.8
HADOOP_HOME=/etc/hadoop-1.2.1
export JAVA_HOME CLASSPATH TOMCAT_HOME HADOOP_HOME
PATH=.:$JAVA_HOME/bin:$TOMCAT_HOME/bin:$HADOOP_HOME/bin:$PATH
export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL

6. ssh
서버와 서버끼리 왕래할때 PWD 없애기위해 private public key를 만들어 배포
pri
[~]#  ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
Generating public/private dsa key pair.
Your identification has been saved in /root/.ssh/id_dsa.
Your public key has been saved in /root/.ssh/id_dsa.pub.
The key fingerprint is:
5a:6a:3a:bb:8c:6b:a0:6b:c3:4e:44:ed:ea:47:86:96 root@hadoopserver1
The key's randomart image is:
+--[ DSA 1024]----+
|                 |
|  .              |
| . .             |
|. .              |
| .o.    S        |
|oE.o   +         |
|++o   +          |
|++.+.o           |
|+==.=+           |
+-----------------+

dsa알고리즘을 이용해서 public/private 키 만들기
[.ssh]# ls
id_sda  id_sda.pub  known_hosts
private public key
[.ssh]# cat id_dsa.pub >> authorized_keys
[~]# ssh hadoopserver1 
Last login: Mon Aug  5 14:04:18 2019 from hadoopserver1


/etc/hadoop-1.2.1/conf

[conf]# vi core-site.xml 
<configuration>
<property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:9000</value>
</property>
<property>
  <name>hadoop.tmp.dir</name>
  <value>/etc/hadoop-1.2.1/tmp</value>
</property>
</configuration>

[conf]# vi hdfs-site.xml 
<configuration>
<property>
  <name>dfs.replication</name>
  <value>1</value>
</property>
<property>
  <name>dfs.name.dir</name>
  <value>/etc/hadoop-1.2.1/name</value>
</property>
<property>
  <name>dfs.data.dir</name>
  <value>/etc/hadoop-1.2.1/data</value>
</property>
<property>
  <name>dfs.webhdfs.enabled</name>
  <value>true</value>
</property>
</configuration>

[conf]# vi mapred-site.xml
<configuration>
<property>
  <name>mapred.job.tracker</name>
  <value>localhost:9001</value>
</property>
</configuration>

[conf]# vi hadoop-env.sh 
9  export JAVA_HOME=/etc/jdk1.8
10 export HADOOP_HOME_WARN_SUPPRESS="TRUE"

[conf]# . ./hadoop-env.sh 
# vi /etc/bashrc
. /etc/. /etc/hadoop-1.2.1/conf/hadoop-env.sh
. /etc/hadoop-1.2.1/conf/hadoop-env.sh


포맷하기
reboot(HADOOP_PATH가 안잡혀서)
[~]# hadoop namenode -format
-> [hadoop-1.2.1] name 생긴다.
[~]# start-all.sh
starting namenode, logging to /etc/hadoop-1.2.1/libexec/../logs/hadoop-root-namenode-hadoopserver1.out
localhost: starting datanode, logging to /etc/hadoop-1.2.1/libexec/../logs/hadoop-root-datanode-hadoopserver1.out
localhost: starting secondarynamenode, logging to /etc/hadoop-1.2.1/libexec/../logs/hadoop-root-secondarynamenode-hadoopserver1.out
starting jobtracker, logging to /etc/hadoop-1.2.1/libexec/../logs/hadoop-root-jobtracker-hadoopserver1.out
localhost: starting tasktracker, logging to /etc/hadoop-1.2.1/libexec/../logs/hadoop-root-tasktracker-hadoopserver1.out

[~]# jps
2656 TaskTracker
2192 NameNode
2437 SecondaryNameNode
2839 Jps
2314 DataNode
2526 JobTracker

http://70.12.114.211:50070/dfshealth.jsp

[hadoop-1.2.1]# hadoop dfs -mkdir /test
# hadoop dfs -put README.txt /data/input1
데이터 넣기

# hadoop jar hadoop-examples-1.2.1.jar wordcount /data/input1 /data/output1
wordcount를 실행하여 input1에있는파일을 분석해서 ouput1으로 뿌려라

```



```
hadoopserver2

[~]# ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
Generating public/private dsa key pair.
Created directory '/root/.ssh'.
Your identification has been saved in /root/.ssh/id_dsa.
Your public key has been saved in /root/.ssh/id_dsa.pub.
The key fingerprint is:
7f:26:3e:ea:e6:59:67:a4:c0:5f:ac:a7:54:f9:f4:e3 root@hadoopserver2
The key's randomart image is:
+--[ DSA 1024]----+
|                 |
|                 |
|                 |
|       .   . .   |
|        S   * .  |
|         + * o . |
|          O * ...|
|        .=.O  . .|
|       +=.o.   E |
+-----------------+



/etc/hadoop-1.2.1/conf

[conf]# vi core-site.xml 
<configuration>
<property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:9000</value>
</property>
<property>
  <name>hadoop.tmp.dir</name>
  <value>/etc/hadoop-1.2.1/tmp</value>
</property>
</configuration>

```



##### 오라클 환경설정

(시작할때마다 실행)

vi /etc/bashrc