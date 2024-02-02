---
title: 'Cool Language'
date: 2015/06/11
tag: programming_language
author: Andy
---
斯坦福的编译课程中的需要做出来的语言，[完整介绍地址](https://spark-university.s3.amazonaws.com/stanford-compilers/resources%2Fcool_manual.pdf)。因为在这个课程中需要把这个语言完成做出来，所以熟悉好这门语言和各种细节是很有必要的。

由于 Virtual Box 的 GUI 看得不爽，找了一下发现了[这个](https://github.com/dlackty/vagrant-cool)。顺着路子找发现了[这个](https://lagunita.stanford.edu/courses/Engineering/Compilers/Fall2014/6b750292e90d4950b895f621a5671b49/)，原来可以在本地上安装好课程环境。我就用 Vagrant 了。

<!--more-->

### Cool语言介绍
特性关键字：  
1. object-oriented   
2. static typing and  type safe  
3. automatic memory management  
4. expression  

Cool语言基于 [MIPS处理器](http://baike.baidu.com/link?url=bohPeNDUZwmg-Spb-gWGrLaN8rAUI0O-QSZTl1-ive7Qd57Bl5zpTEVMZKmptN_9qGtbq1RZA7EfmAWaJ-EpNq) 处理。
本课程中用使用了一种 MIPS 处理器的模拟器 spim。课程上的那个 spim 是 misp 的变种，可以直接运行 cool 语言编译出来的可执行文件。  
  
编译和执行过程就是：  
  
cool 源码-----> spim 的可执行文件-----> spim 翻译成机器指令----->执行

编译命令：

```
coolc file1.cl
```

生成目标可执行文件`file1.s`  


用 spim 虚拟机执行命令：    

```bash
$ spim #进入模拟环境
(spim) load "file1.s" #模拟环境操作
(spim) run
(spim) reinit #清内存，重新加载环境

$ spim -file file.s #也可以直接执行
```

### 类
源文件必须有一个定义好 `main` 方法的 `Main` 类，每个源文件可以有多个类。所有类对外可见。类名称必须大写。类不可重写。  

### 类内部声明  
```
class Cons inherits List  {
  xcar : Int; #属性
  xcdr : List; #属性

  isNil() : Bool { false }; #方法

  init(hd : Int, tl : List) : Cons { #构造方法
    {
      xcar <- hd;
      xcdr <- tl;
      self;
    }
  };

};
```
内部声明只有两种类型：属性和方法，都不可重复定义，首字母都必须小写。方法的最后一行语句为该方法返回值，用 return 可中途返回。赋值符号用`->`而不是`=` 。类构造方法为 `init()` 。没有析构方法，因为用的是自动垃圾回收机制 GC。  

### 继承
```
class C inherits P { ... };
```
采用单继承。C继承自P，C享有P所有属性和方法。声明类不使用继承的话，默认继承自Object。Object是最终类，Int, String, Bool, and IO是基类。

### 类型
所有声明的类名都变成类型。所有变量和参数都需要声明类型。假如C继承自P，那么P声明的变量也可用C来代替。使用 SELF_TYPE 可以返回自身的类类型。编译运行时必须保证所有变量和返回都有类型，保证运行时不会有类型错误发生，这是静态检查，所以 Cool 是静态类型语言。

### 属性
```
<id> : <type> [ <- <expr> ];
```

后面执行语句是可选的，假如没有在声明时指定值，默认值为 void。void 没有值，可以用 `isvoid` 方法判断是否为 void。

### 方法
```
<id>(<id> : <type>,...,<id> : <type>): <type> { <expr> };
```

参数和返回值要声明好返回类型。继承的子类可以重载方法。
### 表达式语句
### 常量
布尔类型常量 true, false。整型类型常量以存数字出现。字符串常量以 "xxxxx" 出现，最大为1024个字。三种常量基于类 Bool，Integer，String。

### 标示符
包括本地变量，方法参数，self，类属性。self 不能以赋值形式出现。

### 赋值
```
<id> <- <expr>
```
可以出现声明式赋值，也可以隐式赋值，类型为值类型。

### 方法调用
```
<expr>.<id>(<expr>,...,<expr>)
<id>(<expr>,...,<expr>) #相当于self.<id>(<expr>,...,<expr>)
<expr>@<type>.id(<expr>,...,<expr>)
```

### 条件判断
判断的条件类型必须为 Bool 型。
```
if <expr> then <expr> else <expr> fi
```

### 循环
同样，判断的条件类型必须为Bool型。
```
while <expr> loop <expr> pool
```

### 块
```
{ <expr>; ... <expr>; }
```
块后面可以不加;号，返回值为最后的语句的值。

### let赋值
```
let <id1> : <type1> [ <- <expr1> ], ..., <idn> : <typen> [ <- <exprn> ] in <expr>
```

### case
```
case <expr0> of
<id1> : <type1> => <expr1>;
. . .
<idn> : <typen> => <exprn>;
esac
```

### new 创建对象
```
new <type>
```

### Isvoid
判断是否为 void
```
isvoid expr
```

### 算术符号
四种基本的算术符号：`+ - * /`。
```
expr1 <op> expr2
```
左右表达式的类型必须为 Integer，Cool 中只有整形算术。  
比较类型`< <= >`。`< <=`可以比较同类型的表达式，返回为 true 或 false。`=`比较地址是否相同，值相同为 false。

### 基本类

### Object
Object 是所有继承链的最终类。
```
abort() : Object
type_name() : String
copy() : SELF_TYPE
```
提供三个方法，abort() 会抛出错误，type_name() 会返回当前对象的类型名，copy() 会返回当前对象的一个副本，但是只是一个复制，不是指针，也不是引用。

### IO
```
out_string(x : String) : SELF_TYPE
out_int(x : Int) : SELF_TYPE
in_string() : String
in_int() : Int
```
out_string 和 out_int 对应打印出 String, Integer，返回 IO 类型。in_string, in_int对应输入 String, Integer。

### Int
只提供类型和值，初始化默认值为0。不能从 Int 类继承。

### String
```
length() : Int
concat(s : String) : String
substr(i : Int, l : Int) : String
```
初始化默认值为""。不能从 String 中继承。

### Bool
只提供类型和值，初始化默认值为 false。不能从 Bool 类中继承。

### Main
每个源文件必须有一个Main类，类中必须有一个main方法。执行程序从main方法开始。

### 词法结构

### Integers, Identifiers, and Special Notation
整数为0~9的数，标识符由字母，数字，底线符号组成，类型名第一个字母为大写，对象第一个字母为字符。keyword 不能作为声明类型

### String
"\x"翻译成"x"，EOF 不能包含在字符串里

```
\b backspace
\t tab
\n newline
\f formfeed
```

### Comments
两个`--`之间的语句，`(*...*)`中间。

### KeyWords

```
class, else, false, fi, if, in, inherits, isvoid, let, loop, pool, then, while,case, esac, new, of, not, true
```
### White Space

```
blank (ascii 32), 
\n (newline, ascii 10),
\f (form feed, ascii 12),
\r (carriage return, ascii 13),
\t (tab, ascii 9),
\v (vertical tab, ascii 11) d
```





