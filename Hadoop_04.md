# 하둡 에코시스템

## 17 하이브

(이전에 저장된 것을 분석하기위해 맵리듀스를 사용했는데 더 쉬운걸 쓰고 싶다.)

SQL문으로 요청을하면 하이브에서는 자동적으로 SQL을 맵리듀스 프로그램으로 변환해준다. (맵리듀스 작성 안해도 됨)



하이브 필수요건 : mysql

<<http://archive.apache.org/dist/hive/hive-1.0.1/>

apache-hive-1.0.1-bin.tar.gz



```
# mysql -u root -p
MariaDB[mysql]>  select user,host from user;
-- hive 만들기 --
MariaDB [mysql]> grant all privileges on *.* to 'hive'@'localhost' identified by '111111';
MariaDB [mysql]> flush privileges;
MariaDB [mysql]> select user,host from user;
+------+---------------+
| user | host          |
+------+---------------+
| root | 127.0.0.1     |
| root | ::1           |
|      | hadoopserver2 |
| root | hadoopserver2 |
|      | localhost     |
| hive | localhost     |
| root | localhost     |
+------+---------------+

-- hive가 사용할 hive_db 만들기 --
MariaDB [mysql]> create database hive_db;
MariaDB [mysql]> grant all privileges on hive_db.* to 'hive'@'%' identified by '111111' with grant option;
MariaDB [mysql]> grant all privileges on hive_db.* to 'hive'@'localhost' identified by '111111' with grant option;
MariaDB [mysql]> flush privileges;
MariaDB [mysql]> commit;

[root@hadoopserver2 file]# mysql -u hive -p
MariaDB [(none)]> use hive_db

[root@hadoopserver2 file]# mv apache-hive-1.0.1-bin hive
[root@hadoopserver2 file]# cp -r hive /etc
# vi /etc/profile
HIVE_HOME=/etc/hive
export JAVA_HOME CLASSPATH TOMCAT_HOME HADOOP_HOME HIVE_HOME
PATH=.:$JAVA_HOME/bin:$TOMCAT_HOME/bin:$HADOOP_HOME/bin:$HIVE_HOME/bin:$PATH

/etc/hive/conf
[conf]# vi hive-site.xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>hive.metastore.local</name>
        => hive가 사용하는 메타정보를 mysql에 저장하겠다
        <value>true</value>
        <description>controls whether to connect to remove metastore server or open a new metastore server in Hive Client JVM</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mariadb://localhost:3306/hive_db?createDatabaseIfNotExist=true</value>
        => 3306으로 들어가서 hive_db 데이터를 사용
        <description>JDBC connect string for a JDBC metastore</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>org.mariadb.jdbc.Driver</value>
        => maria db의 jdbc를 사용하겠다(hive가 mariadb를 사용하기 때문에 mariadb jdbc가 필요)
        <description>Driver class name for a JDBC metastore</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>hive</value>
        <description>username to use against metastore database</description>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>111111</value>
        <description>password to use against metastore database</description>
    </property>
</configuration>


[root@hadoopserver2 file]# cp mariadb-java-client-1.3.5.jar /etc/hive/lib
=> hive가 mariadb에 들어갈 수 있게 (jdbc가 필요함)
	hive가 mysql을 사용하기 위해서

# hadoop dfs -mkdir /tmp => hive가 사용할 폴더
# hadoop dfs -mkdir /user/hive/warehouse
# hadoop dfs -mkdir /tmp/hive
# hadoop dfs -chmod g+w /tmp
# hadoop dfs -chmod g+w /user/hive/warehouse
# hadoop dfs -chmod 777 /tmp/hive

MariaDB [mysql]> grant all privileges on hive_db.* to 'hive'@'localhost' identified by '111111' with grant option;
MariaDB [mysql]> grant all privileges on hive_db.* to 'hive'@'%' identified by '111111' with grant option;
MariaDB [mysql]> commit;
MariaDB [mysql]> select user,host from user;
+------+---------------+
| user | host          |
+------+---------------+
| hive | %             |
| root | 127.0.0.1     |
| root | ::1           |
|      | hadoopserver2 |
| root | hadoopserver2 |
|      | localhost     |
| hive | localhost     |
| root | localhost     |
+------+---------------+


# hadoop dfs -chmod 777 /tmp/hive
# hive
hive> CREATE TABLE HDI(id INT, country STRING, hdi FLOAT, lifeex INT, mysch INT, 
    > 
    > eysch INT, gni INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS 
    > 
    > TEXTFILE;

hive> show tables;
OK
hdi => 새로 생성 됨
Time taken: 0.029 seconds, Fetched: 1 row(s)

데이터 로드
hive> load data local inpath '/root/hdi.txt' into table HDI;
Loading data to table default.hdi
Table default.hdi stats: [numFiles=1, totalSize=9152]
OK

확인
hive> select id, country from hdi;

```



실제 데이터는 hadoop에 저장, hadoop데이터의 구조를 mysql에 저장



--------------------



```
mysql -u hive -p hive_db -> hive_db로 들어가겠다
[hive_db] show tables;
select * from TBLS;

[data] wget http://stat-computing.org/dataexpo/2009/2006.csv.bz2
wget http://stat-computing.org/dataexpo/2009/2007.csv.bz2
wget http://stat-computing.org/dataexpo/2009/2008.csv.bz2


```

