# SparkCore

写Rdd的代码



## Rdd模板

```
import org.apache.spark.{SparkConf, SparkContext}

object SparkContextApp {
  def main(args: Array[String]): Unit = {
    
    val sparkconf = new SparkConf()
    
    //本地模式，两个核，网页上看appname
    /在spark-shell运行的时候再指定，这里的优先级要比在shell里更高，appname在shell里有内置
    //.setMaster("local[2]").setAppName("asd")
    
    //一个context运行在一个jvm。
    val sc = new SparkContext(sparkconf)


    sc.stop()
  }
}
```



## 创建RDD的方法

1.并行化数据，第二个参数是partiton分区数

```
val rdd = sc.parallelize(Array(1,2,3,4,5),2)
```

2.读取一个文件，返回一串RDD，第二个参数是最小分区数

```
val rdd = sc.textFile("file:///E:/asd.txt"，2)
```



## 读取SequencyFile

读取为key-value形式的RDD

```
//读sequenceFile，key（字节的形式,因为seq底层key是二进制的）和value的类型
//用textfile读，有部分数据会乱码
val file = sc.sequenceFile[BytesWritable,String]("/user/hive/warehouse/page_views_sequencefile/000000_0")
```

key以二进制读进来，要显示的化这样做

```
file.map(x=>(x._1.copyBytes(),x._2)).foreach(println)
```

然而hive表存为SequencyFile格式，key部分的信息不是表的数据，所以可以过滤掉。

```
file.map(x=>x._2.split("\t")).map(x=>(x(0),x(1))).foreach(println)
```





## 收集RDD

注意top和collect都会内存加载所有数据

```
//collect是action，把rdd转换为数组。
rdd.collect().foreach(println)
```

```
//取最大的几个数，返回数组
rdd.top(3)
```



## 存储RDD

1.存在文件系统，把RDD存为字符串

```
rdd.saveAsTextFile("file:///E:/asd.txt")
```

2.存在hdfs上

```
import org.apache.hadoop.fs.{FileSystem, Path}

//写到hdfs要检测文件是否存在，存在要删掉，不然报错
//这里没有new configuration是因为，下面sc.hadoopConfiguration用了默认的configuration
val uri = new URI("hdfs://192.168.137.190:9000" )

    val fileSystem = FileSystem.get(uri,sc.hadoopConfiguration,"hadoop")
    if (fileSystem.exists(new Path(args(1)))){
      fileSystem.delete(new Path(args(1)),true)
    }
```

这里的args（1）是自定义参数，和sparkconf的设置一样，我们在程序提交执行时，传入。



## sortBy

给RDD排序，结果仍是RDD，ascending: Boolean = true（默认升序）

```
sortBy(_._2,false)
```

还有scala的sortBy，默认升序，要降序颠倒之

```
sortBy(x => x._2).reverse
```



## 函数后加Partition区别

比如map和mapPartitions，foreach和foreachPartition

区别在于普通的是一条一条数据处理，效率不高，partition的是分区的数据全部一起处理，效率高。

​	但是也要注意，一次处理那么多数据，可能会导致内存溢出。



## coalesce和repartition

coalesce不带shuffle，repartition带shuffle。

coalesce只可缩小分区数，repartition可大可小

作用在一个RDD后面，返回新RDD。



## mapPartitionsWithIndex

返回数据的索引和对应得分区，分区是可迭代的。

```
stus.coalesce(2).mapPartitionsWithIndex((index, partition) => {
  val emps = new ListBuffer[String]
  while(partition.hasNext) {
    emps += ("~~~" + partition.next() + " , 新部门:[" + (index+1) + "]")
  }
  emps.iterator
}).foreach(println)
```



## RDD之WordCount

reduceByKey

```
val file = sc.textFile("file:///F:/asd.txt")
file.flatMap(_.split(",")).map((_,1)).reduceByKey(_+_).collect().foreach(println)
```



groupByKey

```
//[string,Iterable(int)]的形式
//(l1,CompactBuffer(1, 1, 1))
val file = sc.textFile("file:///F:/asd.txt")
file.flatMap(_.split(",")).map((_,1)).groupByKey().map(x=>(x._1,x._2.sum)).collect().foreach(println)
```



## RDD的普通join

