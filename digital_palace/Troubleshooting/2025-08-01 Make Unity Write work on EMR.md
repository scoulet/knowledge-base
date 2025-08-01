
## 1. Unity dependencies to sbt

```scala 
// Unity dependencies  
"io.unitycatalog" %% "unitycatalog-spark" % "0.2.1" excludeAll (  
  ExclusionRule(organization = "org.apache.logging.log4j"),  
  ExclusionRule(organization = "org.slf4j", name = "slf4j-api")  
)
```



## 2. Add the rights options to the spark session


best practice : add a little test to only add those properties if you are actually going to save on unity


## 3. Save as Table


```scala
df.write  
  .option("optimize", "true")  
  .option("maxFileSize", "128MB")  
  // max record by file  
  // .option("maxRecordsPerFile", 1000000)  .option("maxPartitionBytes", 1024 * 1024 * 1024) // 1 G")  
  // .option("delta.targetFileSize", 1024 *1024 * 128) // 128 M  // .option("delta.tuneFileSizesForRewrites", "true") // tune file sizes for rewrites  .format("delta")  
  .mode("overwrite")  
  .partitionBy(partitionsList: _*)
.option("path", pathDelta)  
.saveAsTable(s"${unityCatalog}.${unitySchema}.${tableName}"  
)
```
(?) This gonna create an external location 