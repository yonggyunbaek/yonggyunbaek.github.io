---
title: "hive,impala - jdbc,odbc 연결"
excerpt: "jdbc,odbc 커넥션 테스트"

categories: [cloudera]
tag: [impala,hive]

toc: true
toc_sticky: true

date: 2023-02-07
last_modifed_at: 2023-02-07
---



## 1. cloudera hive,impala JDBC,ODBC 다운로드
 https://www.cloudera.com/downloads/connectors/impala/jdbc/2-6-29.html.html  
 https://www.cloudera.com/downloads/connectors/impala/odbc/2-6-17.html  
 https://www.cloudera.com/downloads/connectors/hive/jdbc/2-6-21.html  
 https://www.cloudera.com/downloads/connectors/hive/odbc/2-6-13.html  

* hive, impala 서비스 ldap 설정 상태

### 1) Impala JDBC
* * *
```
pip3 install JayDebeAPI
```
* * *

```python
import jaydebeapi

conn = jaydebeapi.connect("com.cloudera.impala.jdbc41.DataSource",
                          "jdbc:impala://<hostname>:21050/;AuthMech=3;",
                          {'UID': "<user>", 'PWD': "<passwd>"},
                          '/home/cloudera/ImpalaJDBC41.jar')
curs = conn.cursor()

curs.execute("show databases")
curs.fetchall()

curs.close()
conn.close()​
```

```python
​import jaydebeapi

conn = jaydebeapi.connect("com.cloudera.hive.jdbc.HS2DataSource","jdbc:hive2://<host>:10000/;AuthMech=3",{'UID':"<user>", 'PWD':"<passwd>"},'/root/HiveJDBC-2.6.21.1025/HiveJDBC41.jar')


# /root/HiveJDBC-2.6.21.1025
crsr=conn.cursor()
#crsr.execute('select * from test01.hive_exteral_test_01;')
crsr.execute('show databases;')

print(crsr.fetchall())

```

### 2) ODBC
* ODBC 설치
```
yum --nogpgcheck localinstall ClouderaHiveODBC-2.6.13.1013-1.x86_64.rpm
yum --nogpgcheck localinstall ClouderaImpalaODBC-2.6.11.1011-1.x86_64.rpm
```

* pyodbc 설치
```
pip3 install pyodbc
```

* test.py 커넥션 테스트  

```python
# hive ODBC
import pyodbc

connString = pyodbc.connect('Driver=/opt/cloudera/hiveodbc/lib/64/libclouderahiveodbc64.so;Host=<hostname>;Port=10000;AuthMech=3;UID=<user>;PWD=<passwd>;',autocommit=True)
crsr=connString.cursor()
crsr.execute('show tables;')
print(crsr.fetchall())


------------------------------
$ python3 test.py
[('ranger_atlas_test', ), ('ranger_atlas_test_2', )]
```
  
```python
# impala ODBC
import pyodbc

connString = pyodbc.connect('Driver=/opt/cloudera/impalaodbc/lib/64/libclouderaimpalaodbc64.so;Host=<hostname>;Port=10000;AuthMech=3;UID=<user>;PWD=<passwd>;',autocommit=True)
crsr=connString.cursor()
crsr.execute('select * from testtable;')
print(crsr.fetchall())


------------------------------
$ python3 test.py
[(1, 1)]
```