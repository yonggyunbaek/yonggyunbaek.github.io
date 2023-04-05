---
title: "hive create table error(field.delim)"
excerpt: "field.delim"

categories: [cloudera]
tag: [hive]

toc: true
toc_sticky: true

date: 2023-04-05
last_modifed_at: 2023-04-05
---
작업배경   
* 테이블 migration 진행   
* 기존 클러스터에서 show create 이용해서 ddl 추출 후 신규 클러스터에서 실행
* Error: Error while compiling statement: FAILED: ParseException line 8:1 cannot recognize input near '',\n'' 'serialization' '.' in table properties list (state=42000,code=40000)

* * *

## 1. 추출한 ddl 확인
```SQL
CREATE EXTERNAL TABLE `default`.`customers_4`(
  `name` string,
  `purchanse_count` int)
ROW FORMAT SERDE
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
  'field.delim'='^G',
  'serialization.format'='\n')
STORED AS INPUTFORMAT
  'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  '/warehouse/tablespace/external/hive/customers_4'
TBLPROPERTIES (
  'bucketing_version'='2',
  'external.table.purge'='true',
  'transient_lastDdlTime'='1680698916');
```

## 2. 실제 데이터 구분자 확인
```
a1
b2
c3
```

## 3. 구분자 에 대한 정보 확인

참고: https://www.fileformat.info/info/unicode/char/0007/index.htm   
C/C++/Java source code	"\u0007"   

## 4. DDL 수정
```SQL
CREATE EXTERNAL TABLE `default`.`customers_4`(
  `name` string,
  `purchanse_count` int)
ROW FORMAT SERDE
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
------------------------------
  'field.delim'='\u0007',
------------------------------
  'serialization.format'='\n')
STORED AS INPUTFORMAT
  'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  '/warehouse/tablespace/external/hive/customers_4'
TBLPROPERTIES (
  'bucketing_version'='2',
  'external.table.purge'='true',
  'transient_lastDdlTime'='1680698916');
```

## 5. DDL 실행 후 show create로 확인
### 1) beeline cli 출력 
```shell
+----------------------------------------------------+
|                   createtab_stmt                   |
+----------------------------------------------------+
| CREATE EXTERNAL TABLE `default`.`customers_4`(     |
|   `name` string,                                   |
|   `purchanse_count` int)                           |
| ROW FORMAT SERDE                                   |
|   'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'  |
| WITH SERDEPROPERTIES (                             |
|   'field.delim'='',                               |
|   'serialization.format'='\n')                     |
| STORED AS INPUTFORMAT                              |
|   'org.apache.hadoop.mapred.TextInputFormat'       |
| OUTPUTFORMAT                                       |
|   'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat' |
| LOCATION                                           |
|   'hdfs://<host>:8020/warehouse/tablespace/external/hive/customers_4' |
| TBLPROPERTIES (                                    |
|   'bucketing_version'='2',                         |
|   'external.table.purge'='true',                   |
|   'transient_lastDdlTime'='1680703862')            |
+----------------------------------------------------+
```
### 2) beeline 파일 output
```vi
+----------------------------------------------------+
|                   createtab_stmt                   |
+----------------------------------------------------+
| CREATE EXTERNAL TABLE `default`.`customers_4`(     |
|   `name` string,                                   |
|   `purchanse_count` int)                           |
| ROW FORMAT SERDE                                   |
|   'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'  |
| WITH SERDEPROPERTIES (                             |
|   'field.delim'='^G',                               |
|   'serialization.format'='\n')                     |
| STORED AS INPUTFORMAT                              |
|   'org.apache.hadoop.mapred.TextInputFormat'       |
| OUTPUTFORMAT                                       |
|   'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat' |
| LOCATION                                           |
|   'hdfs://<host>:8020/warehouse/tablespace/external/hive/customers_4' |
| TBLPROPERTIES (                                    |
|   'bucketing_version'='2',                         |
|   'external.table.purge'='true',                   |
|   'transient_lastDdlTime'='1680703862')            |
+----------------------------------------------------+
```

### 3) hue에서 실행결과
```SQL
CREATE EXTERNAL TABLE `default`.`customers_4`(
`name` string,
`purchanse_count` int)
ROW FORMAT SERDE
'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
'field.delim'='',
'serialization.format'='\n')
STORED AS INPUTFORMAT
'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT
'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
'hdfs://<host>:8020/warehouse/tablespace/external/hive/customers_4'
TBLPROPERTIES (
'bucketing_version'='2',
'external.table.purge'='true',
'transient_lastDdlTime'='1680703862')
```