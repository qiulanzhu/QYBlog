---
title: Hbase中批量删除RowKey
categories: 
- Hbase
---

### 问题
在使用Hbase的时候常常遇到这样的情况：  
需要删除一段RowKey区间范围的数据,而不想truncate清空所有的数据。  
但Hbase只提供了删除一行RowKey的接口`deleteall ‘<table name>’, ‘<row>’`  

### 解决办法
写shell脚本批量删除。

### 脚本
创建脚本`batchDeleteRowKey.sh`

```shell
#!/bin/bash
tablename=$1
startrow=$2
endrow=$3
if [ $# -eq 0 ];then
echo "请输入表名，startRow,endRow"
exit 0
fi
echo "scan '${tablename}',{STARTROW=>'${startrow}',ENDROW=>'${endrow}'}" |hbase shell|awk -F ' ' '{print $1'\t'}'|uniq> ./file.txt
#删除前6行非表中数据
sed -i '1,6d' file.txt
#删除最后一行（空行）
sed -i '$d' file.txt
#删除最后一行（总条数）
sed -i '$d' file.txt
cat ./file.txt|awk '{print $1}'|while read rowvalue
do
echo -e "deleteall '${tablename}','${rowvalue}'" >> ./deleteRowFile.txt
done
echo "exit" >> ./deleteRowFile.txt

#执行步骤
# 第一步，生成需要删除的key（为避免线上执行失误 ，先检查deleteRowFile.txt中要删除的rowkey是否正确）
#sh batchDeleteRowKey.sh dw:member_profile  201816000 201816~~~
# 第二步，执行删除动作
#hbase shell ./deleteRowFile.txt
# 第三步，删除中间文件
#rm ./deleteRowFile.txt
#rm ./file.txt
```