```
//（"1"，("1", "豆豆")）
val g5 = sc.parallelize(Array(("1", "豆豆"), ("2", "牛哥"), ("34", "进取"))).map(x=>(x._1,x))

//（"34"，("34", "深圳", 18)）
val f11 = sc.parallelize(
  Array(("34", "深圳", 18), ("10", "北京", 2))
).map(x => (x._1,x))

//注意要join需要数据是key-value的形式，所以上面的要做类型转换
//join后的数据结构rdd.RDD[(String, ((String, String), (String, String, Int)))]
//（"34"，（("34", "进取")，("34", "深圳", 18)））
g5.join(f11).map(x => {
  x._1+","+ x._2._1._2 + "," + x._2._2._2 + "," + x._2._2._3
}).collect()
```



## RDD的Broadcastjoin

这种要用map的方式实现join，用大表map，在map里，用大表里的数据去匹配小表。

不会shuffle

```
//key-value形式，最佳实践
val g5 = sc.parallelize(Array(("1", "豆豆"), ("2", "牛哥"), ("34", "进取"))).collectAsMap()
//广播小表
val g5Broadcast = sc.broadcast(g5)

//（"34"，("34", "深圳", 18)）
val f11 = sc.parallelize(Array(("34", "深圳", 18), ("10", "北京", 2))).map(x => (x._1, x))

    f11.mapPartitions(partition => {
    
    	//"豆豆"，"牛哥"，"进取"
      val g5student = g5Broadcast.value
      
      //contains是判断map里的这个key是否有绑定，就是有值
      //针对每一次 for 循环的迭代, yield 会产生一个值，被循环记录下来 (内部实现上，像是一个缓冲区).
      // 当循环结束后, 会返回所有 yield 的值组成的集合.返回集合的类型与被遍历的集合类型是一致的
      
      //key="34"，value=("34", "深圳", 18)
      for ((key,value) <- partition if (g5student.contains(key)))
        yield(key,g5student.getOrElse(key,""),value._2)

    }
```



## RDD cache

是lazy的



## 累加器longAccumulator

一般调试代码时用，主要cache的使用

```
//累加器名字
val accum = sc.longAccumulator("asd")

val data = sc.parallelize(1 to 10)

val newData = data.map{x => {
      if(x%2 == 0){
        accum.add(1)
      }else 1
    }}
    
	//使用action操作触发执行
	//相当于把当前状态记录下来，还没计算，accum=0
    newData.cache().count
    
    //此时accum的值为5，是我们要的结果
    println(accum.value)

    //再执行一个action，accum没有变成10。
    //没有cache就是10
    newData.collect()
```





## 提交程序jar包执行

HADOOP_CONF_DIR是设置

master指定运行模式

--deploy-mode 默认是yarn-client模式

--class是程序入口

--name是程序名字

后面是jar包

两个路径是程序的两个入参

```
export HADOOP_CONF_DIR=/home/hadoop/app/hadoop-2.6.0-cdh5.7.0/etc/hadoop $SPARK_HOME/bin/spark-submit \ 
--master yarn \ 
##--deploy-mode cluster \ 
--class com.ruozedata.bigdata.core02.LogApp \ 
--name LogServerApp \ 
/home/hadoop/data/g5-spark-1.0.jar \ 
hdfs://hadoop000:9000/g5/generatefile hdfs://hadoop000:9000/g5/asd
```



## 序列化

```
//定义序列化方式
val sparkConf=new SparkConf().set("spark.serializer","org.apache.spark.serializer.KryoSerializer")
//要序列化注册的类
.registerKryoClasses(Array(classOf[Student]))
```

可以设置缓存的存储方式为序列化

```
studentRDD.persist(StorageLevel.MEMORY_ONLY_SER)
```



# SparkSQL



## Sql模板

```
import org.apache.spark.sql.SparkSession

object SparkSessionApp {
  def main(args: Array[String]): Unit = {
    //SparkSession也是调用sparkcontext
    val session = SparkSession.builder().appName("SparkSessionApp")
      .master("local[2]")
      .getOrCreate()

  // sparksession其实就是sparkcontext的上一层而已
  session.sparkContext.parallelize(Array(1,2,3)).foreach(println)

    session.stop()
  }
}
```



## 开启读hive

要在idea里用sparksql，需要把3个文件放到resources下

![1552959758433](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1552959758433.png)



然后在构建sparksession的时候开启

```
.enableHiveSupport()

//然后即可直接访问，解析成dataframe
sessionApp.sql("select * from page_views limit 10").show()
//也可以这样读hive，解析成dataframe
sessionApp.table("page_views limit 10").show()
```



## RDD转换为Dataframe（不定义schema信息）

需要导入隐式转换

