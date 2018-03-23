---
title: Spark大数据开发之三：RDD输出结果存入Mysql
categories:
- Spark
---
## 一、开发需求
> 将Spark计算的结果存入Mysql

## 二、参考代码
> 1. RDD 转换成 DataFrame
> 2. DataFrame 保存到 Mysql
```scala
val df = spark.sqlContext.createDataFrame(audsRowRdd, schema)
val props = new Properties()
props.put("user", config.getString("db.user"))
props.put("password", config.getString("db.password"))
df.write.mode("append").jdbc(config.getString("db.url"), "mysql_table", props)
```

## 三、遇到的问题
> 在本地调试的时候，数据正常存入Mysql。在集群运行时，出现错误：`No suitable driver`。堆栈信息如下：
```
ERROR yarn.ApplicationMaster: User class threw exception: java.sql.SQLException: No suitable driver
java.sql.SQLException: No suitable driver
	at java.sql.DriverManager.getDriver(DriverManager.java:315)
	at org.apache.spark.sql.execution.datasources.jdbc.JDBCOptions$$anonfun$7.apply(JDBCOptions.scala:84)
	at org.apache.spark.sql.execution.datasources.jdbc.JDBCOptions$$anonfun$7.apply(JDBCOptions.scala:84)
	at scala.Option.getOrElse(Option.scala:121)
	at org.apache.spark.sql.execution.datasources.jdbc.JDBCOptions.<init>(JDBCOptions.scala:83)
	at org.apache.spark.sql.execution.datasources.jdbc.JDBCOptions.<init>(JDBCOptions.scala:34)
	at org.apache.spark.sql.execution.datasources.jdbc.JdbcRelationProvider.createRelation(JdbcRelationProvider.scala:53)
	at org.apache.spark.sql.execution.datasources.DataSource.write(DataSource.scala:518)
	at org.apache.spark.sql.DataFrameWriter.save(DataFrameWriter.scala:215)
	at org.apache.spark.sql.DataFrameWriter.jdbc(DataFrameWriter.scala:446)
	at com.yoorstore.bigdata.rr.SupplyCouponCircle$.main(SupplyCouponCircle.scala:945)
	at com.yoorstore.bigdata.rr.SupplyCouponCircle.main(SupplyCouponCircle.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.spark.deploy.yarn.ApplicationMaster$$anon$2.run(ApplicationMaster.scala:637)
```

## 四、分析问题
> 由于本地调试运行正常，而集群环境运行失败。排查集群环境对应的Mysql驱动包是否正常。

## 五、解决问题
> 将项目中用到的Mysql驱动包`mysql-connector-java-5.1.38.jar`复制到集群的每个主机上。比如每台主机的这个位置：`/usr/lib/mysql/mysql-connector-java-5.1.38.jar`
> 在spark-submit提交的时候添加参数`–driver-class-path /usr/lib/mysql/mysql-connector-java-5.1.38.jar`



