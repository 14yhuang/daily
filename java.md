# java是啥

java是解释型语言

![1554171743678](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1554171743678.png)



# instanceof，typeof

typeof是判断变量是什么基本类型的；

instanceof是判断对象到底是什么类型的；



# 变量的3种类型

-  类变量：独立于方法之外的变量，用 static 修饰。
-  实例变量：独立于方法之外的变量，不过没有 static 修饰。  
- 局部变量：类的方法中的变量。

```
public class Variable{

static int allClicks=0;    // 类变量

String str="hello world";  // 实例变量
public void method(){

int i =0;  // 局部变量
}
}
```



局部变量在方法、构造方法、或者语句块被执行的时候创建，当它们执行完成后，变量将会被销毁；

实例变量在对象创建的时候创建，在对象被销毁的时候销毁

无论一个类创建了多少个对象，类只拥有类变量的一份拷贝。



# 修饰符



- **default** (即缺省，什么也不写）: 在**同一包内可见**，不使用任何修饰符。使用对象：类、接口、变量、方法。
- **private** : 在同一类内可见。使用对象：变量、方法。 **注意：不能修饰类（外部类，包外）**
- **public** : 对所有类可见。使用对象：类、接口、变量、方法
- **protected** : 对同一包内的类和所有子类可见。使用对象：变量、方法。 **注意：不能修饰类（外部类）**。



![1554172718292](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1554172718292.png)



## 访问修饰符的继承

- 父类中声明为 public 的方法在子类中也必须为 public。
- 父类中声明为 protected 的方法在子类中要么声明为 protected，要么声明为 public，不能声明为 private。
-  父类中声明为 private 的方法，不能够被继承。





## 非访问修饰符



### static 修饰符

用来修饰类方法和类变量。

- **静态变量：**

   static 关键字用来声明独立于对象的静态变量，无论一个类实例化多少对象，它的静态变量只有一份拷贝。 静态变量也被称为类变量。**局部变量不能被声明为 static 变量**。 

- **静态方法：**

   static 关键字用来声明独立于对象的静态方法。**静态方法不能使用类的非静态变量**。

  

### final 修饰符

用来修饰类、方法和变量，

**final 修饰的类不能够被继承**

**修饰的方法不能被继承类重新定义**

**修饰的变量为常量，是不可修改的。**



### abstract 修饰符

用来创建抽象类和抽象方法。



一个类不能同时被 abstract 和 final 修饰。**如果一个类包含抽象方法，那么该类一定要声明为抽象类**

**抽象方法不能被声明成 final 和 static。**





### synchronized  修饰符

synchronized 关键字声明的方法同一时间只能被一个线程访问



### transient 修饰符

序列化的对象包含被 transient 修饰的实例变量时，java 虚拟机(JVM)跳过该特定的变量。 

**不会被持久化**



### volatile 修饰符

volatile 修饰的成员变量在每次被线程访问时，都强制从共享内存中重新读取该成员变量的值。而且，当成员变量发生变化时，会强制线程将变化值回写到共享内存。**这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值。**

一个 volatile 对象引用可能是 null。 



# 运算符



## ？：条件运算符

真如何，假如何

```
variable x = (expression) ? value if true : value if false
```



## instanceof 运算符

类型判断

```
boolean result = name instanceof String; // 由于 name 是 String 类型，所以返回真
```





# string类，stringBuffer



string要修改会重新创建对象。

和 String 类不同的是，StringBuffer 和 StringBuilder 类的对象能够被多次的修改，并且不产生新的未使用对象



StringBuilder有速度优势

StringBuffer 有线程安全



# 数组

类型加 [] 为数组

```
double[] myList = {1.9, 2.9, 3.4, 3.5};
```

![1554176219944](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1554176219944.png)



# 正则引用



```
import java.util.regex.*;
//上面的引用等于下面的
import java.util.regex.Matcher;
import java.util.regex.Pattern;

class RegexExample1{
public static void main(String args[]){
String content = "I am noob " +
"from runoob.com.";

String pattern = ".*runoob.*";
boolean isMatch = Pattern.matches(pattern, content);

System.out.println("字符串中是否包含了 'runoob' 子字符串? " + isMatch);
}
}
```



# 方法

如果没有返回值，void

![1554180344026](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1554180344026.png)



## 构造方法

构造方法和它所在类的名字相同，但构造方法没有返回值。

Java自动提供了一个默认构造方法，默认构造方法的访问修改符和类的访问修改符相同(类为 public，构造函数也为 public；类改为 private，构造函数也改为 private)。

一旦你定义了自己的构造方法，默认构造方法就会失效。



## 可变参数

在方法声明中，在指定参数类型后加一个省略号(...) 。 

一个方法中只能指定一个可变参数，它必须是方法的最后一个参数。任何普通的参数必须在它之前声明。 

```
public static void printMax( double... numbers) {
```



## finalize() 方法

它在对象被垃圾收集器析构(回收)之前调用，这个方法叫做 finalize( )，它用来清除回收对象。

finalize() 一般格式是：

```
protected void finalize() {    // 在这里终结代码 }
```

 关键字 protected 是一个限定符，它确保 finalize() 方法不会被该类以外的代码调用。 

 当然，Java 的内存回收可以由 JVM 来自动完成。如果你手动使用，则可以使用上面的方法。  



# IO

```
import java.io.*;
public class FileRead {
public static void main(String args[]) throws IOException {
File file = new File("Hello1.txt");
// 创建文件
file.createNewFile();
// creates a FileWriter Object
FileWriter writer = new FileWriter(file);
// 向文件写入内容
writer.write("This\n is\n an\n example\n");
writer.flush();
writer.close();
// 创建 FileReader 对象
FileReader fr = new FileReader(file);
char[] a = new char[50];
fr.read(a); // 从数组中读取内容
for (char c : a)
System.out.print(c); // 一个个打印字符
fr.close();
}
}
```



# 异常



```
//throws作用是提醒开发者代码可能出现的错误
public void deposit(double amount) throws RemoteException
  {
    // Method implementation
    throw new RemoteException();
  }
  //Remainder of class definition
}
```



try catch finally

无论是否发生异常，finally 代码块中的代码总会被执行。



# 继承

java接口可以多继承

![1554191606910](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1554191606910.png)



## implements关键字

使用 implements 关键字可以变相的使java具有多继承的特性，使用范围为类继承接口的情况，**可以同时继承多个接口**（接口跟接口之间采用逗号分隔）。

```

public interface A {
    public void eat();
    public void sleep();
}
 
public interface B {
    public void show();
}
 
public class C implements A,B {
}

```



## super和this

super关键字：我们可以通过super关键字来实现对父类成员的访问，用来引用当前对象的父类。

this关键字：指向自己的引用，或构造方法。

```
class Animal {
  void eat() {
    System.out.println("animal : eat");
  }
}
 
class Dog extends Animal {
  void eat() {
    System.out.println("dog : eat");
  }
  void eatTest() {
    this.eat();   // this 调用自己的方法
    super.eat();  // super 调用父类方法
  }
}
```



##  final关键字

 final 关键字声明类可以把类定义为不能继承的，即最终类；或者用于修饰方法，该方法不能被子类重写



## 构造器

如果父类的构造器带有参数，则必须在子类的构造器中显式地通过 super 关键字调用父类的构造器并配以适当的参数列表。

如果父类构造器没有参数，则在子类的构造器中不需要使用 super 关键字调用父类构造器，系统会自动调用父类的无参构造器。 



# 重写和重载

重写于继承之中。

重载于同类之中。

![1554192523135](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1554192523135.png)







# 抽象



在Java中抽象类表示的是一种继承关系，一个类只能继承一个抽象类，而一个类却可以实现多个接口。

```
public abstract class Employee
```



# 接口



```
import java.lang.*;
//引入包
 
public interface NameOfInterface extends Sports, Event
{
   //任何类型 final, static 字段
   //抽象方法
}
```





# mysql



```
import java.sql.*;


	// JDBC 驱动名及数据库 URL
    static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";  
    static final String DB_URL = "jdbc:mysql://localhost:3306/RUNOOB";
 
    // 数据库的用户名与密码，需要根据自己的设置
    static final String USER = "root";
    static final String PASS = "123456";
    
    
    Connection conn = null;
    Statement stmt = null;
    
    // 注册 JDBC 驱动
            Class.forName("com.mysql.jdbc.Driver");
            conn = DriverManager.getConnection(DB_URL,USER,PASS);
            stmt = conn.createStatement();
            String sql;
            sql = "SELECT id, name, url FROM websites";
            ResultSet rs = stmt.executeQuery(sql);
            
            // 展开结果集数据库
            while(rs.next()){
                // 通过字段检索
                int id  = rs.getInt("id");
                String name = rs.getString("name");
                String url = rs.getString("url");
            }
            // 完成后关闭
            rs.close();
            stmt.close();
            conn.close();
```



































