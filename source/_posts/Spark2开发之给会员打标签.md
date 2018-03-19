---
title: Spark开发之给会员打标签
categories:
- Spark
---

## 一、开发需求
>* 根据会员的基础信息(性别、年龄、生日等)和Pos消费记录，给每个会员打标签。
>* 标签分为如下几类：`人口属性`、`地域属性`、`购物偏好`、`RFM基础标签`、`购物行为`。

![](http://data.hiqiuyi.cn:8001/2018-03-19/tag.png)

## 二、数据准备
>* 生成hive宽表：`会员基础信息表`左关联`Pos消费记录表`，关联字段取会员id。

## 三、开发思路
>1. spark.sql读取宽表，根据会员id分组宽表记录。一个分组对应一个会员的所有消费记录和会员基础信息。
>2. 根据分组记录，给该会员打标签。
>3. 最后将会员标签存入HBASE,以会员id作为key。

## 四、参考代码
### SparkSession配置 
```scala
        val spark = SparkSession.builder()
            .appName("MarkTag")
            .config("mapred.input.dir.recursive","true")
            .config("mapreduce.input.fileinputformat.input.dir.recursive","true")
            .enableHiveSupport()
            .getOrCreate()
```
其中`mapred.input.dir.recursive`和`mapreduce.input.fileinputformat.input.dir.recursive`是为了使spark.sql执行hive-sql时，能够处理hive中LOCATION对应的数据存放在子目录下的情况。

### HBASE配置
```scala
        val conf = HBaseConfiguration.create()
        conf.set(TableOutputFormat.OUTPUT_TABLE, "dw:vips")  //保存会员标签的表
        conf.set("mapreduce.job.outputformat.class",  "org.apache.hadoop.hbase.mapreduce.TableOutputFormat")
```

### 根据会员id分组
```scala
val sql = s"select ${fieldStr} from dw.v_ysd_tag_aud_pos_info" 
val rows = spark.sql(sql)

var vipTag = rows.rdd.map(item => {
    // 数据预处理TODO
    dosomethong()    
    
    (audienceId, posdata)
}).groupByKey().map(item =>{
    // 执行打标签TODO
    val baseTagStr = getBaseTag()
    
    val hBaseKey = item._1  // 会员id作为key
    var p = new Put(hBaseKey.getBytes)
    p.addColumn("bt".getBytes, "base_tag".getBytes, baseTagStr.getBytes)
    val imPut = (new ImmutableBytesWritable(Bytes.toBytes(hBaseKey)), p)
    imPut
})
```

### 保存标签到HBASE
```scala
vipTag.saveAsNewAPIHadoopDataset(conf)
```
### shell查看hbase
```shell
scan 'dw:vips', {'LIMIT' => 1}
```
![](http://data.hiqiuyi.cn:8001/2018-03-19/result.png)
