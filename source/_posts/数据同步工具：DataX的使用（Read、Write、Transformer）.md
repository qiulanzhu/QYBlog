title: 数据同步工具：DataX的使用（Read、Write、Transformer）
categories:
- 工具使用
---
## DataX支持的数据库
<div align=center>![](http://data.hiqiuyi.cn:8001/2018-03-30/datax_surpport.png)</div>

## DataX速度快的原因
> DataX同步数据，是直接对数据源进行读写操作。而非`select`再`insert`

## DataX安装
> 环境要求
> * JDK(1.6以上)
> * Python

> DataX工具包下载：
<div align=center>![下载地址](http://datax-opensource.oss-cn-hangzhou.aliyuncs.com/datax.tar.gz)</div>

> 下载后解压至本地某个目录，进入bin目录，即可运行同步作业：
> 自检脚本:
> `python {YOUR_DATAX_HOME}/bin/datax.py {YOUR_DATAX_HOME}/job/job.json`
> 下图表示自检成功。
<div align=center>![](http://data.hiqiuyi.cn:8001/2018-03-30/run_result.png)</div>

## 从Hbase同步数据到Mysql,实现数据增量同步
job.json配置文件如下:
ps:json文件中不能有注释，为了便于说明才加上的，使用时应删除注释。
```
{
    "job": {
        "setting": {
            "speed": {
                "byte": 10485760
            },
            "errorLimit": {
                "record": 0,
                "percentage": 0.02
            }
        },
        "content": [
            {
                "reader": {
                  "name": "hbase11xreader",                 // 配置成读Hbase
                  "parameter": {
                      "hbaseConfig": {
                       "hbase.rootdir": "$hbase_dir",
                       "hbase.cluster.distributed": "true",
                       "hbase.zookeeper.quorum": "$hbase_zk"
                      },
                      "table": "$hbase_table",
                      "encoding": "utf-8",
                      "mode": "normal",
                      "column": [
                            {
                                "name": "i: j_id",
                                "type": "string"
                            },
                            {
                                "name": "i: t_id",
                                "type": "string"
                            },
                            {
                                "name": "i: a_id",
                                "type": "string"
                            },
                            {
                                "value": "$current_time",
                                "type": "string"
                            }

                      ],
                      "range": {
                          "startRowkey": "",
                          "endRowkey": "",
                          "isBinaryRowkey": true
                      }
                  }
                },
               "writer": {
                   "name": "mysqlwriter",               // 配置成写Mysql
                   "parameter": {
                       "writeMode": "replace",
                       "username": "$db_user",
                       "password": "$db_password",
                       "column": [
                           "job_id",
                           "task_plan_id",
                           "audience_id",
                           "created_time"
                       ],
                       "session": [
                         "set session sql_mode='ANSI'"
                       ],
                       "preSql": [
                           ""
                       ],
                       "connection": [
                           {
                               "jdbcUrl": "jdbc:mysql://$db_host:$db_port/$db_name?characterEncoding=utf8",
                               "table": [
                                   "$mysql_table"
                               ]
                           }
                       ]
                   }
                },
               "transformer": [                         // 实现增量同步，只同步当天数据
                  {
                      "name": "dx_filter",
                      "parameter":
                      {
                        "columnIndex":0,
                        "paras":["not like","$cur_date"]
                      }
                  }
               ]
            }
        ]
    }
}


```



