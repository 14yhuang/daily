scala 1

## 类型判断与转换

```
10.isInstanceOf[Int]
10.asInstanceOf[Long]
```



## getclass返回一个值的类型

```
println(10.getClass())
```

## classof

getClass 方法得到的是某个东西（已经实例化）的类型

而 classOf[A] 参数是一个类，得到这个类的路径



## 变长参数 *

```
def sum(a:Int*)={
```







## 没有入参的函数调用时可以不加（）



## to和until

1.to（10）= 1到10的集合

1.until（10）=Range（1，10）=1到9的集合



## 占位符 _

```
var name:String =_
```



## 参数类型自动适配 _*

假如有个变长的函数

```
def sum(args: Int*) = {
```

然后这样使用会报错，因为传入的是个集合

```
val s = sum(1 to 5)
```

但是可以让它类型转换

```
val s = sum(1 to 5: _*)
```





## private和protected

private表示只有该类或对象，伴生类对象能用。

protected表示该类或对象和子类才能用

```
private val gender="male"
```

还有作用域的功能

```
//workDetails还可以被professional类或对象访问
private[professional] var workDetails = null
//this表示只在当前类或对象才可以使用，甚至new对象里都不能用
private[this] var secrets = null
```



## trait接口

一个类可以继承多个接口



## 与或非

&&

||

！



## 抽象与继承重载

```
abstract class Person2{
  def hit
  val name:String
}
```

抽象类new时要实现

```
override def hit: Unit = "asd"
```

继承，第一个父类用extends连接，多父类后面用with连接

```
class Student2 extends Person2{
```



## final

被final修饰的方法或变量不能被重载



## Object静态

多个Object对象共享一个Object参数，一个对象改变此参数，另外的对象参数也会变。

比如一个object有1个参数a，初始值为0.	第一个对象调用使a=1，第二个对象也变成了1.



class就不会这样，不同的new互不影响



## apply

```
//有一个Object MyArray
object MyArray {

//后面加（）相当于执行的是MyArray里的apply方法
var arr3 = MyArray()
```



## 伴生对象和类

伴生就是，object和class的名字一样。伴生对象的意义就是弥补scala里class没有静态修饰符的功能。



伴生类和伴生对象的特点是可以相互访问被private修饰的字段

```
object Applyapp {
  def main(args: Array[String]): Unit = {
    //c就相当于new的ApplyTest
    val c= ApplyTest()
    //class ApplyTest的apply
    c()
  }
}

//伴生对象和类，object可以直接调用，class要new
class ApplyTest{
  def test()={
    println("class Applytest enter")
  }
  def apply()={
    println("class apply")
    println(ApplyTest.inc)
  }
}

object ApplyTest{
  var count=0
  private def inc={
    count=count+1
    count
  }

  def apply()={
    println("object apply")
    new ApplyTest
  }
}
```





## 单例对象

是一个通过使用object关键字而不是使用class关键字声明的对象



## String，StringBuilder，StringBuffer

string是常量，执行string之间的操作比如连接，会重新创建一个string。

StringBuffer 和 StringBuilder是字符串变量

StringBuffer 线程安全（适合多线程），StringBuilder线程不安全（适合单线程）

```
val fl = new StringBuilder()
fl.append(generateurl()).append("\t").append(generatetime)
  .append("\t").append(generateliuliang2(30))
```



## 字符串函数

字符串截取

substring（起始索引，结束索引）



去除头尾空格

trim



得到某字符在字符串中的索引，第二个参数是从哪个索引位置开始搜索

```
indexOf("-",7)
```



## 定长数组

```
val a = new Array[String](5)
```

修改，1是索引，从0开始

```
a(1)="ruoze"
```



## 变长数组

添加和修改和定长数组不太一样

```
import scala.collection.mutable.ArrayBuffer
val c = ArrayBuffer[Int]()
```

添加

```
c += 1
c++= Array(6,7,8)
```

指定位置添加

```
//第0个位置添加为1
c.insert(0,1)
```

删除

```
//把索引为1的移走
c.remove(1)
//从第几个位置移几个
c.remove(1,3)
//删掉最后两个
c.trimEnd(2)
```

把可变数组变为不可变

```
c.toArray
```





