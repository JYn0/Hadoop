# 하둡 에코시스템

## 17 하이브

(이전에 저장된 것을 분석하기위해 맵리듀스를 사용했는데 더 쉬운걸 쓰고 싶다.)

SQL문으로 요청을하면 하이브에서는 자동적으로 SQL을 맵리듀스 프로그램으로 변환해준다. (맵리듀스 작성 안해도 됨)

![hadoop0401](https://user-images.githubusercontent.com/50862497/62686031-d6e9f780-b9fe-11e9-9f72-aa39e61d8b02.png)



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
safe mode 해제
[root@hadoopserver2 conf]# hadoop dfsadmin -safemode leave
Safe mode is OFF

mysql -u hive -p hive_db -> hive_db로 들어가겠다
[hive_db] show tables;
select * from TBLS;

[data] wget http://stat-computing.org/dataexpo/2009/2006.csv.bz2
wget http://stat-computing.org/dataexpo/2009/2007.csv.bz2
wget http://stat-computing.org/dataexpo/2009/2008.csv.bz2

[root@hadoopserver2 data]# bzip2 -kd 2006.csv.bz2 
[root@hadoopserver2 data]# bzip2 -kd 2007.csv.bz2 
[root@hadoopserver2 data]# bzip2 -kd 2008.csv.bz2 



hive> CREATE TABLE airline_delay(
    > Year INT,
    > MONTH INT,
    > DayofMonth INT,
    > DayofWeek INT,
    > DepTime INT,
    > CRSDepTime INT,
    > ArrTime INT,
    > CRSArrTime INT,
    > UniqueCarrier STRING,
    > FlightNum INT,
    > TailNum STRING,
    > ActualElapsedTime INT,
    > CRSElapsedTime INT,
    > AirTime INT,
    > ArrDelay INT,
    > DepDelay INT,
    > Origin STRING,
    > Dest STRING,
    > Distance INT,
    > TaxiIn INT,
    > TaxiOut INT,
    > Cancelled INT,
    > CancellationCode STRING
    > COMMENT 'A = carrier, B = weather, C = NAS, D = security',
    > Diverted INT COMMENT '1 = yes, 0 = no',
    > CarrierDelay STRING,
    > WeatherDelay STRING,
    > NASDelay STRING,
    > SecurityDelay STRING,
    > LateAircraftDelay STRING)
    > COMMENT 'TEST DATA'
    > PARTITIONED BY (delayYear INT)
    > ROW FORMAT DELIMITED
    >     FIELDS TERMINATED BY ','
    >     LINES TERMINATED BY '\n'
    >     STORED AS TEXTFILE;


hive> LOAD DATA LOCAL INPATH '/root/data/2006.csv' OVERWRITE INTO TABLE airline_delay PARTITION (delayYear='2006');
LOAD DATA LOCAL INPATH '/root/data/2007.csv' OVERWRITE INTO TABLE airline_delay PARTITION (delayYear='2007');
LOAD DATA LOCAL INPATH '/root/data/2008.csv' OVERWRITE INTO TABLE airline_delay PARTITION (delayYear='2008');

hive> select * from airline_delay where delayYear='2008' LIMIT 10;
OK
airline_delay.year	airline_delay.month	airline_delay.dayofmonth	airline_delay.dayofweek	airline_delay.deptime	airline_delay.crsdeptime	airline_delay.arrtime	airline_delay.crsarrtime	airline_delay.uniquecarrier	airline_delay.flightnum	airline_delay.tailnum	airline_delay.actualelapsedtime	airline_delay.crselapsedtime	airline_delay.airtime	airline_delay.arrdelay	airline_delay.depdelay	airline_delay.origin	airline_delay.dest	airline_delay.distance	airline_delay.taxiin	airline_delay.taxiout	airline_delay.cancelled	airline_delay.cancellationcode	airline_delay.diverted	airline_delay.carrierdelay	airline_delay.weatherdelay	airline_delay.nasdelay	airline_delay.securitydelay	airline_delay.lateaircraftdelay	airline_delay.delayyear

hive> select count(*) from airline_delay where delayYear='2008';

2018년 월별 ArrDelay와 DepDelay 평균 구하기
hieve> select Month, avg(ArrDelay), avg(DepDelay) from airline_delay where delayYear='2007' group by Month order by Month;
Total MapReduce CPU Time Spent: 24 seconds 190 msec
OK
month	_c1	_c2
1	9.162151701506165	10.286743415948312
2	13.51979483296776	14.02253123732965
3	10.084908470559062	11.836802542694253
4	8.516229825822615	10.07771370814071
5	7.037888681043307	8.329205401044868
6	16.17952800579826	16.214701071994014
7	14.107679837700504	14.802613636005757
8	12.57153344196042	13.515875591278409
9	3.7494980749698845	6.157685958765665
10	6.5082592714725775	7.973724233737243
11	4.793344024722863	7.446888306310218
12	16.213714049846818	16.201398473962534
Time taken: 42.457 seconds, Fetched: 12 row(s)

```



### Java Hive 연동

<https://www.slf4j.org/download.html>

slf4j-1.7.27.zip

![hadoop0402](https://user-images.githubusercontent.com/50862497/62686055-e9643100-b9fe-11e9-8853-4841245c6ec1.png)



```
eclipse(java app)에서 hive로 들어가기

[root@hadoopserver2 conf]# hive --service hiveserver2
(자바가 요청하는 것을 기다림)

eclipse
new java project
add external jars - hivelibs
package hive;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class Hive {

	public static void main(String[] args) throws Exception {
		Class.forName("org.apache.hive.jdbc.HiveDriver");
		Connection conn = DriverManager.getConnection("jdbc:hive2://70.12.114.210:10000/default", "", "");
		Statement stmt = conn.createStatement();
		ResultSet rs = stmt.executeQuery("SELECT * FROM hdi");
		while (rs.next()) {
			System.out.println(rs.getString(2));
		}
		conn.close();
		System.out.println("Success....");
	}
}

```

```java
package hive;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class Hive {

	public static void main(String[] args) throws Exception {
		Class.forName("org.apache.hive.jdbc.HiveDriver");
		Connection conn = DriverManager.getConnection("jdbc:hive2://70.12.114.210:10000/default", "root", "111111");
		Statement stmt = conn.createStatement();
		ResultSet rs = stmt.executeQuery("select Month, avg(ArrDelay), avg(DepDelay) from airline_delay where delayYear='2008' group by Month order by Month");
		while (rs.next()) {
			for(int i=1; i<4; i++) {
				System.out.print(rs.getString(i)+"\t|");
			}
			System.out.println();
		}
		conn.close();
		System.out.println("Success....");
	}
}

console

1	|10.188855960349496	|11.47609595943289	|
2	|13.077836997760205	|13.706226305045202	|
3	|11.19236458018227	|12.49126948010275	|
4	|6.807297481094145	|8.201132754082797	|
5	|5.978448290248828	|7.642741440912969	|
6	|13.266756009659792	|13.609818079614008	|
7	|9.975049681276131	|11.807544712497146	|
8	|6.91091468997087	|9.61475257451315	|
9	|0.6977328787273043	|3.961818849518357	|
10	|0.4154954706912698	|3.803487686795168	|
11	|2.015857969430839	|5.420469498039744	|
12	|16.680505081496417	|17.30437978049954	|
Success....

```