```
import sessionApp.implicits._
val g5 = sessionApp.sparkContext.
  parallelize(Array(("1", "豆豆"), ("2", "牛哥"), ("34", "进取")))
    .toDF("id","name")
g5.show()
g5.printSchema()
```

结果如下

```
+---+----+
| id|name|
+---+----+
|  1|豆豆|
  |  2|牛哥|
  | 34|进取|
  +---+----+

root
|-- id: string (nullable = true)
|-- name: string (nullable = true)
```



## RDD转换为DataFrame（定义schema信息）

第一种是反射的方式去推测RDD的schema的具体类型。当schema信息可以确定的时候可以使用（需要定义caseclass）。

第二种是编程的方式去创建Datasets。当不能事先用caseclass定义schema信息时使用（）

```
import org.apache.spark.sql.types.{LongType, StringType, StructField, StructType}
import org.apache.spark.sql.{Row, SparkSession}
```



### 反射方式（case class）

​	schema信息在case class里定义

```
 def inferReflection(session: SparkSession)={
    import session.implicits._
    //创建RDD
    val info = session.sparkContext.textFile("file:///F:/asd.txt")

    //作用caseclass，这样相当于给每个参数有对应的类型和名称
    //scala映射支持自动转换一个包含case class 的RDD成为一个DataFrame
    //这相当于先作用case class再转成dataframe
    val df = info.map(_.split(",")).map(x=>Info(x(0),x(1),x(2).toLong)).toDF()
    
    
    
    df.groupBy("ip","domain").sum("liu").show()

  }

}
//这个case class定义的就是表的schema
case class Info(ip:String,domain:String,liu:Long)

```



​	

### 编程方式（StructType）

​	schema信息在structType里定义

​	没有用toDF（），所以不用导隐式转换

```
def programmatically(session: SparkSession)={

  val info = session.sparkContext.textFile("file:///F:/asd.txt")
  
  //1.从原始rdd创建一个row类型rdd
  val rdd = info.map(_.split(",")).map(x=>Row(x(0),x(1),x(2).toLong))
  
  // 2.创建一个schema由StructType代表匹配上一步的rdd
  val struct = StructType(Array(
    //第三个参数是数据是否能为空
    StructField("ip",StringType,false),
    StructField("domain",StringType,false),
    StructField("response",LongType,false)
  ))
  
  //3.把schema作用在rdd上，转换完成
  val df = session.createDataFrame(rdd,struct)

}
```





## 查询Dataframe的方式



### Dataframe加算子的方式

但是要使用聚合类算子要导东西

```
import org.apache.spark.sql.functions._
```

```
df.groupBy('domain).agg(sum("response") as ("rs") ).select("domain","rs").show()
```



### 注册视图用sql查

把Dataframe注册成一张零时表

```
//注册一个视图，相当于零时表，这样就可以用sql去查询
df.createOrReplaceTempView("info")
val infoDF=session.sql("select domain,sum(liu) from info group by domain")
```



### 用类Rdd的算子取dataframe的一列值

这里map的返回值是dataframe

```
infoDF.map(x=>x.getAs[String]("domain")).show()
```





## 注册一个函数

后面调用这样即可，select province(ip)

```
//函数名province,这里的ip是调用这个函数的传参
session.udf.register("province" ,(ip:String)=>{
  IPUtils.getInfo(ip)
})

object IPUtils{
  def getInfo(ip:String) = {
    ("深圳市","联通")
  }
}
```



## 自定义hive函数（java）



```
import org.apache.hadoop.hive.ql.exec.UDF;

//自定义hiveUDF
public class ruozeUDF extends UDF{

    public String evaluate(String name){
        return "ruozedata:" + name;
    }
// 静态方法是类方法，调用时不需要创建类实例。

    /**
     * 一般使用频繁的方法用静态方法，用的少的方法用动态的。静态的速度快，占内存。动态的速度相对慢些，但调用完后，立即释放类，可以节省内存，可以根据自己的需要选择是用动态方法还是静态方法。
     * @param args
     */
  public static void main(String[] args) {
        ruozeUDF udf = new ruozeUDF();
        System.out.println(udf.evaluate("doudou"));
  }
}
```



## Dataframe增加一列

```
//列名isp，放的值是什么
val asd = df.withColumn("isp",col("response")*3)
```



## Dataframe转换为rdd



```
//dataframe要作用一个RDD的算子需要加rdd
//getAs是拿到domain的所有值
infoDF.rdd.map(x=>x.getAs[String]("domain")).collect().foreach(println)

//索引方式也可以取值
infoDF.rdd.map(x=>x(0)).collect().foreach(println)
```



