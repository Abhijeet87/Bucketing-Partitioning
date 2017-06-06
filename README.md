# Bucketing-Partitioning
Hive examples on Buckets and Partitions



<b>hive with bucketing </b>

```sql
drop table if exists format.Bucket_data;
create table format.Bucket_data(userid INT, firstname STRING, lastname STRING, email STRING, sex String, IP STRING)
CLUSTERED BY (userid) INTO 4 BUCKETS
ROW FORMAT DELIMITED
//Since its CSV, comma separated fields
FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE;

SET hive.enforce.bucketing=true;
LOAD DATA LOCAL INPATH 'Bucket_mock_data.csv' OVERWRITE INTO TABLE format.Bucket_data;
```
//SAVED IN -->/user/hive/warehouse/format.db/bucket_data/Bucket_mock_data.csv

Sampling:

```sql
SELECT * FROM format.Bucket_data TABLESAMPLE (BUCKET 1 OUT OF 4);
SELECT * FROM format.Bucket_data TABLESAMPLE (10 PERCENT);
```
Sample Output:
```sql
hive (default)> SELECT * FROM format.Bucket_data TABLESAMPLE (1 PERCENT);
OK
```
|bucket_data.userid|bucket_data.firstname|bucket_data.lastname|bucket_data.email|bucket_data.sex|bucket_data.ip
| -------- |:-------------------:| :----------------- |:---------------:|:-------------:|-------------:|
|1|	Appolonia|	McConnulty|	amcconnulty0@kickstarter.com|	Female|	160.188.24.166||
|2|	Halsey|	Gilpillan|	hgilpillan1@smugmug.com|	Male|	131.165.186.5|6
|3|	Maris|	Garstang|	mgarstang2@wsj.com|	Female|	34.58.251.20|3
|4|	Angelina|	Cuniam|	acuniam3@pinterest.com|	Female|	166.20.2.72.154
|5|	Dalli|	Khristoforov|	dkhristoforov4@time.com|	Male|	133.159.115.130
|6|	Johnnie|	Franzotto|	jfranzotto5@meetup.com|	Male|	127.198.77.169
|7|	Hadley|	Cunah|	hcunah6@japanpost.jp|	Male|	195.54.84.13|
|8|	Haze|	Ilyinykh|	hilyinykh7@goo.ne.jp|	Male|	177.2.2.20
|9|	Aluin|	Pitchford|	apitchford8@biblegateway.com|	Male|	219.169.251.10
|10|	Henrik|	Breukelman|	hbreukelman9@nature.com|	Male|	29.66.228.19
Time taken: 0.034 seconds, Fetched: 10 row(s)


**Bucketing and Partitioning**

```sql
drop table if exists format.Bucket_PTN_data;
create table format.Bucket_PTN_data(userid INT, firstname STRING, lastname STRING, email STRING, IP STRING)
PARTITIONED BY (sex String)
CLUSTERED BY (userid) INTO 4 BUCKETS
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE;


SET hive.enforce.bucketing=true;
set hive.exec.dynamic.partition.mode=nonstrict;

create table format.Bucket_PTN_input(userid INT, firstname STRING, lastname STRING, email STRING,sex STRING, IP STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',' 
STORED AS TEXTFILE;

LOAD DATA LOCAL INPATH 'Bucket_mock_data.csv' OVERWRITE INTO TABLE format.Bucket_PTN_input;

INSERT OVERWRITE TABLE format.Bucket_PTN_data PARTITION (sex) select userid,firstname,lastname,email, ip,sex from format.Bucket_PTN_input;
```
OUTPUT:

```sql
Kill Command = /usr/lib/hadoop/bin/hadoop job  -kill job_1492370298181_0037
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 4
2017-06-05 17:52:28,537 Stage-1 map = 0%,  reduce = 0%
2017-06-05 17:52:34,885 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 1.14 sec
2017-06-05 17:52:45,850 Stage-1 map = 100%,  reduce = 25%, Cumulative CPU 2.61 sec
2017-06-05 17:52:48,085 Stage-1 map = 100%,  reduce = 50%, Cumulative CPU 3.95 sec
2017-06-05 17:52:49,158 Stage-1 map = 100%,  reduce = 75%, Cumulative CPU 5.33 sec
2017-06-05 17:52:50,216 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 6.73 sec
MapReduce Total cumulative CPU time: 6 seconds 730 msec
Ended Job = job_1492370298181_0037
Loading data to table format.bucket_ptn_data partition (sex=null)
	 Time taken for load dynamic partitions : 207
	Loading partition {sex=Female}
	Loading partition {sex=Male}
	 Time taken for adding to write entity : 1

[cloudera@quickstart Hive_Bucket_Practise]$ hadoop fs -ls /user/hive/warehouse/format.db/bucket_ptn_data/
Found 2 items
drwxrwxrwx   - cloudera supergroup          0 2017-06-05 17:52 /user/hive/warehouse/format.db/bucket_ptn_data/sex=Female
drwxrwxrwx   - cloudera supergroup          0 2017-06-05 17:52 /user/hive/warehouse/format.db/bucket_ptn_data/sex=Male
[cloudera@quickstart Hive_Bucket_Practise]$ hadoop fs -ls /user/hive/warehouse/format.db/bucket_ptn_data/sex=Female;
Found 4 items
-rwxrwxrwx   1 cloudera supergroup       7343 2017-06-05 17:52 /user/hive/warehouse/format.db/bucket_ptn_data/sex=Female/000000_0
-rwxrwxrwx   1 cloudera supergroup       7167 2017-06-05 17:52 /user/hive/warehouse/format.db/bucket_ptn_data/sex=Female/000001_0
-rwxrwxrwx   1 cloudera supergroup       6823 2017-06-05 17:52 /user/hive/warehouse/format.db/bucket_ptn_data/sex=Female/000002_0
-rwxrwxrwx   1 cloudera supergroup       7507 2017-06-05 17:52 /user/hive/warehouse/format.db/bucket_ptn_data/sex=Female/000003_0
```
