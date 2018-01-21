# Lua学习笔记(三)

![Lua](https://www.w3cschool.cn/attachments/image/20170622/1498119030897469.png)

[**教程地址**](https://www.w3cschool.cn/lua)
***

## 十一. table(表)
### 1. table(表)的构造
>构造器是创建和初始化表的表达式。表是Lua特有的功能强大的东西。最简单的构造函数是{}，用来创建一个空表。可以直接初始化数组:  

<pre><code>-- 初始化表
mytable = {}

-- 指定值
mytable[1]= "Lua"

-- 移除引用
mytable = nil
-- lua 垃圾回收会释放内存
</code></pre>

>当我们为 table a 并设置元素，然后将 a 赋值给 b，则 a 与 b 都指向同一个内存。如果 a 设置为 nil ，则 b 同样能访问 table 的元素。如果没有指定的变量指向a，Lua的垃圾回收机制会清理相对应的内存。   

<pre><code>-- 简单的 table
mytable = {}
print("mytable 的类型是 ",type(mytable))

mytable[1]= "Lua"
mytable["wow"] = "修改前"
print("mytable 索引为 1 的元素是 ", mytable[1])
print("mytable 索引为 wow 的元素是 ", mytable["wow"])

-- alternatetable和mytable的是指同一个 table
alternatetable = mytable

print("alternatetable 索引为 1 的元素是 ", alternatetable[1])
print("mytable 索引为 wow 的元素是 ", alternatetable["wow"])

alternatetable["wow"] = "修改后"

print("mytable 索引为 wow 的元素是 ", mytable["wow"])

-- 释放变量
alternatetable = nil
print("alternatetable 是 ", alternatetable)

-- mytable 仍然可以访问
print("mytable 索引为 wow 的元素是 ", mytable["wow"])

mytable = nil
print("mytable 是 ", mytable)
</code></pre>

以上代码执行的结果为：

<pre><code>mytable 的类型是     table
mytable 索引为 1 的元素是     Lua
mytable 索引为 wow 的元素是     修改前
alternatetable 索引为 1 的元素是     Lua
mytable 索引为 wow 的元素是     修改前
mytable 索引为 wow 的元素是     修改后
alternatetable 是     nil
mytable 索引为 wow 的元素是     修改后
mytable 是     nil
</code></pre>

### 2. table操作

|方法|用途|
|----|---|
|table.contact(table[,sep[,start[,end]]])|出参数中指定table的数组部分从start位置到end位置的所有元素, 元素间以指定的分隔符(sep)隔开|
|table.insert(table,[,pos]value)|在table的数组部分指定位置(pos)插入值为value的一个元素. pos参数可选, 默认为数组部分末尾.|
|table.maxn(table)|指定table中所有正数key值中最大的key值. 如果不存在key值为正数的元素, 则返回0。**Lua5.2之后该方法已经不存在了**|
|tabel.remove(table[,pos])|返回table数组部分位于pos位置的元素. 其后的元素会被前移. pos参数可选, 默认为table长度, 即从最后一个元素删起|
|table.sort(table[,comp])|对给定的table进行升序排序|

***
## 十二. 模块与包

>模块类似于一个封装库，从 Lua 5.1 开始，Lua 加入了标准的模块管理机制，可以把一些公用的代码放在一个文件里，以 API 接口的形式在其他地方调用，有利于代码的重用和降低代码耦合度。

>Lua 的模块是由变量、函数等已知元素组成的 table，因此创建一个模块很简单，就是创建一个 table，然后把需要导出的常量、函数放入其中，最后返回这个 table 就行。

### 1. require函数    
**require函数用来加载模块**   

`require("module")`或者`require"module"`  

>执行 require 后会返回一个由模块常量或函数组成的 table，并且还会定义一个包含该 table 的全局变量。

**可以给模块取别名`local m = require("module")`**
#### 加载机制
>require函数的搜索路径放在全局变量`package.path`中，Lua启动后会以环境变量`LUA_PATH`的值将其初始化。如果没找到这个环境变量，则使用一个编译时定义的默认路径来初始化。

设置`LUA_PATH`环境变量：
<pre><code>~/.profile或者~/.bashrc

#LUA_PATH
export LUA_PATH="~/lua/?.lua;;"
</code></pre>

>文件路径以 ";" 号分隔，最后的 2 个 ";;" 表示新加的路径后面加上原来的默认路径。

>如果找到目标文件，则会调用 package.loadfile 来加载模块。否则，就会去找 C 程序库。

>搜索的文件路径是从全局变量 package.cpath 获取，而这个变量则是通过环境变量 LUA_CPATH 来初始。

>搜索的策略跟上面的一样，只不过现在换成搜索的是 so 或 dll 类型的文件。如果找得到，那么 require 就会通过 package.loadlib 来加载它。


### 2. C包

>C包在使用以前必须首先加载并连接，在大多数系统中最容易的实现方式是通过动态连接库机制。

>Lua在一个叫loadlib的函数内提供了所有的动态连接的功能。这个函数有两个参数:库的绝对路径和初始化函数。所以典型的调用的例子如下:

<pre><code>local path = "/usr/local/lua/lib/libluasocket.so"
local f = loadlib(path, "luaopen_socket")
</code></pre>

>oadlib函数加载指定的库并且连接到Lua，然而它并不打开库（也就是说没有调用初始化函数），反之他返回初始化函数作为Lua的一个函数，这样我们就可以直接在Lua中调用他。

>如果加载动态库或者查找初始化函数时出错，loadlib将返回nil和错误信息。我们可以修改前面一段代码，使其检测错误然后调用初始化函数：

<pre><code>local path = "/usr/local/lua/lib/libluasocket.so"
local f = assert(loadlib(path, "luaopen_socket"))
f()  -- 真正打开库
</code></pre>

>一般情况下我们期望二进制的发布库包含一个与前面代码段相似的stub文件，安装二进制库的时候可以随便放在某个目录，只需要修改stub文件对应二进制库的实际路径即可。

>将stub文件所在的目录加入到LUA_PATH，这样设定后就可以使用require函数加载C库了。