## sparkSql读取mysql



```
import java.util.Properties

val jdbcUrl = "jdbc:mysql://192.168.137.190:3306/ruozedb"
val table="dept"
val protity = new Properties()
protity.setProperty("user","ruoze")
protity.setProperty("password","123456")

sparkSession.read.jdbc(jdbcUrl,table,protity).show()
```



## sparkSQL读取文件为Dataframe



简化写法

```
sparkSession.read.text("/home/hadoop/data/student.txt").show（）
```

标准写法

```
sparkSession.read.format("text").load("/home/hadoop/data/student.txt").show
```

带参数的写法

```
sparkSession.read.format("text").option("path","/home/hadoop/data/student.txt").load().show()
```



## sparkSQL 写文件


标准写写法,save只能指定到目录，不能指定文件名


```
df.select("").write.format("json").save("").show()
```

指定写模式，追加，覆盖等

```
df.select("").write.format("json").mode("Overwrite").save("").show()
```

常用option

```
//自动推测schema类型
option("inferSchema","true")

//第一行数据做表头
option("header","true")

//指定分割符
option("sep",";")
```



## 写到Hive

默认是parquet格式

```
df.select("name").write.saveAsTable("xxx")
```



## sparkSql读取元数据



```
val catalog=sparkSession.catalog
catalog.listDatabases().show()

catalog.listTables("default").show()
catalog.listColumns("default","ruoze_emp").show()
```



## 读取文件作用schema



```
val ds = sparkSession.read.format("csv").as[Sales]
```



## sql启用broadcast join

1.参数类设置

这个值是阈值，小于这个阈值的表会用broadcast join。-1就是禁用。

```
spark.sql.autoBroadcastJoinThreshold
```

默认10M

```
scala> val threshold =  spark.conf.get("spark.sql.autoBroadcastJoinThreshold").toInt
threshold: Int = 10485760
```



2.或者直接在代码里用

```
val qBroadcastLeft = """
  SELECT /*+ BROADCAST (lf) */ *
  FROM range(100) lf, range(1000) rt
  WHERE lf.id = rt.id
"""
```

查看下执行计划，发现是BROADCASTjoin

```
sql(qBroadcastLeft).explain)
```



## 实现一个外部数据源源

要定义一个外部数据源可以参考JDBCrelation和JDBCrelationprovider。

JDBCrelation相当于用户可以使用里面的方法实现数据select，过滤和插入。

JDBCrelationprovider相当于用户和JDBCrelation的中转站，解析用户传入的参数。

![1553236778068](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1553236778068.png)



### 模仿JDBCrelationprovider



```
import org.apache.spark.sql.SQLContext
import org.apache.spark.sql.sources.{BaseRelation, RelationProvider, SchemaRelationProvider}
import org.apache.spark.sql.types.StructType

//jdbcrelationprovider
//必须叫DefaultSource，不然调用报错
//schemarelationprovider可以让用户传入schema
class DefaultSource extends RelationProvider  with SchemaRelationProvider{

  def createRelation(
                      sqlContext: SQLContext,
                      parameters: Map[String, String],
                      schema: StructType): BaseRelation = {
    //读文件就要path
    val path = parameters.get("path")
    path match {
    //如果有路径就进入JdbcCrelation
      case Some(x) => new TextDatasourceRelation(sqlContext, x, schema)
      case _ => throw new IllegalArgumentException("path is required for custom-text-datasource-api")

    }
  }

//如果用户没有传入schema，就是null
  override def createRelation(sqlContext: SQLContext, parameters: Map[String, String]): BaseRelation = {
        createRelation(sqlContext,parameters,null)

  }
}
```





### utils转换类型

```
import org.apache.spark.sql.types.{DataType, LongType, StringType}

object Utils {
  def castTo(value:String,dataType: DataType)={
    dataType match {
        // case LongType => value.toLong 一样
      case _:LongType => value.toLong
      case _:StringType => value
    }
  }

}
```



### JDBCRelation



