> 需求背景：从hdfs指定路径读取文件，将文件内容转为字符串

> 试错方案：最初的方法是使用hdfs提供的API，按行读取，发现读取太慢，故放弃

> 解决方案：使用sparkContext.textFile()方法来读取，然后collect到driver端，遍历collect得到的列表，append生成字符串

```scala
val regionJsonPath: String = "hdfs:///prod/data_platform/date_warehouse/ETL/bi/jars/test/mall/regions/"
var regionLines: Array[String] = spark.sparkContext.textFile(regionJsonPath).collect()
var regionString: String = ""
for (line <- regionLines) {
	regionString ++= line
}
println(regionString)
```

#####sparkContext.textFile()读取文件原理

上面是使用textFile()方法来读取文件，textFile()具体实现如下