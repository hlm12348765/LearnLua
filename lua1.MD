# Lua学习笔记(一)

![Lua](https://www.w3cschool.cn/attachments/image/20170622/1498119030897469.png)

[**教程地址**](https://www.w3cschool.cn/lua)
***
## 一. 基本语法
### 1. 注释  

**单行注释**  
<pre><code> --单行注释 </code></pre>  

**多行注释**
<pre><code> --[[
多行注释
 --]]</code></pre>  

### 2. 标示符与保留字  

**标示符**  

大小写字母或者下划线开头加上字母或者下划线或者数字，Lua区分大小写。  

**保留字**  

下划线加大写字母一般用于表示Lua的保留字:  

| and | break | do | else |  
|-----|:-----:|:---:|----:|  
|elseif | end | false | for |  
|function | if | in | local |  
| nil | not | or | repeat | while |  
|return | then | true | until |  

### 3. 全局变量
变量默认为全局变量，全局变量不需声明，赋值即创建。访问一个没有初始化的全局变量，将得到结果nil，想删除一个变量，只需要把它赋值为nil。当且仅当一个变量不等于nil时，这个变量存在。  

***
## 二. 数据类型  

**Lua是动态类型语言**
### 1. 基本数据类型  
|数据类型|描述|  
|-----|:----:|
|nil|表示无效值|  
|boolean| false/true|  
|number| 双精度类型的实浮点数|  
|string|字符串由单/双引号表示|  
|function| 由C或者Lua编写的函数|  
|userdata|表示任意存储在变量中的C数据结构|  
|thread|表示执行的独立线路，用于执行协同程序|
|table|Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以是数字或者是字符串。在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。|  

### 2. nil
对全局变量和table赋值nil，相当于将它们删除。  
### 3. boolean
false和nil为“假”，其他为“真”。
### 4. number  
Lua默认只有双精度实型(可修改luaconf.h)
### 5. string
字符串可以用单引号''双引号""或者2个方括号[[]]表示。  

在对一个数字字符串上进行算术操作时，Lua 会尝试将这个数字字符串转成一个数字:
<pre><code> > print('2'+6)
 8.0   
 > print('2+6')  
 2+6  </code></pre>  

使用..来连接字符串，使用#来计算字符串长度:
<pre><code> > print('a'..'b'..'c')
 abc  
>print(#"www.w3cschool.cn")
 16</code></pre>  

### 6. table
**Lua里表的默认索引一般从1开始**  

table 不会固定长度大小，有新数据添加时 table 长度会自动增长，没初始的 table 都是 nil。

### 7. function
在Lua中，函数是被看作“第一类值(First-Class Value)”，函数可以存在变量里：
<pre><code> -- function_test.lua

 function factorial1(n)
     if n == 0 then
         return 1
     else
         return n * factorial1(n-1)
     end
 end

 print(factorial1(5))
 factorial2 = factorial1
 print(factorial2(5)) </code></pre>

脚本执行结果为：
<pre><code> $ lua function_test.Lua
 120
 120 </code></pre>

function 可以以匿名函数（anonymous function）的方式通过参数传递:
<pre><code> -- function_test2.Lua

 function anonymous(tab, fun)
     for k, v in pairs(tab) do
         print(fun(k, v))
     end
 end

 tab = { key1 = "val1", key2 = "val2" }
 anonymous(tab, function(key, val)
     return key .. " = " .. val
 end) </code></pre>

脚本执行结果为：
<pre><code> $ lua function_test2.Lua
 key1 = val1
 key2 = val2 </code></pre>

### 8. thread
在 Lua 里，最主要的线程是协同程序（coroutine）。它跟线程（thread）差不多，拥有自己独立的栈、局部变量和指令指针，可以跟其他协同程序共享全局变量和其他大部分东西。

线程跟协程的区别：线程可以同时多个运行，而协程任意时刻只能运行一个，并且处于运行状态的协程只有被挂起（suspend）时才会暂停。

### 9. userdata
userdata 是一种用户自定义数据，用于表示一种由应用程序或 C/C++ 语言库所创建的类型，可以将任意 C/C++ 的任意数据类型的数据（通常是 struct 和 指针）存储到 Lua 变量中调用。
***
## 三. 变量
Lua中的变量包括3种类型：  
* 全局变量：函数外的变量默认为全局变量，除非用local显式地声明。
* 局部变量：函数内的变量与函数参数默认为局部变量，作用域为从声明位置开始到所在语句快结束，或者直到下一个同名局部变量声明。
* 表中的域  

**变量的默认值均为nil**
### 1. 赋值语句
赋值是改变一个变量的值和改变表域的最基本的方法。  
<pre><code> a, b = 10, 2*x    <-->   a = 10, b = 2*x
 x, y = y, x  -- swap 'x' for 'y'  (类似python)
 a. 变量个数 > 值的个数    按变量个数补足nil
 b. 变量个数 < 值的个数    多余的值会被忽略</code></pre>

多值赋值经常用来交换变量，或将函数调用返回给变量：
<pre><code> a, b = f() </code></pre>
f()返回两个值，第一个赋给a，第二个赋给b。  

**尽可能使用局部变量：**  
1. 避免命名冲突  
2. 访问局部变量的速度比全局变量更快  

### 2. 索引
对table的索引使用[]，或成员运算符.
<pre><code> t[i]
 t.i                     -- 当索引为字符串类型时
 gettable_event(t, i)    -- 本质的函数调用</code></pre>

***

## 四. 循环

![循环](https://img.w3cschool.cn/attachments/image/20160829/1472467762955033.png)
### 1. Lua循环
|循环类型|描述|  
|-------|----|
|while循环|在条件为 true 时，让程序重复地执行某些语句。执行语句前会先检查条件是否为 true。|
|for循环|重复执行指定语句，重复次数可在 for 语句中控制。|
|repeat...until|重复执行循环，直到指定条件为真时停止|
|循环嵌套|可以在循环内嵌套一个或多个循环语句（while、for、do..while）|
#### while循环  
 语法：  
 <pre><code> while (condition)
   do
     statements
   end</code></pre>
![while](https://img.w3cschool.cn/attachments/image/lua_while_loop.jpg)  

#### for循环
* 数值for循环
<pre><code> for var = exp1, exp2, exp3 do
    <执行体>
   end</code></pre>
var从exp1变化到exp2，每次变化以exp3为步长递增var，并执行一次"执行体"。exp3是可选的，如果不指定，默认为1。
<pre><code> for i=1, f(x) do
    print(i)
   end
   -- [[for的三个表达式在循环开始前一次性求值，以后不
 再进行求值。比如上面的f(x)只会在循环开始前执行一次，
 其结果用在后面的循环中。]]-- </code></pre>

* 泛型for循环  
泛型for循环通过一个迭代器函数来遍历所有值。
<pre><code> -- 打印数组a的所有值
 for i, v in ipairs(a)
     do print(v)
 end</code></pre>

#### repeat...until循环
<pre><code> repeat
    statements
 until (condition)</code></pre>

 ![repeat...until](http://www.runoob.com/wp-content/uploads/2015/05/lua_repeat_until_loop.jpg)

####  循环嵌套
* for循环嵌套

<pre><code>for init, max/min value, increment
do
     for init, max/min value, increment
     do
        statements
     end
     statements
end
</code></pre>

<pre><code>j =2
for i=2,10 do
   for j=2,(i/j) , 2 do
      if(not(i%j))
      then
         break
      end
      if(j > (i/j))then
         print("i 的值为：",i)
      end
   end
end</code></pre》
执行结果为：
<pre><code>i 的值为：	8
i 的值为：	9
i 的值为：	10</code></pre>

### 2. 循环控制语句
|控制语句|描述|  
|-------|----|
|break|退出当前循环或语句，开始执行紧接着的语句|

![break](http://www.runoob.com/wp-content/uploads/2015/05/cpp_break_statement.jpg)

### 3. 无限循环
<pre><code>while (true)
do
    print(“循环将永远执行下去”)
end</code></pre>