```
import org.apache.spark.internal.Logging
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.sources._
import org.apache.spark.sql.types.{LongType, StringType, StructField, StructType}
import org.apache.spark.sql.{DataFrame, Row, SQLContext, SaveMode}

//jdbcrelation
class TextDatasourceRelation(override val sqlContext: SQLContext,
                             path:String,
                             userSchema:StructType)
//日志
  extends BaseRelation
    with TableScan
    with PrunedScan
    with PrunedFilteredScan
    with InsertableRelation
    with Logging {
  //可以用外面传进来StructType，也可以默认
  override def schema: StructType = {
    if (userSchema != null) {
      userSchema
    } else {
      StructType(
        //::和，皆可
        StructField("id",LongType,false)::
        StructField("name",StringType,false)::
          StructField("gender",StringType,false)::
          StructField("salary",LongType,false)::
          StructField("comm",LongType,false)::Nil
      )
    }
  }

  override def buildScan(): RDD[Row] = {

    logWarning("this is ruozedata custom buildScan().")

    //wholeTextFiles,返回前面文件名+后面内容，我们只需要内容
    val rdd = sqlContext.sparkContext.wholeTextFiles(path).map(x=>x._2)
    //返回的是schema的数组
    val schemaFields = schema.fields

    /*
    把field作用到rdd上面去
    如何根据schema的field的数据类型以及字段顺序整合到rdd
     */

    val rows = rdd.map(fileContent => {
      //注意fileContent是一段内容，换行分
      val lines = fileContent.split("\n")
      //转换成seq，list
      val data = lines.map(x => x.split(",").map(x=>x.trim).toList)

      //按照字段field整合到rdd
      //按照index压在一起
      /**
        * 10000 long
        * ruoze string
        */
      val typedValues = data.map(x => x.zipWithIndex.map {
        case (value, index) => {
        //数据的index顺序和定义的schema的index顺序一样，拿到列名
          val colName = schemaFields(index).name
//如果是gender列，值0，1要转换
          Utils.castTo(if (colName.equalsIgnoreCase("gender")) {
            if (value == "0") {
              "男"
            } else if (value == "1") {
              "女"
            } else {
              "未知"
            }
          } else {
            value
          }, schemaFields(index).dataType)
        }
      })
//转换成row格式
      typedValues.map(x=>Row.fromSeq(x))
    })
//rdd
     rows.flatMap(x=>x)

  }
```





# SparkStreaming

一个dstream对应一连串时间的RDD，每个时间间隔1个rdd。

1个Dstream绑定1个receiver，receiver运行在executor端。
有receiver的executor，必须至少有两个线程，1个线程运行receiver接受数据，其他的处理数据。
receiver读kafka，增加线程数，不能增大并行，还是串行。只不过

某话题拥有10个分区，如果你的消费者应用程序只配置一个线程对这个话题进行读取，那么这个线程将从10个分区中进行读取。 同上，你配置5个线程，那么每个线程都会从2个分区中进行读取。 同上，你配置10个线程，那么每个线程都会负责1个分区的读取。 同上，你配置多达14个线程。那么这14个线程中的10个将平分10个分区的读取工作，剩下的4个将会被闲置。 因为一个Receiver仅会占用一个core，所以这里设置的线程数并不会实际增加读取消息的吞吐量，只是串行读取。且该线程数也不会影响Stage中的任务并发数。

https://www.jianshu.com/p/c8669261165a



注意点

1.一旦已经启动了一个context，就没有新的计算加进来，就是start后面写业务代码是无效的。
2.一旦一个context被停止了，他就不能被重启。
3.同一时间只有一个streamingcontext是活的，在一个JVM里。



![1553149115960](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1553149115960.png)



## Streaming模板



```
import org.apache.spark._
import org.apache.spark.streaming.{Seconds, StreamingContext}


object StreamingWCApp {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setMaster("local[2]").setAppName("StreamingWCApp")
    //streaming数据多久被切成一批,必须设置
    val ssc = new StreamingContext(conf, Seconds(10))

    val lines=ssc.textFileStream("hdfs://hadoop000:9000/ss/logs")
    val results = lines.flatMap(_.split(",")).map((_,1)).reduceByKey(_+_)
    //打印每批次前10条
    results.print()

    //启动streaming
    ssc.start()
    ssc.awaitTermination()
  }
}
```



## Transform算子



对源DStream的每个RDD应用RDD-to-RDD函数，返回一个DStream。

它可以用于实现，DStream API中所没有提供的操作。比如说，DStream 
API中，并没有提供将一个DStream中的每个batch，与一个特定的RDD进行join的操作。但是我们自己就可以使用transform操作来实现该功能，返回值还是Dstream。

```
lines.map(x=>(x.split(",")(0),x)).transform(rdd=>{
  rdd.leftOuterJoin(blacksRDD).filter(x=>{
    x._2._2.getOrElse(false) !=true
  }).map(_._2._1)
}).print()
```





