---
title: "hive metastore에서 테이블 리스트 추출"
excerpt: "hive metastore에서 테이블 리스트 추출"

categories: [cloudera]
tag: [hive, mysql]

toc: true
toc_sticky: true

date: 2023-03-19
last_modifed_at: 2023-03-19
---

**metastore 가 mariadb로 설치된 상태**
* * *

## 1. db_name tbl_name 형태로 추출
```SQL
select name AS db_name,tbl_name FROM TBLS INNER JOIN DBS ON TBLS.DB_ID = DBS.DB_ID WHERE DBS.name='default'
```

### 1) 응용(테이블 리스트 추출해서 msck repair 하는 쿼리로 만들기)
```shell
mysql -u${user} -p${password} metastore -N -e "select concat('msck repair table ', DBS.name,'.',TBLS.tbl_name,' drop partitions;') FROM TBLS INNER JOIN DBS ON TBLS.DB_ID = DBS.DB_ID WHERE DBS.name='<db_name>';" > hive_tables.sql

# -N: header 생략
```

#### 출력 내용 예시
vi hive_tables.sql
```text
msck repair table default.customers drop partitions;
msck repair table default.customers_2 drop partitions;
```

