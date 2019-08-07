# HADOOP

## 02 하둡 개발 준비

빅데이터 사용환경을위해 하둡(sw)을 사용

* 네임노드 : 네임노드에 요청을하면 데이터를 나누어서 데이터노드에 넣거나(복제), 넣은 정보를 알고있어서 정보를 취합해서 꺼냄, 보조네임노드에 백업하기

  네비게이션 역할

* 데이터노드 : 데이터 저장, 모니터링

* 보조(Secondary) 네임노드 : 네임노드가 죽었을 때, 새로운 네임노드에 백업된 데이터를 주는 것(recovery할 때)

![hadoop02](https://user-images.githubusercontent.com/50862497/62586526-03b3e700-b8f9-11e9-848c-8845bba4a12d.png)



##### 순서

```
1. 하둡 다운로드
2. /etc/밑에 설정
3. 자바설치
4. SSH 설정(다른서버에 접속할 때 pwd 안물어보게하려고)
	내가 생서한 public key를 다른 서버에 복사해서 놓아두면 다른 서버에 자유롭게 접속 가능
5. 하둡 파일 풀어서 셋팅
6. 하둡 환경설정 파일 수정
	hadoop-env.sh -> JKD, JAVA_HOME(profile에 있어도 여기를 한번 확인함), WARN
	masters, slaves -> 한 서버에 다 설치할 때는 안해도됨
		보조네임노드, 데이터노드 실행할 서버 써주기
	core-site.xml -> name의 port
	hdfs-site.xml -> replication 복제 수, 어떤 머신에 어떤 데이터노드에 어떠한 파일들이 저장되어있는지 저장되어 있는 곳, dfs.data.dif data가 저장되는 공간, web으로 접근할 수 있게
	mapred-site.xml -> jobtracker : 분석 요청을 받고 결과값을 return
	/etc/profile -> HADOOP_HOME 설정
7. 네임노드에서 모든 수정작업이 완료되면 tar로 묶어서 모든 데이터노드 서버로 네임노드의 하둡환경설정파일 전송하여 설치
8. 네임노드를 초기화하고 모든 데몬 실행하면 하둡 실행
9. 하둡 실행
	포맷(HADOOP1에서 포맷하면 HADOOP2,3,4가 포맷됨)
	HADOOP1에서 start-all.sh 실행하면 각각의 프로세스 동작(HADOOP2,3,4)
	
```



7번 -> 4대의 서버에 옮기기

HADOOP1(VMware virtual machine Configuration) -> display:HADOOP1

OPEN HADOOP1 

Network - NAT , MAC : 00:50:56:28:68:58(192.168.111.201)

```
[~]# hostnamectl set-hostname hadoop1
[~]# vi /etc/sysconfig/network-scripts/ifcfg-ens33 
HWADDR="00:50:56:28:68:58"
TYPE="Ethernet"
BOOTPROTO=none
IPADDR=192.168.111.201
NETMASK=255.255.255.0
GATEWAY=192.168.111.2
DNS1=192.168.111.2
[~]# systemctl restart network
[~]# vi /etc/hosts
192.168.111.201 hadoop1
192.168.111.202 hadoop2
192.168.111.203 hadoop3
192.168.111.204 hadoop4
[~]# poweroff

```



C:\CENTOS에서 HADOOP1을 HADOOP2,3,4복사

00:50:56:28:68:58 hadoop1 192.168.111.201

00:50:56:26:71:FC hadoop2 192.168.111.202

00:50:56:20:F2:FA hadoop3 192.168.111.203

00:50:56:24:8B:DC hadoop4 192.168.111.204





```

[root@hadoop1 ~]# ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
Generating public/private dsa key pair.
Created directory '/root/.ssh'.
Your identification has been saved in /root/.ssh/id_dsa.
Your public key has been saved in /root/.ssh/id_dsa.pub.
The key fingerprint is:
f5:2e:41:0b:aa:01:7d:fa:9b:8f:61:5a:10:af:13:2c root@hadoop1
The key's randomart image is:
+--[ DSA 1024]----+
|                 |
|   .             |
|  . o . . o      |
|   o = . + o     |
|  E * o S o .    |
|   . B     o     |
|    + =   . .    |
|     = =   .     |
|    . +..        |
+-----------------+

[root@hadoop2 ~]# ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
Generating public/private dsa key pair.
Created directory '/root/.ssh'.
Your identification has been saved in /root/.ssh/id_dsa.
Your public key has been saved in /root/.ssh/id_dsa.pub.
The key fingerprint is:
7d:f1:bc:cb:55:72:83:7b:19:9f:69:ad:4c:76:64:4d root@hadoop2
The key's randomart image is:
+--[ DSA 1024]----+
|                 |
|                 |
|            .   E|
|         .   +...|
|        S . ..+o*|
|           .  .*X|
|             .+*=|
|             =o= |
|              =  |
+-----------------+

[root@hadoop3 ~]# ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
Generating public/private dsa key pair.
Created directory '/root/.ssh'.
Your identification has been saved in /root/.ssh/id_dsa.
Your public key has been saved in /root/.ssh/id_dsa.pub.
The key fingerprint is:
8f:d1:4a:2f:f7:f2:73:9f:23:03:84:d4:bf:ba:af:a6 root@hadoop3
The key's randomart image is:
+--[ DSA 1024]----+
|          .      |
|         . .     |
|        . . .    |
|         o . .   |
|        S o   .  |
|       . * . .   |
|        + + o    |
|         o.+.o...|
|         Eo=*+ooo|
+-----------------+
[root@hadoop4 ~]# ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
Generating public/private dsa key pair.
Created directory '/root/.ssh'.
Your identification has been saved in /root/.ssh/id_dsa.
Your public key has been saved in /root/.ssh/id_dsa.pub.
The key fingerprint is:
46:b0:a4:f5:45:b6:05:2f:a5:15:d9:5d:33:15:fc:4e root@hadoop4
The key's randomart image is:
+--[ DSA 1024]----+
|      +  .=.=+.=*|
|     + + o B. ..+|
|    . . o + .   .|
|       .   .    E|
|        S      o |
|       .        .|
|                 |
|                 |
|                 |
+-----------------+

[.ssh]# cat id_dsa.pub > authorized_keys

hadoop1에서 2,3,4에 자유롭게 들어가기 위한 작업
[root@hadoop1 .ssh]# scp authorized_keys root@hadoop2:~/.ssh/authorized_keys
[root@hadoop1 .ssh]# scp authorized_keys root@hadoop3:~/.ssh/authorized_keys
[root@hadoop1 .ssh]# scp authorized_keys root@hadoop4:~/.ssh/authorized_keys

-- 확인하기 --
[root@hadoop1 .ssh]# ssh root@hadoop3 ls ~/.ssh
authorized_keys
id_dsa
id_dsa.pub
[root@hadoop1 .ssh]# ssh hadoop2
Last login: Tue Aug  6 11:03:03 2019
[root@hadoop2 ~]# exit
logout
Connection to hadoop2 closed.

-- hadoop1 --
[file]# tar xvf hadoop-1.2.1.tar.gz
[file]# cp -r hadoop-1.2.1 /etc

# vi /etc/profile
HADOOP_HOME=/etc/hadoop-1.2.1
export JAVA_HOME CLASSPATH TOMCAT_HOME HADOOP_HOME
PATH=.:$JAVA_HOME/bin:$TOMCAT_HOME/bin:$HADOOP_HOME/bin:$PATH

/etc/hadoop-1.2.1/conf
[conf]# vi hadoop-env.sh 
9  export JAVA_HOME=/etc/jdk1.8
10 export HADOOP_HOME_WARN_SUPPRESS="TRUE"

[root@hadoop1 conf]# vi masters
hadoop2 (-> secondary namenode)
[root@hadoop1 conf]# vi slaves
hadoop2
hadoop3
hadoop4

[root@hadoop1 conf]# vi core-site.xml 
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://192.168.111.201:9000</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/etc/hadoop-1.2.1/tmp</value>
  </property>
</configuration>


[root@hadoop1 conf]# vi hdfs-site.xml 
<configuration>
<property>
  <name>dfs.replication</name>
  <value>2</value>
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


[root@hadoop1 conf]# vi mapred-site.xml 
<configuration>
  <property>
   <name>mapred.job.tracker</name>
   <value>192.168.111.201:9001</value>
  </property> 
</configuration>

[root@hadoop1 ~]# vi /etc/bashrc
. /etc/hadoop-1.2.1/conf/hadoop-env.sh

[root@hadoop1 etc]# scp /etc/bashrc root@hadoop1:/etc
[root@hadoop1 etc]# scp /etc/bashrc root@hadoop2:/etc
[root@hadoop1 etc]# scp /etc/bashrc root@hadoop3:/etc
[root@hadoop1 etc]# scp /etc/bashrc root@hadoop4:/etc

# systemctl stop firewalld
# systemctl disable firewalld

```

SETTING 끝

```
[root@hadoop1 etc]# tar cvfz hadoop.tar.gz hadoop-1.2.1/

root권한으로 hadoop2의 /etc밑으로 보내기
[root@hadoop1 etc]# scp hadoop.tar.gz root@hadoop2:/etc
[root@hadoop1 etc]# scp hadoop.tar.gz root@hadoop3:/etc
[root@hadoop1 etc]# scp hadoop.tar.gz root@hadoop4:/etc

[root@hadoop1 etc]# scp /etc/profile root@hadoop2:/etc
[root@hadoop1 etc]# scp /etc/profile root@hadoop3:/etc
[root@hadoop1 etc]# scp /etc/profile root@hadoop4:/etc

[root@hadoop1 etc]# ssh root@hadoop2 "cd /etc;tar xvfz hadoop.tar.gz;rm -rf hadoop.tar.gz"
[root@hadoop1 etc]# ssh root@hadoop3 "cd /etc;tar xvfz hadoop.tar.gz;rm -rf hadoop.tar.gz"
[root@hadoop1 etc]# ssh root@hadoop4 "cd /etc;tar xvfz hadoop.tar.gz;rm -rf hadoop.tar.gz"

```

(reboot)

```
[root@hadoop1 ~]# hadoop namenode -format
[root@hadoop1 ~]# start-all.sh  -> 모든
[root@hadoop1 etc]# jps
3207 Jps
2876 NameNode
3069 JobTracker
[root@hadoop2 etc]# jps
2851 Jps
2580 DataNode
2742 TaskTracker
2639 SecondaryNameNode
[root@hadoop3 etc]# jps
2757 Jps
2570 DataNode
2651 TaskTracker
[root@hadoop4 etc]# jps
2773 Jps
2584 DataNode
2665 TaskTracker

[root@hadoop1 hadoop-1.2.1]# ls
name
[root@hadoop2 hadoop-1.2.1]# ls
data tmp
[root@hadoop3 hadoop-1.2.1]# ls
data tmp
[root@hadoop4 hadoop-1.2.1]# ls
data tmp

```



-----------

### .

1. SSH 활성화

   ssh-keygen

2. 방화벽 해제

   systemctl stop firewalld

   systemctl disable firewalld

3. namenode 컴퓨터에서 hadoop 설치 및 configuration 

   /etc/profile

   /etc/bashrc

4. hadoop.tar.gz 각 서버에 복사

5. namenode format

   hadoop namenode -format

6. start-all.sh



- 잘못될 경우, /data, /name, /tmp 삭제 후 다시 format



7. namenode 컴퓨터에서 README.txt.파일을 hadoop에 저장 후 wordcount 프로그램 실행

   ```
   [root@hadoop1 ~]# hadoop dfs -mkdir /test
   [root@hadoop1 hadoop-1.2.1]# hadoop dfs -put README.txt /test

   [root@hadoop1 hadoop-1.2.1]# hadoop dfs -mkdir /data
   [root@hadoop1 hadoop-1.2.1]# hadoop dfs -mkdir /data/input
   [root@hadoop1 hadoop-1.2.1]# hadoop dfs -put README.txt /data/input/README.txt
   [root@hadoop1 hadoop-1.2.1]# hadoop jar hadoop-examples-1.2.1.jar wordcount /data/input1 /data/output1

   http://192.168.111.201:50070
   Browse the filesystem 확인

   [root@hadoop1 hadoop-1.2.1]# hadoop dfs -get /data/inpu1/README.txt RD.txt

   JOBTRACKER가 HADOOP2,3,4의 TASKTRACKER에게 요청
   데이터 분석 취합 파일을 다시 JOBTRACKER에게 보냄

   ```

   ​