## UpdateStateByKey算子

此算子可已在当前使用到之前批次的数据（从程序启动开始所有批次）



要想完成截止到目前为止的操作，必须将历史的数据和当前最新的数据累计起来，所以需要一个地方来存放历史数据，这个地方就是checkpoint目录。

```
ssc.checkpoint("hdfs://hadoop000:9000/ss/logs")
```



第一个参数

```

val results=lines.flatMap(_.split(",")).map((_,1)).reduceByKey(_+_)
val state = results.updateStateByKey(updateFunction)
state.print()

def updateFunction(currentValues:Seq[Int],preValues:Option[Int]):Option[Int] = {
    val curr = currentValues.sum
    val pre = preValues.getOrElse(0)

    Some(curr + pre)
  }
```





## ForeachRDD算子

这个算子主要是把Dstream的所有rdd的数据放到外部数据源。

```
import java.sql.DriverManager

import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

results.foreachRDD(rdd =>{
	//注意这行是工作在driver端的，创建连接的代码要写在worker端
  rdd.foreachPartition(partition=>{
  //worker端创建连接
    val connection = createConnection()

    partition.foreach(pair=>{
              val sql=s"insert into wc(word,count) values('${pair._1}',${pair._2})"
              connection.createStatement().execute(sql)
              connection.close()
    })
  })
  
  
def createConnection() ={
    Class.forName("com.mysql.jdbc.Driver")
    DriverManager.getConnection("jdbc:mysql://hadoop000:3306/g5_spark","root","123456")
  }
```



## Windows算子

有状态的算子，也要开启checkpoint

```
ssc.checkpoint("hdfs://hadoop000:9000/ss/logs")
```



Spark Streaming提供了滑动窗口操作的支持，从而让我们可以对一个滑动窗口内的数据执行计算操作，而不是程序启动时（updateStateByKey）。
假设每隔5s 1个batch,上图中窗口长度为15s，窗口滑动间隔10s。窗口长度和滑动间隔必须是batchInterval的整数倍。如果不是整数倍会检测报错。

![1553139620305](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1553139620305.png)



以reduceByKeyAndWindow为例，

第一个参数是要实现啥功能，

第二个参数是windows的长度

第三个参数是窗口滑动时间间隔

```
lines.flatMap(_.split(",")).map((_,1)).reduceByKeyAndWindow((a:Int,b:Int)=>(a+b),Seconds(10),Seconds(5)).print()
```



## Receiver读kafka



createStream4个参数

第一个ssc

第二个元数据存在zookeeper的地址

第三个这个消费者的组id

第四个topic和对应线程数的map。

返回（Kafka message key, Kafka message value)

```
val topic ="ruozeg5sparkkafka"
//注意kafka分区和partition分区无关
val numPartitions=1

val zkQuorum="192.168.137.190:2181,192.168.137.190:2182,192.168.137.190:2183/kafka"
var groupId="test_g5"
//元祖转map
val topics =topic.split(",").map((_,numPartitions)).toMap

//返回个receiver，带有kafka keyvalue的DStream
val messages = KafkaUtils.createStream(ssc,zkQuorum,groupId,topics)

//第一列是null
messages.map(_._2)
  .flatMap(_.split(",")).map((_,1)).reduceByKey(_+_).print()
```





## Direct方式Kafka



createDirectStream3个参数

第一个参数ssc

第二个参数kafka的broker地址（非zookeeper）

第三个参数topic的set



【kafka key的类型，kafka value的类型，kafka key值解析器，kafka value解析器】

```
val topic = "ruozeg5sparkkafka"
val kafkaParams = Map[String, String]("metadata.broker.list"->"hadoop000:9092","metadata.broker.list"->"hadoop000:9093","metadata.broker.list"->"hadoop000:9094")
val topics = topic.split(",").toSet

//[]里是[key class], [value class], [key decoder（解码） class], [value decoder class] ]
//(streamingContext, [map of Kafka parameters], [set of topics to consume])
val messages = KafkaUtils.createDirectStream[String,String,StringDecoder,StringDecoder](ssc,kafkaParams, topics)

messages.map(_._2) // 取出value
  .flatMap(_.split(",")).map((_,1)).reduceByKey(_+_)
  .print()
```



由于direct的方式没有用zookeeper直接管理offset，所以需要手动设置管理。



### checkpoint存偏移量（有缺陷）

缺陷是，修改程序后，重运行，可能会