## 数组函数

### mkstring

```
val a = Array("apple", "banana", "cherry")
```

```
a.mkString("[", ", ", "]")
res6: String = [apple, banana, cherry]
```



# Seq序列

相当于在Array和List之上的接口

![1552824445732](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1552824445732.png)

# 集合

Scala 集合分为可变的和不可变的集合。

可变集合这意味着你可以修改，添加，移除一个集合的元素。

而不可变集合类，相比之下，永远不会改变。不过，你仍然可以模拟添加，移除或更新操作。但是这些操作将在每一种情况下都返回一个新的集合，同时使原来的集合不发生改变。

## List

List的特征是其元素以线性方式存储，集合中可以存放重复对象。



造个空List，不可变集合

```
var a = Nil
//正常构建
val l =List(1,2,3,4,5)
```

函数

```
l.head
//注意tail是除了第一个元素外的其他元素，如果只一个元素，返回空列表
l.tail
```

拼接 ：：

```
//1拼接一个空的集合就是只有1的集合
val l2 = 1::Nil
val l3= 2::l2
```



## 可变ListBuffer

```
import scala.collection.mutable.ListBuffer
val l4 =ListBuffer[Int]()
```

添加

```
l4 ++= List(1,2,3)
```

转换

```
l4.toArray
//转换为不可变
l4.toList
```

判断为空吗

```
l4.isEmpty
```



## Set

集合中的对象不按特定的方式排序，并且没有重复对象。

```
val x = Set(1,3,5,7)
```



## Map

key-value形式



创建

```
val a =Map("name"->"ruoze","age"->30)
//查看
println(a("name"),a("age"))
```



## 可变Map

```
import scala.collection.mutable.HashMap
val b =HashMap("name"->"ruoze","age"->30)
b("age")=19
```

循环

```
for ((key,value)<- b){
  println(key + ":" + value)
}
```

得到key的去重集合

```
b.keySet
```



## 元祖Tuple

元组是不同类型的值的集合，可以重复

```
val x = (10, "Runoob")
//取第一个值
println(a._1)
```



## Option

Option[T] 表示有可能包含值的容器，也可能不包含值。

如果值存在， Option[T] 就是一个 Some[T] ，如果不存在， Option[T] 就是对象 None 。

```
val m =Map(1->2)
```


```
//返回Some(2)
println(m.get(1))
//返回2
println(m.get(1).get)
//如果用get，原先是None，会报错
println(m.get(2).getOrElse(0))
```



## 迭代器Iterator

Scala Iterator（迭代器）不是一个集合，它是一种用于访问集合的方法。

迭代器 it 的两个基本操作是 **next** 和 **hasNext**。

调用 **it.next()** 会返回迭代器的下一个元素，并且更新迭代器的状态。

调用 **it.hasNext()** 用于检测集合中是否还有元素。

```
val it = Iterator("Baidu", "Google", "Runoob", "Taobao")
      
      while (it.hasNext){
         println(it.next())
```



## 循环

```
for (ele<-c) {
    println(ele)
}
```





## case class

**最重要的意义就是支持模式匹配**

建立一个case class

```
case class Dog(name:String)
```

可以直接使用

```
println(Dog)
```



## 构造器

一个Scala类中，除了参数，方法以外的代码都是主构造的内容

辅助构造器的功能可以让类有多种传入参数的方式，不同的传入可以有不同的功能



辅助构造器，都以def this开始

参数不可以用val或var修饰

```
def this(name:String,age:Int,gender:String){
```

辅助构造器，第一行必须调用主构造器或其他辅助构造器（之前定义的，不能之后的）

```
//第一个辅助构造器只能this主构造器的参数
this(name,age)
```

给辅助构造器参数赋值

```
this.school="stu"
```





## scala算子

filter

### take

取前3个

```
val l = List(1,2,3,4)
l.take(3)
```

### reduce

```
val l = List(1,2,3,4)
//    1-2-3-4
println(l.reduce(_-_))
```

### reduceLeft

```
//    1,2
//    -1,3
//    -4,4
    l.reduceLeft((x,y) => {
      println(x+","+y)
      x-y
    })
```



### reduceRight

