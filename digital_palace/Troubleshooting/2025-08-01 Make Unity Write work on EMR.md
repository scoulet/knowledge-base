
## 1. Unity dependencies to sbt

```scala 
// Unity dependencies  
"io.unitycatalog" %% "unitycatalog-spark" % "0.2.1" excludeAll (  
  ExclusionRule(organization = "org.apache.logging.log4j"),  
  ExclusionRule(organization = "org.slf4j", name = "slf4j-api")  
)
```



## 2. Add the rights options to the spark session


/** Sets options for integrating Spark with Databricks Unity Catalog..  
  * @return  
  */  
def setUnityOptions(): SparkConfUtils = {  
  val unityCatalog = conf.getString("unity.unity-default-catalog")  
  val unityAuthOptions = UnityUtils.getUnityAuthOptionsFromVault(conf)  
  
  sparkConf  
    .set(  
      "spark.sql.defaultCatalog",  
      unityCatalog  
    )  
    .set(  
      "spark.sql.catalog.spark_catalog",  
      "org.apache.spark.sql.delta.catalog.DeltaCatalog"  
    )  
    .set(  
      s"spark.sql.catalog.${unityCatalog}",  
      "io.unitycatalog.spark.UCSingleCatalog"  
    )  
    .set(  
      s"spark.sql.catalog.${unityCatalog}.uri",  
      s"https://${conf.getString("unity.databricks-api-host")}/api/2.1/unity-catalog"  
    )  
    // .set(  
    //   s"spark.sql.catalog.${unityCatalog}.client_id",    //   unityAuthOptions("client_id")    // )    // .set(    //   s"spark.sql.catalog.${unityCatalog}.client_secret",    //   unityAuthOptions("client_secret")    // )    .set(  
      s"spark.sql.catalog.${unityCatalog}.token",  
      unityAuthOptions("token")  
    )  
    .set(  
      s"spark.sql.catalog.${unityCatalog}.auth.type",  
      unityAuthOptions("auth.type")  
    )  
    .set("spark.hadoop.fs.s3.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem")  
  Logger.info(  
    s"Set up unity spark conf with secrets ${unityAuthOptions  
      .map { case (k, v) =>  
        val masked = if (v.length <= 3) "*" * v.length  
        else "*" * (v.length - 3) + v.takeRight(3)  
        s"$k=$masked"  
      }  
      .mkString(", ")}"  
  )  
  this  
}
### to overlap the catalog setup directly on the EMR (exp: glue)

```scala

.set("spark.hadoop.hive.metastore.uris", "")
.set("spark.sql.catalogImplementation", "in-memory")
.set(
"hive.metastore.client.factory.class",
"org.apache.hadoop.hive.metastore.HiveMetaStoreClientFactory"
)```

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
(?) This gonna create an external location (à peaufiner)