```
import kafka.serializer.StringDecoder
import org.apache.spark.SparkConf
import org.apache.spark.streaming.kafka.KafkaUtils
import org.apache.spark.streaming.{Duration, Seconds, StreamingContext}

object CheckpointOffsetApp {

  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setMaster("local[2]").setAppName("CheckpointOffsetApp")

    //没有会自动创建
    val checkpointPath = "hdfs://hadoop000:9000/offset_g5/checkpoint"
    val topic="ruozeg5sparkkafka"
    val interval =10
    val kafkaParams = Map[String, String]("metadata.broker.list"->"hadoop000:9092","metadata.broker.list"->"hadoop000:9093","metadata.broker.list"->"hadoop000:9094","auto.offset.reset"->"smallest")
    val topics = topic.split(",").toSet


    def function2CreateStreamingContext()={
      val ssc = new StreamingContext(conf,Seconds(10))
      //[]里是[key class], [value class], [key decoder（解码） class], [value decoder class] ]
      //(streamingContext, [map of Kafka parameters], [set of topics to consume])
      val messages = KafkaUtils.createDirectStream[String,String,StringDecoder,StringDecoder](ssc,kafkaParams, topics)
      
      ssc.checkpoint(checkpointPath)
      messages.checkpoint(Duration(8*10.toInt*1000))

      messages.foreachRDD(rdd=>{
        if (!rdd.isEmpty()){
          println("------asd------"+rdd.count())
        }
      })
      ssc
    }

    //如果检查点数据存在就根据检查点数据重建context，如果不存在就根据第二个参数构建context
    val ssc =StreamingContext.getOrCreate(checkpointPath,function2CreateStreamingContext)
    ssc.start()
    ssc.awaitTermination()


  }
```





## 用mysql管理偏移量

1.先在application.conf设置好kafka和mysql的连接参数

![1553167052906](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1553167052906.png)

```
db.default.driver="com.mysql.jdbc.Driver"
db.default.url="jdbc:mysql://hadoop000/g5offset?characterEncoding=utf-8"
db.default.user="root"
db.default.password="root"
metadata.broker.list="hadoop000:9092,hadoop000:9093,hadoop000:9094"
auto.offset.reset="smallest"
kafka.topics="ruozeg5sparkkafka"
group.id="test_g5"
```



2.然后写个工具类，用来获取application的参数

```
import com.typesafe.config.ConfigFactory



//用来读取application.conf的其他参数
object ValueUtils {
  //加载application.conf
  val load=ConfigFactory.load()

  def getStringValue(key:String,defaultValue:String="")={
    load.getString(key)
  }

  def main(args: Array[String]): Unit = {
    println(getStringValue("kafka.topics"))
  }

}
```



3.写代码咯

```
import kafka.common.TopicAndPartition
import kafka.message.MessageAndMetadata
import kafka.serializer.StringDecoder
import org.apache.spark.SparkConf
import org.apache.spark.streaming.kafka.{HasOffsetRanges, KafkaUtils}
import org.apache.spark.streaming.{Seconds, StreamingContext}
import scalikejdbc._
import scalikejdbc.config.DBs

object MySQLOffsetApp {

  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setMaster("local[2]").setAppName("MysqlOffsetApp")

    //从application.conf加载配置信息
    DBs.setup()

    //用来构建stream的参数，详情看源码
    val fromOffsets= DB.readOnly(implicit session =>{
      sql"select * from offsets_storage".map(rs => {
        (TopicAndPartition(rs.string("topic"), rs.int("partitions")), rs.long("offset"))
      }).list().apply()
    }).toMap


    val topic = ValueUtils.getStringValue("kafka.topics")
    val interval=10

//kafka参数就用工具类取咯
    val kafkaParams=Map(
      "metadata.broker.list"->ValueUtils.getStringValue("metadata.broker.list"),
      "auto.offset.reset"->ValueUtils.getStringValue("auto.offset.reset"),
      "group.id"->ValueUtils.getStringValue("group.id")
    )
    val topics =topic.split(",").toSet
    val ssc=new StreamingContext(conf,Seconds(10))



    ////TODO... 去MySQL里面获取到topic对应的partition的offset

    //没有offset从头消费
    val messages=if(fromOffsets.size==0){
      KafkaUtils.createDirectStream(ssc,kafkaParams,topics)
      //有offset从指定位置消费
    }else{
      //把数据和元数据转换成需要的类型，模板
      val messageHandler = (mm:MessageAndMetadata[String,String]) => (mm.key(),mm.message())
      KafkaUtils.createDirectStream[String,String,StringDecoder,StringDecoder,(String,String)](ssc,kafkaParams,fromOffsets,messageHandler)
    }

    messages.foreachRDD(rdd=>{
      if(!rdd.isEmpty()){
        println("---the count---"+rdd.count())

//转成参数
 @param topic Kafka topic name
 * @param partition Kafka partition id
 * @param fromOffset Inclusive starting offset
 * @param untilOffset Exclusive ending offset
 
        val offsetRanges=rdd.asInstanceOf[HasOffsetRanges].offsetRanges
        offsetRanges.foreach(x=>{
          println(s"--${x.topic}---${x.partition}---${x.fromOffset}---${x.untilOffset}---")

          //替换到mysql最新的offset
          DB.autoCommit(implicit session =>{
            sql"replace into offsets_storage(topic,groupid,partitions,offset) values(?,?,?,?)"
              .bind(x.topic,ValueUtils.getStringValue("group.id"),x.partition,x.untilOffset)
              .update().apply()
          })
        })
      }
    })

    ssc.start()
    ssc.awaitTermination()
  }

}
```





