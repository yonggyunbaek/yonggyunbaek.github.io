---
title: "impala 통계 정보에 대하여(추가 예정)"
excerpt: "kudu 테이블 통계 정보 생성"

categories: [cloudera]
tag: [impala]

toc: true
toc_sticky: true

date: 2022-12-13
last_modifed_at: 2022-12-13
---


## 1. compute stats
## 2. compute incremental stats
## 3. sampling
## 4. ALTER
### 1) row count 삽입

```SQL
alter table <TABLE_NAME> set tblproperties('numRows'=<rowcount>, 'STATS_GENERATED_VIA_STATS_TASK'='true');
```
### 2) column stats 삽입

```SQL
alter table <TABLE_NAME> set set column stats <columnname> ('numDVs'='<DV>', 'numNulls'='<numN>', 'maxsize'='<Max_size>, 'avgsize'='<Avg_size>');
```
* column type에 따라서 maxsize, avgsize 는 update 할 수 없음(고정값)

