---
title: "csv파일 spark이용해서 kudu저장"
excerpt: "csvsparkkdudu"

categories: [cloudera]
tag: [spark, kudu]

toc: true
toc_sticky: true

date: 2023-04-18
last_modifed_at: 2023-04-18
---

# csv --> spark --> kudu
## 1. Load and prepare the CSV data   



```txt
/* kudutest.csv
A,B,NUM,DT
a,b,1,2023-02-27 14:52:06
c,d,2,2023-02-27 15:00:00
e,f,3,2023-02-27 16:00:00
*/
```

```shell
HADOOP_USER_NAME=hdfs spark3-shell --jars /opt/cloudera/parcels/CDH/lib/kudu/kudu-spark3_2.12.jar
```

```scala
import org.apache.spark.sql.types._

val kdtest = spark.sqlContext.read.format("csv").option("header","true").option("inferSchema","true").load("/tmp/kudutest.csv")
//val kdtest = spark.sqlContext.read.format("csv").option("header","true").schema(customschema).load("/tmp/kudutest.csv")
kdtest.printSchema
kdtest.createOrReplaceTempView("kdtest")

/*
scala> spark.sql("SELECT count(*) FROM kdtest").show()
+--------+
|count(1)|
+--------+
|       3|
+--------+


scala> spark.sql("SELECT * FROM kdtest").show()
+---+---+---+-------------------+
|  A|  B|NUM|                 DT|
+---+---+---+-------------------+
|  a|  b|  1|2023-02-27 14:52:06|
|  c|  d|  2|2023-02-27 15:00:00|
|  e|  f|  3|2023-02-27 16:00:00|
+---+---+---+-------------------+
*/
```

## 2. Load and prepare the Kudu table
1번 쉘 세션 유지한 상태에서 추가 수행

```scala
import collection.JavaConverters._
import org.apache.kudu.client._
import org.apache.kudu.spark.kudu._
val kuduContext = new KuduContext("<hostname>:7051", spark.sparkContext)

kuduContext.tableExists("csvsparkkudu")

if(kuduContext.tableExists("csvsparkkudu")) {
	kuduContext.deleteTable("csvsparkkudu")
}

//kudu key column --> nullable = false
val customschema = StructType(Array(StructField("A",StringType, nullable = true),
StructField("B",StringType, nullable = false),
StructField("NUM", IntegerType, nullable = false),
StructField("DT", StringType, nullable = true)))



kuduContext.createTable("default.csvsparkkudu", customschema,
  /* primary key */ Seq("B", "NUM"),
  new CreateTableOptions()
    .setNumReplicas(3)
    .addHashPartitions(List("B").asJava, 4))


kuduContext.insertRows(kdtest, "default.csvsparkkudu")
// Create a DataFrame that points to the Kudu table we want to query.
val csvsparkkudu = spark.read
	.option("kudu.master", "<hostname>:7051")
	.option("kudu.table", "default.csvsparkkudu")
	// We need to use leader_only because Kudu on Docker currently doesn't
	// support Snapshot scans due to `--use_hybrid_clock=false`.
	.option("kudu.scanLocality", "leader_only")
	.format("kudu").load
csvsparkkudu.createOrReplaceTempView("default.csvsparkkudu")
spark.sql("SELECT count(*) FROM default.csvsparkkudu").show()
spark.sql("SELECT * FROM default.csvsparkkudu LIMIT 5").show()
```

## 3. Impala로 확인
INVALIDATE METADATA 수행하면 external table 생성되는 것으로 보임 (추가 확인 필요)