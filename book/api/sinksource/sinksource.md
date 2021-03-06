#一、flink在批处理中常见的source
```
flink在批处理中常见的source主要有两大类。
1.基于本地集合的source（Collection-based-source）
2.基于文件的source（File-based-source）
```

##1.基于本地集合的source
```
在flink最常见的创建DataSet方式有三种。
1.使用env.fromElements()，这种方式也支持Tuple，自定义对象等复合形式。
2.使用env.fromCollection(),这种方式支持多种Collection的具体类型
3.使用env.generateSequence()方法创建基于Sequence的DataSet
```
###执行程序
```scala
package code.book.batch.sinksource.scala

import org.apache.flink.api.scala.{DataSet, ExecutionEnvironment, _}
import scala.collection.immutable.{Queue, Stack}
import scala.collection.mutable
import scala.collection.mutable.{ArrayBuffer, ListBuffer}

object DataSource001 {
  def main(args: Array[String]): Unit = {
    val env = ExecutionEnvironment.getExecutionEnvironment
    //0.用element创建DataSet(fromElements)
    val ds0: DataSet[String] = env.fromElements("spark", "flink")
    ds0.print()

    //1.用Tuple创建DataSet(fromElements)
    val ds1: DataSet[(Int, String)] = env.fromElements((1, "spark"), (2, "flink"))
    ds1.print()

    //2.用Array创建DataSet
    val ds2: DataSet[String] = env.fromCollection(Array("spark", "flink"))
    ds2.print()

    //3.用ArrayBuffer创建DataSet
    val ds3: DataSet[String] = env.fromCollection(ArrayBuffer("spark", "flink"))
    ds3.print()

    //4.用List创建DataSet
    val ds4: DataSet[String] = env.fromCollection(List("spark", "flink"))
    ds4.print()

    //5.用List创建DataSet
    val ds5: DataSet[String] = env.fromCollection(ListBuffer("spark", "flink"))
    ds5.print()

    //6.用Vector创建DataSet
    val ds6: DataSet[String] = env.fromCollection(Vector("spark", "flink"))
    ds6.print()

    //7.用Queue创建DataSet
    val ds7: DataSet[String] = env.fromCollection(Queue("spark", "flink"))
    ds7.print()

    //8.用Stack创建DataSet
    val ds8: DataSet[String] = env.fromCollection(Stack("spark", "flink"))
    ds8.print()

    //9.用Stream创建DataSet（Stream相当于lazy List，避免在中间过程中生成不必要的集合）
    val ds9: DataSet[String] = env.fromCollection(Stream("spark", "flink"))
    ds9.print()

    //10.用Seq创建DataSet
    val ds10: DataSet[String] = env.fromCollection(Seq("spark", "flink"))
    ds10.print()

    //11.用Set创建DataSet
    val ds11: DataSet[String] = env.fromCollection(Set("spark", "flink"))
    ds11.print()

    //12.用Iterable创建DataSet
    val ds12: DataSet[String] = env.fromCollection(Iterable("spark", "flink"))
    ds12.print()

    //13.用ArraySeq创建DataSet
    val ds13: DataSet[String] = env.fromCollection(mutable.ArraySeq("spark", "flink"))
    ds13.print()

    //14.用ArrayStack创建DataSet
    val ds14: DataSet[String] = env.fromCollection(mutable.ArrayStack("spark", "flink"))
    ds14.print()

    //15.用Map创建DataSet
    val ds15: DataSet[(Int, String)] = env.fromCollection(Map(1 -> "spark", 2 -> "flink"))
    ds15.print()

    //16.用Range创建DataSet
    val ds16: DataSet[Int] = env.fromCollection(Range(1, 9))
    ds16.print()

    //17.用fromElements创建DataSet
    val ds17: DataSet[Long] =  env.generateSequence(1,9)
    ds17.print()
  }
}

```
##2.基于文件的source（File-based-source）
```
flink支持多种存储设备上的文件，包括本地文件，hdfs文件，alluxio文件等。
flink支持多种文件的存储格式，包括text文件，CSV文件等。
```
###执行程序
```scala
package code.book.batch.sinksource.scala

import org.apache.flink.api.scala.{DataSet, ExecutionEnvironment,_}

object DataSource002 {
  def main(args: Array[String]): Unit = {

    val env = ExecutionEnvironment.getExecutionEnvironment
    //1.读取本地文本文件,本地文件以file://开头
    val ds1: DataSet[String] = env.readTextFile("file:///Applications/flink-1.1.3/README.txt")
    ds1.print()

    //2.读取hdfs文本文件，hdfs文件以hdfs://开头,不指定master的短URL
    val ds2: DataSet[String] = env.readTextFile("hdfs:///input/flink/README.txt")
    ds2.print()

    //3.读取hdfs CSV文件,转化为tuple
    val path = "hdfs://qingcheng11:9000/input/flink/sales.csv"
    val ds3 = env.readCsvFile[(String, Int, Int, Double)](
      filePath = path,
      lineDelimiter = "\n",
      fieldDelimiter = ",",
      lenient = false,
      ignoreFirstLine = true,
      includedFields = Array(0, 1, 2, 3))
    ds3.print()

    //4.读取hdfs CSV文件,转化为case class
    case class Sales(transactionId: String, customerId: Int, itemId: Int, amountPaid: Double)
    val ds4 = env.readCsvFile[Sales](
      filePath = path,
      lineDelimiter = "\n",
      fieldDelimiter = ",",
      lenient = false,
      ignoreFirstLine = true,
      includedFields = Array(0, 1, 2, 3),
      pojoFields = Array("transactionId", "customerId", "itemId", "amountPaid")
    )
    ds4.print()
  }
}
```