## 模拟生产者发数据



```
import kafka.javaapi.producer.Producer;
import kafka.producer.KeyedMessage;
import kafka.producer.ProducerConfig;

import java.util.Properties;
import java.util.UUID;

//kafka数据发送，producer
public class RuozeKafkaProducer {

    public static void main(String[] args) {

        Properties properties = new Properties();
        properties.put("serializer.class","kafka.serializer.StringEncoder");
        properties.put("metadata.broker.list","hadoop000:9092,hadoop000:9093,hadoop000:9094");

        ProducerConfig producerConfig = new ProducerConfig(properties);
        Producer<String,String> producer = new Producer<String, String>(producerConfig);

        String topic ="ruozeg5sparkkafka";
        for (int index=0; index<100;index++){
            producer.send(new KeyedMessage<String, String>(topic,index+"",index+"ruozeshuju:"+ UUID.randomUUID()));
        }

        System.out.println("若泽数据Kafka生产者生产数据完毕...");


    }

}
```





## curator操作zookeeper

https://blog.51cto.com/zero01/2109137



导包

```
import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.retry.ExponentialBackoffRetry;
```



### 建立客户端，参数

客户端

```
CuratorFramework client = null;
```

zookeeper地址参数

```
String zk ="192.168.137.190:2181,192.168.137.190:2182,192.168.137.190:2183/kafka"
```



### 连接zookeeper



```
public void setUp(){
//自动重连策略
    RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 5);
    //namespace就是zookeeper的一个目录，前面会自动拼个/，即/g5
    client = CuratorFrameworkFactory.builder().connectString(zk)
            .sessionTimeoutMs(20000).retryPolicy(retryPolicy)
            .namespace("g5").build();
    client.start();
}

//关闭连接
public void closeZKClient() {
        if (client != null) {
            this.client.close();
        }
    }
```

获取客户端连接状态

```java
 // 获取当前客户端的状态
 isZkCuratorStarted = client.isStarted();
```



### 创建节点

throws Exception抛除异常不管它

```
public void testCreateNode() throws Exception{
    //创建节点，没有父节点就递归创建
    //持久化节点，瞬时节点session过了就看不到了
    //节点的acl权限
    //最后的结果于在g5/huahua 的数据有 你爱花花吗?
    client.create().creatingParentsIfNeeded()
            .withMode(CreateMode.PERSISTENT)
            .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
            .forPath("/huahua","你爱花花吗?".getBytes());
}
```



### 更新节点数据



```
//指定数据版本
    Stat resultStat = client.setData().withVersion(0).forPath("/huahua","你爱花花吗?1111".getBytes());

//查看数据版本
	resultStat.getVersion()

```



### 删除节点



```
client.delete()
//如果删除失败，后端还是会继续删除，知道成功
.guaranteed()
//递归删除
.deletingChildrenIfNeeded()
.withVersion(stat.getVersion())
.forPath("/g5");
```



### 读取节点数据



```
String asd = new String(client.getData().forPath("/huahua"));
System.out.println(asd);
```



### 获取子节点列表



```
List<String> dsa = client.getChildren().forPath("/");
System.out.println(dsa);
```



### 查询某节点是否存在



```java
Stat statExist = client.checkExists().forPath(nodePath);
        if (statExist == null) {
            System.out.println(nodePath + " 节点不存在");
        } else {
            System.out.println(nodePath + " 节点存在");
        }
```

