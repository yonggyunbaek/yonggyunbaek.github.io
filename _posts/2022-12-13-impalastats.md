---
title: "impala 통계 정보에 대하여"
excerpt: "kudu 테이블 통계 정보 생성"

categories: [cloudera]
tag: [impala]

toc: true
toc_sticky: true

date: 2022-12-13
last_modifed_at: 2023-01-02
---

**impala는 테이블 사이즈가 큰지 작은지, distinct 값들이 많은지 적은지 등에 대한 정보가 있다면 join 쿼리나 insert 작업을 적절하게 구조화하고 병렬화할 수 있다.**

## 1. compute stats 
* * *
```SQL
COMPUTE STATS [db_name.]table_name  [ ( column_list ) ] 
-- column_list 여러개면 ,로 구분
```
* * *
* 컬럼 리스트 미지정시 모든 컬럼에 대한 통계를 계산한다.   
* 지정한 컬럼이 통계 계산을 지원하지 않는 type이거나 partitioning 컬럼일 경우 에러가 발생한다.

#### 1) compute incremental stats
* * *
```SQL
COMPUTE INCREMENTAL STATS [db_name.]table_name [PARTITION (partition_spec)]
```
* * *
* incremental 만 partition지정 가능하다   
* <font color='LightCoral'>compute stats와 compute incremental stats를 한 테이블에 같이 사용하지 않는다. 둘 중 하나로 전환할 경우 drop stats 실행 후 전환한다.</font>

#### 2) sampling (문서만 확인 - test 필요)
* 설정 enable 필요함   
config 변경 --enable_stats_extrapolation   또는   
특정 테이블만 설정  

* * *
```SQL 
ALTER TABLE mytable test_table SET TBLPROPERTIES("impala.enable.stats.extrapolation"="true")

COMPUTE STATS [db_name.]table_name  [ ( column_list ) ] [TABLESAMPLE SYSTEM(percentage) [REPEATABLE(seed)]]
-- 10%는 TABLESAMPLE SYSTEM(10) 
-- seed 임의의 양의 정수 - 쿼리가 다시 실행될 때 매번 동일한 데이터 파일 집합을 선택하도록 하는 옵션
```
* * *
## 2. ALTER 
#### 1) row count update
* * *
```SQL
alter table <TABLE_NAME> set tblproperties('numRows'=<rowcount>, 'STATS_GENERATED_VIA_STATS_TASK'='true');
```
* * *
#### 2) column stats update
* * *
```SQL
alter table <TABLE_NAME> set set column stats <columnname> ('numDVs'='<DV>', 'numNulls'='<numN>', 'maxsize'='<Max_size>, 'avgsize'='<Avg_size>');
```
* * *
* <font color='LightCoral'> column type에 따라서 maxsize, avgsize 는 update 할 수 없다(고정값) </font>

## 3. 통계정보 확인
* * *
```SQL
show table stats [db_name.]table_name
show column stats [db_name.]table_name
-- '-1'이 있으면 통계정보 없는 상태
```
* * *
또는 쿼리 실행전 explain 했을때 통계정보가 없다는 warning이 발생하지 않으면 됨