##3.基于文件的source（遍历目录）
```
flink支持对一个文件目录内的所有文件，包括所有子目录中的所有文件的遍历访问方式。
```
###执行程序
```scala
package code.book.batch.sinksource.scala

import org.apache.flink.api.scala.ExecutionEnvironment
import org.apache.flink.configuration.Configuration

/**
  * 递归读取hdfs目录中的所有文件，会遍历各级子目录
  */
object DataSource003 {
  def main(args: Array[String]): Unit = {
    val env = ExecutionEnvironment.getExecutionEnvironment
    // create a configuration object
    val parameters = new Configuration
    // set the recursive enumeration parameter
    parameters.setBoolean("recursive.file.enumeration", true)
    // pass the configuration to the data source
    val ds1 = env.readTextFile("hdfs:///input/flink").withParameters(parameters)
    ds1.print()
  }
}
```
#二、flink在批处理中常见的sink
```
1.基于本地集合的sink（Collection-based-sink）
2.基于文件的sink（File-based-sink）
```
##1.基于本地集合的sink（Collection-based-sink）
```
package code.book.batch.sinksource.scala
import org.apache.flink.api.scala.{DataSet, ExecutionEnvironment, _}
object DataSink000 {
  def main(args: Array[String]): Unit = {
    //1.定义环境
    val env = ExecutionEnvironment.getExecutionEnvironment
    //2.定义数据 stu(age,name,height)
    val stu: DataSet[(Int, String, Double)] = env.fromElements(
      (19, "zhangsan", 178.8),
      (17, "lisi", 168.8),
      (18, "wangwu", 184.8),
      (21, "zhaoliu", 164.8)
    )
    //3.sink到标准输出
    stu.print

    //3.sink到标准error输出
    stu.printToErr()

    //4.sink到本地Collection
    print(stu.collect())
  }
}
```
##2.基于文件的sink（File-based-sink）
```
flink支持多种存储设备上的文件，包括本地文件，hdfs文件，alluxio文件等。
flink支持多种文件的存储格式，包括text文件，CSV文件等。
```