```
//    3,4
//    2,-1
//    1,3
    l.reduceRight((x,y) => {
      println(x+","+y)
      x-y
    })
```



### fold

```
//    10,1
//    9,2
//    7,3
//    4,4
println(l.fold(10)(_-_))
```

### count

```
//count里要有条件
println(l.count(_>1))
```

### flatten

```
val f=List(List(1,2),List(3,4))
println(f.flatten)
结果
List(1, 2, 3, 4)
```

### flatMap

```
val f=List(List(1,2),List(3,4))
// List(2, 4, 6, 8)
println(f.flatMap(_.map(_*2)))
```



## 柯里化curry

柯里化(Currying)指的是将原来接受两个参数的函数变成新的接受一个参数的函数的过程。新的函数返回一个以原有第二个参数为参数的函数。降低代码耦合性。

```
def sum(a:Int,b:Int)=a+b
//  curry
def sum(a:Int)(b:Int)=a+b
```



## IO scala，java

```
//注意导这个包
import scala.io.Source
```

读取文件

```
val file = Source.fromFile("f:/asd.txt")
```

输出文件

```
for(line <- file.getLines()){
  println(line)
}
```

写文件，scala用的是java的类

```
import java.io.File
val writer = new PrintWriter(new File("test.txt" ))

writer.println(generatefile())
     //注意不能直接关，如果缓存区的数据没有全部写入，会丢失，这里就是强制写完
    writer.flush()
    //关了才能输出在控制台
    writer.close()
```

从屏幕上读取用户输入

```
val line = StdIn.readLine()

println("谢谢，你输入的是: " + line)
```



## scala操作hdfs

```
import java.net.URI

import org.apache.hadoop.conf.Configuration
import org.apache.hadoop.fs.{FSDataInputStream, FileSystem, Path}
import org.apache.hadoop.io.IOUtils


object HDFSApp1 {
  def main(args: Array[String]): Unit = {

//连接到hdfs
    val configuration = new Configuration
    val uri = new URI("hdfs://192.168.137.190:9000" )
    //第三个是用户名
    val fileSystem = FileSystem.get(uri,configuration,"hadoop")

//hdfs文件地址
    val in:FSDataInputStream = fileSystem.open(new Path("/g5/asd.txt"))
//把文件读到控制台，第一个参数文件路径，第二个输出到哪，第三个读多少数据输出一次
    IOUtils.copyBytes(in,System.out,1024)

//删除文件，true表示递归
    val flag = fileSystem.delete(new Path("/g5/test"),true)
    println(flag)
  }
}
```



# scala JdBC

1.创建连接

​	在resources文件夹下创建个application.conf文件，在里面设置连接参数

![1552870584960](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1552870584960.png)

​	然后在代码里建立连接

```
DBs.setup()
```

```
//注意这里也是设置数据库的配置，和上面DBs方式2选一
val driverClass = "com.mysql.jdbc.Driver"
val jdbcUrl = "jdbc:mysql://192.168.137.190:3306/ruozedb";
val username = "ruoze";
val pwd = "123456";
Class.forName(driverClass)
ConnectionPool.singleton(jdbcUrl, username, pwd)
```

导包

```
import scalikejdbc._
```



## 读取操作

有两种写法，带case class和不带的。

不带case class

```
val third = DB.readOnly(implicit session => {
  sql"select deptno from dept".map(rs=>(rs.string("deptno"))).list.apply()
})
```

结果

```
List((10,NEW YORK), (20,DALLAS), (30,CHICAGO), (40,BOSTON))
```



带case class，有个不好的地方，case class每个参数都要传

```
val first = DB.readOnly(implicit session => {
  sql"select * from dept".map(rs=>dept(rs.string("deptno"),rs.string("dname"))).list.apply()
})

case class dept(no:String,name:String)
```

结果

```
List(dept(10,ACCOUNTING), dept(20,RESEARCH), dept(30,SALES), dept(40,OPERATIONS))
```



## 插入操作

```
def insertsql(deptno:String,dname:String,loc:String) = {
  val insertdata = DB.autoCommit(implicit session => {
    sql"insert into dept values(${deptno},${dname},${loc})".update.apply()
  })
  val insertdata2 = DB.autoCommit(implicit session => {
    sql"insert into dept values(?,?,?)".bind(2,"asd","asd").update.apply()
  })
}
```



