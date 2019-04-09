# shell是啥

任何代码最终都要被“翻译”成二进制的形式才能在计算机中执行。



编译型语言：c/c++，程序运行之前将所有代码都翻译成二进制形式，也就是生成可执行文件，用户拿到的是最终生成的可执行文件，看不到源码。



解释型语言，脚本：shell，python，需要一边执行一边翻译，不会生成任何可执行文件，用户必须拿到源码才能运行程序。程序运行后会即时翻译，翻译完一部分执行一部分，不用等到所有代码都翻译完。



编译型语言的优点是执行速度快、对硬件要求低、保密性好，适合开发操作系统、大型应用程序、数据库等。
脚本语言的优点是使用灵活、部署容易、跨平台性好，非常适合Web开发以及小工具的制作。



Linux 正则表达式以及三剑客 grep、awk、sed 等命令。



# |管道符

**管道符“|”将两个命令隔开，管道符左边命令的输出就会作为管道符右边命令的输入。连续使用管道意味着第一个命令的输出会作为第二个命令的输入，第二个命令的输出又会作为第三个命令的输入，依此类推。**



# awk

适合格式化文本，对文本进行较复杂格式处理。默认空格或tab分割

他读取文本时是一行一行读进来处理



常用语法形式：awk [options] 'commands' filenames



## -F指定分割

这就是指定，分割

打印第一和第二列

$0 表示所有行

```
awk -F, '{print $1,$2}'   log.txt
```



## -f加载awk脚本

```
将所有的awk命令插入一个单独文件，然后调用：
awk -f awk-script-file input-file(s)
其中，-f选项加载awk-script-file中的awk脚本，input-file(s)跟上面的是一样的。
```

awk脚本开头

```
#!/bin/awk
```



## -v指定外部变量值



```
v var=var_value
```

在awk程序执行前，把awk变量var的值设置为var_value，这个var变量在BEGIN块中也有效，经常用来把shell变量引入awk程序。





## begin，end

begin在读取每行前执行一次，共执行n（行）次



end在读完后执行一次，共执行一次。

```
awk '{count++;print $0;} {print "fck"} END{print "user count is ", count}' /etc/passwd
```



## 内置变量

![1553567649643](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1553567649643.png)

```
awk  -F ':'  '{print "filename:" FILENAME ",linenumber:" NR ",columns:" NF ",linecontent:"$0}' /etc/passwd
```



## print和printf

print函数的参数可以是变量、数值或者字符串。字符串必须用双引号引用，参数用逗号分隔。如果没有逗号，参数就串联在一起而无法区分。这里，逗号的作用与输出文件的分隔符的作用是一样的，只是后者是空格而已。



printf函数，其用法和c语言中printf基本相似,可以格式化字符串,输出复杂时，printf更加好用，代码更易懂。

```
awk  -F ':'  '{printf("filename:%10s,linenumber:%s,columns:%s,linecontent:%s\n",FILENAME,NR,NF,$0)}' /etc/passwd
```



## 正则模式

**~ 表示模式开始。// 中是模式，也是正则。**

```
输出第二列包含 "th"，并打印第二列与第四列
awk '$2 ~ /th/ {print $2,$4}' log.txt
```



```
整行匹配，输出包含"re" 的行
$ awk '/re/ ' log.txt
```



不取匹配到的，模式取反

```
awk '$2 !~ /th/ {print $2,$4}' log.txt

awk '!/th/ {print $2,$4}' log.txt
```



## ；换行

等价于换行

如果代码已分行写，则不用





# sed

Stream Editor文本流编辑，sed是一个“非交互式的”面向字符流的编辑器。能同时处理多个文件多行的内容，可以不对原文件改动，把整个文件输入到屏幕,可以把只匹配到模式的内容输入到屏幕上。还可以对原文件改动，但是不会再屏幕上返回结果。



在sed中正则表达式是写在 /.../ 两个斜杠中间的



```
sed的命令格式： sed [option] 'sed command' filename

sed的脚本格式：sed [option] -f 'sed script' filename
```



![1553573335504](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1553573335504.png)



sed的正则表达



| ^           | 锚点行首的符合条件的内容，用法格式"^pattern" |
| ----------- | -------------------------------------------- |
| $           | 锚点行首的符合条件的内容，用法格式"pattern$" |
| ^$          | 空白行                                       |
| .           | 匹配任意单个字符                             |
| *           | 匹配紧挨在前面的字符任意次(0,1,多次)         |
| .*          | 匹配任意长度的任意字符                       |
| \？         | 匹配紧挨在前面的字符0次或1次                 |
| \{m,n\}     | 匹配其前面的字符至少m次，至多n次             |
| \{m,\}      | 匹配其前面的字符至少m次                      |
| \{m\}       | 精确匹配前面的m次\{0,n\}:0到n次              |
| \<          | 锚点词首----相当于 \b，用法格式：\<pattern   |
| \>          | 锚点词尾，用法格式:\>pattern                 |
| \<pattern\> | 单词锚点                                     |
|             | 分组，用法格式：pattern，引用\1,\2           |
| []          | 匹配指定范围内的任意单个字符                 |
| [^]         | 匹配指定范围外的任意单个字符                 |
| [:digit:]   | 所有数字, 相当于0-9， [0-9]---> [[:digit:]]  |
| [:lower:]   | 所有的小写字母                               |
| [:upper:]   | 所有的大写字母                               |
| [:alpha:]   | 所有的字母                                   |
| [:alnum:]   | 相当于0-9a-zA-Z                              |
| [:space:]   | 空白字符                                     |
| [:punct:]   | 所有标点符号                                 |



动作说明是跟在正则表达式后面 ‘ ’ 里面的

![1553573758881](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1553573758881.png)



# grep

适合单纯的查找或匹配文本

后面接正则语句



```
grep [OPTION] pattern files
```



他的正则是写在  ' ' 里面的

\是转义字符，忽略正则表达式中特殊字符的原有含义



![1553572863484](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1553572863484.png)



```
grep ‘[a-z]\{5\}’ aa
显示所有包含每个字符串至少有5个连续小写字符的字符串的行。
```



![1553572524744](C:\Users\14yhuang\AppData\Roaming\Typora\typora-user-images\1553572524744.png)



正则也可用到文件名上

```
$ grep ‘test’ d*
显示所有以d开头的文件中包含 test的行。
$ grep ‘test’ aa bb cc
显示在aa，bb，cc文件中匹配test的行。
```