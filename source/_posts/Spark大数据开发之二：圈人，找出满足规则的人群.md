---
title: Spark大数据开发之二：圈人，找出满足规则的人群
categories:
- Spark
---
## 一、开发需求
> * 根据任务规则，圈出符合规则的所有会员，以方便运营针对这些会员做特定的营销活动。
> * 一次有多个任务需要圈人。
> * 常见任务规则如下。

<div align=center>
![任务规则](http://data.hiqiuyi.cn:8001/2018-03-23/task_rule.png)
</div>


## 二、数据准备
> 1. Mysql中的规则表。
> 2. Hbase中的会员标签表。[Spark大数据开发之一：给会员打标签](http://data.hiqiuyi.cn/2018/03/22/Spark2%E5%BC%80%E5%8F%91%E4%B9%8B%E7%BB%99%E4%BC%9A%E5%91%98%E6%89%93%E6%A0%87%E7%AD%BE/)

## 三、开发思路
> 1. 读取所有需要圈人的任务ID,以及每个任务对应的规则。
> 2. 遍历会员标签表。
> 3. 对每一个会员，依次判断符合那些任务规则。
> 4. 将符合条件的会员保存到Hbase，RowKey为[日期~任务ID~会员ID]

## 四、参考代码
### 1. 读取Mysql
```scala
import java.sql.{Connection, DriverManager, Statement}

val connection = DriverManager.getConnection("db.url", "db.user", "db.password")
val statement: Statement = connection.createStatement()
var sql = ""    // 查询任务id和任务规则
var resultSet = statement.executeQuery(sql)
while (resultSet.next()) {
    // resultSet.getString("task_id")
    // ...
}
```
### 2. 遍历会员标签表
>创建`Hive会员标签表`映射`Hbase会员标签表`，通过SparkSession.sql执行hive-sql查询`Hive会员标签表`,遍历所有会员。

```scala
val spark = SparkSession.builder().appName("TaskCircleCommon").enableHiveSupport().getOrCreate()
val sql = "SELECT vip_id, tag_info from dw.vips"
val rows = spark.sql(sql)
val rddData = rows.rdd.flatMap(item => {
    // 圈人逻辑，判断会员符合哪些任务规则
    
    
    // 返回待保存到Hbase的数据结构
    var putArr: ArrayBuffer[(ImmutableBytesWritable,Put)] = ArrayBuffer()
    val hBaseKey = curDate + "~" + taskId + "~" + audienceId
    var p = new Put(hBaseKey.getBytes)
    p.addColumn("i".getBytes, "a_id".getBytes, audienceId.getBytes)
    val imPut = (new ImmutableBytesWritable(Bytes.toBytes(hBaseKey)), p)
    putArr += imPut
    putArr
})
```

### 3. 保存结果到Hbase
```scala
//hbase配置
val conf = HBaseConfiguration.create()
conf.set(TableOutputFormat.OUTPUT_TABLE, "dw:c_plan_targeting")
conf.set("mapreduce.job.outputformat.class", "org.apache.hadoop.hbase.mapreduce.TableOutputFormat")

// 保存
rddData.saveAsNewAPIHadoopDataset(conf)
```

### 4. 前端显示
<div align=center border="8">
![圈人结果](http://data.hiqiuyi.cn:8001/2018-03-23/task_show.png)
<div>