```scala
package code.book.batch.sinksource.scala

import org.apache.flink.api.scala.{DataSet, ExecutionEnvironment, _}
import org.apache.flink.core.fs.FileSystem.WriteMode
object DataSink001 {
  def main(args: Array[String]): Unit = {
    //0.主意：不论是本地还是hdfs.若Parallelism>1将把path当成目录名称，若Parallelism=1将把path当成文件名。
    val env = ExecutionEnvironment.getExecutionEnvironment
    val ds1: DataSet[(Int, String)] = env.fromCollection(Map(1 -> "spark", 2 -> "flink"))
    //1.写入到本地，文本文档,NO_OVERWRITE模式下如果文件已经存在，则报错，OVERWRITE模式下如果文件已经存在，则覆盖
    ds1.setParallelism(1).writeAsText("file:///output/flink/datasink/test001.txt",
    WriteMode.OVERWRITE)
    env.execute()

    //2.写入到hdfs，文本文档,路径不存在则自动创建路径。
    ds1.setParallelism(1).writeAsText("hdfs:///output/flink/datasink/test001.txt",
    WriteMode.OVERWRITE)
    env.execute()

    //3.写入到hdfs，CSV文档
    //3.1读取csv文件
    val inPath = "hdfs:///input/flink/sales.csv"
    case class Sales(transactionId: String, customerId: Int, itemId: Int, amountPaid: Double)
    val ds2 = env.readCsvFile[Sales](
      filePath = inPath,
      lineDelimiter = "\n",
      fieldDelimiter = ",",
      lenient = false,
      ignoreFirstLine = true,
      includedFields = Array(0, 1, 2, 3),
      pojoFields = Array("transactionId", "customerId", "itemId", "amountPaid")
    )
    //3.2将CSV文档写入到hdfs
    val outPath = "hdfs:///output/flink/datasink/sales.csv"
    ds2.setParallelism(1).writeAsCsv(filePath = outPath, rowDelimiter = "\n",
    fieldDelimiter = "|", WriteMode.OVERWRITE)
    env.execute()
  }
}
```
##3.基于文件的sink（数据进行排序）
```
可以使用sortPartition对数据进行排序后再sink到外部系统。
```
###执行程序
```scala
package code.book.batch.sinksource.scala
import org.apache.flink.api.common.operators.Order
import org.apache.flink.api.scala.{DataSet, ExecutionEnvironment, _}
import org.apache.flink.core.fs.FileSystem.WriteMode
object DataSink002 {
  def main(args: Array[String]): Unit = {
    val env = ExecutionEnvironment.getExecutionEnvironment
    //stu(age,name,height)
    val stu: DataSet[(Int, String, Double)] = env.fromElements(
      (19, "zhangsan", 178.8),
      (17, "lisi", 168.8),
      (18, "wangwu", 184.8),
      (21, "zhaoliu", 164.8)
    )
    //1.以age从小到大升序排列(0->9)
    stu.sortPartition(0, Order.ASCENDING).print
    //2.以naem从大到小降序排列(z->a)
    stu.sortPartition(1, Order.DESCENDING).print
    //3.以age升序，height降序排列
    stu.sortPartition(0, Order.ASCENDING).sortPartition(2, Order.DESCENDING).print
    //4.所有字段升序排列
    stu.sortPartition("_", Order.ASCENDING).print
    //5.以Student.name升序
    //5.1准备数据
    case class Student(name: String, age: Int)
    val ds1: DataSet[(Student, Double)] = env.fromElements(
      (Student("zhangsan", 18), 178.5),
      (Student("lisi", 19), 176.5),
      (Student("wangwu", 17), 168.5)
    )
    val ds2 = ds1.sortPartition("_1.age", Order.ASCENDING).setParallelism(1)
    //5.2写入到hdfs,文本文档
    val outPath1="hdfs:///output/flink/datasink/Student001.txt"
    ds2.writeAsText(filePath = outPath1, WriteMode.OVERWRITE)
    env.execute()
    //5.3写入到hdfs,CSV文档
    val outPath2="hdfs:///output/flink/datasink/Student002.csv"
    ds2.writeAsCsv(filePath = outPath2,rowDelimiter = "\n",
    fieldDelimiter = "|||",WriteMode.OVERWRITE)
    env.execute()
  }
}
```