## 删除操作

```
val deletedata = DB.autoCommit(implicit session => {
  sql"""
       delete from dept where deptno=(?) or deptno=(?)
     """.bind(1,2).update.apply()
})
```



## 字符串插值

```
val name  ="asd"
//下面两个println一样
println("hello:"+name)
println(s"hello:$name")

//字符串插值
//""""""输出多行字符串
val b =
  s"""
    |huanying asd
    |fuck you
    |$name asd
  """.stripMargin

println(b)
```



## 隐式转换implicit



### 类型增强

**低级函数2高级函数（x：低级函数）：高级函数=new 高级函数（高级函数的参数给值）**

```

implicit def man2superman(man: Man):Superman=new Superman(man.name,man.name)
val man =new Man("xxx")
//如果原函数有fly方法，优先使用原函数的
man.fly()


class Man(var name:String){
//  def fly()={
//    println("asd")
//  }
}

class Superman(var name: String,var name2:String){
  def fly()={
    println("niubi")
  }
}
```



### 隐式参数

可以不给隐式参数赋值

```
object Context_Implicits {
    implicit val default: String = "Java"
}

object Param {
    //函数中用implicit关键字 定义隐式参数
    def print(context: String)(implicit language: String){
        println(language+":"+context)
    }
}

object Implicit_Parameters {
    def main(args: Array[String]): Unit = {
        //隐式参数正常是可以传值的，和普通函数传值一样  但是也可以不传值，因为有缺省值(默认配置)
        Param.print("Spark")("Scala") 
        
        import Context_Implicits._
        //隐式参数没有传值，编译器会在全局范围内搜索 有没有implicit String类型的隐式值 并传入
        Param.print("Hadoop")          //Java:Hadoop
    }
}
```



### 类型匹配

```
implicit def int2Range(num : Int): Range = 1 to num

def spreadNum(range: Range): String =range.mkString(",")

//spreadNum应当传入一个range，这里传入了个Int，会搜索是否有Int转换为range的隐式函数
spreadNum(5)
```



## 随机数

```
import scala.util.Random
Random.nextInt(100)
```



## 模式匹配



```
teacher match {
  case "qwe"=> println("ewq")
  case "asd"=> println("dsa")
  case "zxc"=> println("cxz")
}
```



## 偏函数

匹配输入参数

```
def saysomething(name:String)= name match {
  case "qwe"=> println("ewq")
  case "asd"=> println("dsa")
  case "zxc"=> println("cxz")
}
```



## try catch finally

主要是用来处理异常的，出现catch住的异常，程序不会异常终止

```
try{
      val i=1/0
      println(i)
    }catch {
  //Ar毕竟是Exception的子类，但是如果写的更前，报Ar
      case e:Exception => println("aaaa")
      case e:ArithmeticException =>println("cuola")
    }finally {
      println("这个finall是一定要执行的")
    }
```



## throw抛出自定义异常

```
throw new ArithmeticException("You are not eligible")
```



## throws

throw（语句直接抛出的一个异常）

throws（声明方法时，该方法可能抛出的异常，感觉和注释差不多）

```
class ExceptionExample4{  
@throws(classOf[NumberFormatException])  
def validate()={  
"abc".toInt  
}  
} 
```



## scala正则表达式

```
import scala.util.matching.Regex

val pattern = new Regex("abl[ae]\\d+")
val str = "ablaw is able1 and cool"
println((pattern findAllIn str).mkString(","))
```





# scala泛型

**泛型方法**：指定方法可以接受任意类型参数。
**泛型类**：指定类可以接受任意类型参数。







## scala Wordcount

纯scala里没有bykey

```
import scala.io.Source

object tt{
  def main(args: Array[String]): Unit = {
    val file=Source.fromFile("F:\\asd.txt")

//getLines之后是迭代器，需要转换，不然后面不能用groupby
    var first = file.getLines().toList.flatMap(_.split(",")).map((_,1))
    //(l1,List((l1,1), (l1,1), (l1,1))) groupby后
    var second = first.groupBy(_._1).mapValues(_.size)
    second.foreach(println)

  }
}
```

