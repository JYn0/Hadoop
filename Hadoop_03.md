# HADOOP

## 03 하둡 분산 파일 시스템



C:\Windows\System32\drivers\etc\hosts

70.12.114.210 hadoopserver2



##### 기존의 대용량 시스템

DAS(Direct-Attached Storage) - 서버에 직접 연결된 스토리지(외장형 하드디스크)

NAS(Network-Attached Storage) - 일종의 파일 서버, 별도의 운영체제 사용, 파일 시스템을 안정적으로 공유

SAN(Storage Area Network) - 수십에서 수백 대의 SAN 스토리지를 데이터 서버에 연결해 총괄적으로 관리해주는 네트워크



하둡 - HDFS와 map-red 제공

##### HDFS

Hadoop Distributed File System(분산저장환경)

저사양 서버를 이용해 스토리지 구성할 수 있다

* 장애를 빠른시간에 인지하고 대처
* 스트리밍 방식의 데이터 접근
* 대용량 데이터 저장
* 데이터 무결성(수정 불가 / 이동,삭제,복사 가능) -> 변조 불가



* 64MB 블록으로 나눠져 분산된 서버에 저장

##### hdfs 명령어

```
[root@hadoopserver2 ~]# hadoop fs -mkdir /mydir
[root@hadoopserver2 ~]# hadoop fs -lsr /
[root@hadoopserver2 ~]# hadoop fs -du
[root@hadoopserver2 ~]# hadoop fs -put anaconda-ks.cfg /mydir
[root@hadoopserver2 ~]# hadoop fs -ls /mydir
[root@hadoopserver2 ~]# hadoop fs -cat /mydir/anaconda-ks.cfg
[root@hadoopserver2 ~]# hadoop fs -mv /mydir/anaconda-ks.cfg /user/root/mydir
[root@hadoopserver2 ~]# hadoop fs -rmr /mydir


```



---------



## 04 맵리듀스

Map과 Reduce 두 가지 단계로 데이터 처리

map(key value로 분류), reduce(뭉치거나 줄이는 과정)



```
[root@hadoopserver2 hadoop-1.2.1]# hadoop fs -mkdir /data
[root@hadoopserver2 hadoop-1.2.1]# hadoop fs -mkdir /data/input
[root@hadoopserver2 hadoop-1.2.1]# hadoop fs -put t1.txt /data/input
[root@hadoopserver2 hadoop-1.2.1]# hadoop fs -put t2.txt /data/input
[root@hadoopserver2 hadoop-1.2.1]# hadoop fs -ls /data/input
[root@hadoopserver2 hadoop-1.2.1]# hadoop jar hadoop-examples-1.2.1.jar wordcount /data/input /data/output
=> map 먼저 실행


```



 