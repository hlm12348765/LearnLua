# Lua学习笔记(四)

![Lua](https://www.w3cschool.cn/attachments/image/20170622/1498119030897469.png)

[**教程地址**](http://www.runoob.com/lua/lua-tutorial.html)

***

## 十三. 元表(Metatable)
> Lua 提供了元表(Metatable)，允许我们改变table的行为，每个行为关联了对应的元
方法。

* setmetatable(table,metatable): 对指定table设置元表(metatable)，如果元表(metatable)中存在__metatable键值，setmetatable会失败 。

* getmetatable(table): 返回对象的元表(metatable)。


### 1.  `__index`元方法
**由于markdown语法原因， __ 和index之间书写时增加了空格，实际没有这个空格。**

>当你通过键来访问 table 的时候，如果这个键没有值，那么Lua就会寻找该table的metatable（假定有metatable）中的 `__index`键。如果 `__index`包含一个表格，Lua会在表格中查找相应的键。   
>如果 __ index包含一个函数的话，Lua就会调用那个函数，table和键会作为参数传递给函数。

`__index` 元方法查看表中元素是否存在，如果不存在，返回结果为 nil；如果存在则由 `__index` 返回结果。

<pre><code>mytable = setmetatable({key1 = "value1"}, {
  __ index = function(mytable, key)
    if key == "key2" then
      return "metatablevalue"
    else
      return nil
    end
  end
})

print(mytable.key1,mytable.key2)
</code></pre>

实例输出结果为：

<pre><code>value1    metatablevalue
</code></pre>

等价于：

<pre><code>mytable = setmetatable({key1 = "value1"}, { __ index = { key2 = "metatablevalue" } })
print(mytable.key1,mytable.key2)
</code></pre>

>Lua查找一个表元素时的规则，其实就是如下3个步骤:
* 在表中查找，如果找到，返回该元素，找不到则继续
* 判断该表是否有元表，如果没有元表，返回nil，有元表则继续。
* 判断元表有没有`__index`方法，如果`__index`方法为nil，则返回nil；如果`__index`方法是一个表，则重复1、2、3；如果`__index`方法是一个函数，则返回该函数的返回值。

### 2.`__newindex`方法
>`__newindex` 元方法用来对表更新，`__index`则用来对表访问 。

>当你给表的一个缺少的索引赋值，解释器就会查找`__newindex`元方法：如果存在则调用这个函数而不进行赋值操作。

### 3. 表的操作符
|模式|描述|模式|描述|
|---|----|---|----|
|__add|'+'|__unm|'-' 取负|
|__sub|'-'|__contact|'..'|
|__mul|'*'|__eq|'=='|
|div|'/'|__it|'<'|
|mod|'%'|__le|'>'|

### 4. call元方法
> 函数调用操作 func(args)。 当 Lua 尝试调用一个非函数的值的时候会触发这个事件 （即 func 不是一个函数）。 查找 func 的元方法， 如果找得到，就调用这个元方法， func 作为第一个参数传入，原来调用的参数（args）后依次排在后面。

<pre><code>-- 计算表中最大值，table.maxn在Lua5.2以上版本中已无法使用
-- 自定义计算表中最大键值函数 table_maxn，即计算表的元素个数
function table_maxn(t)
    local mn = 0
    for k, v in pairs(t) do
        if mn < k then
            mn = k
        end
    end
    return mn
end

-- 定义元方法__call
mytable = setmetatable({10}, {
  __ call = function(mytable, newtable)
    sum = 0
    for i = 1, table_maxn(mytable) do
        sum = sum + mytable[i]
    end
    for i = 1, table_maxn(newtable) do
        sum = sum + newtable[i]
    end
    return sum
  end
})
newtable = {10,20,30}
print(mytable(newtable))
</code></pre>

以上实例执行输出结果为：
`70`

### 5. `__tostring`元方法
>`__tostring` 元方法用于修改表的输出行为

<pre><code>mytable = setmetatable({ 10, 20, 30 }, {
  __ tostring = function(mytable)
    sum = 0
    for k, v in pairs(mytable) do
        sum = sum + v
    end
    return "表所有元素的和为 " .. sum
  end
})
print(mytable)
</code></pre>

以上实例执行输出结果为：`表所有元素的和为 60`

***

## 十四. 协同程序

**Lua 协同程序(coroutine)与线程比较类似：拥有独立的堆栈，独立的局部变量，独立的指令指针，同时又与其它协同程序共享全局变量和其它大部分东西。**

### 1. 线程和协同程序区别
>线程与协同程序的主要区别在于，一个具有多个线程的程序可以同时运行几个线程，而协同程序却需要彼此协作的运行。

>在任一指定时刻只有一个协同程序在运行，并且这个正在运行的协同程序只有在明确的被要求挂起的时候才会被挂起。

>协同程序有点类似同步的多线程，在等待同一个线程锁的几个线程有点类似协同。

### 2. 基本语法
|方法|描述|
|----|---|
|coroutine.create()|创建coroutine，返回coroutine，参数是一个函数，当和resume配合使用的时候就唤醒函数调用|
|coroutine.resume()|重启coroutine，和create配合使用，只要调用就会返回一个boolean值|
|coroutine.yield()|挂起coroutine，将coroutine设置为挂起状态|
|coroutine.status()|查看coroutine的状态(dead/suspend/running)|
|coroutine.wrap()|创建coroutine，返回一个函数，一旦你调用这个函数，就进入coroutine|
|coroutine.running()|返回正在跑的coroutine，一个coroutine就是一个线程，当使用running的时候，就是返回一个corouting的线程号|
[Lua手册中更详细的描述](http://www.runoob.com/manual/lua53doc/manual.html#2.6)

<pre><code>--coroutine_test.lua
co = coroutine.create(
    function(i)
        print(i);
    end
)

coroutine.resume(co, 1)  -- 1
print(coroutine.status(co))  -- dead

print('---------')

co = coroutine.wrap(
    funtion(i)
        print(i);
    end  
)

co(1)

print('----------')

co2 = coroutine.create(
    function()
        for i=1,10 do
            print(i)
            if i == 3 then
                print(coroutine.status(co2))  -- running
                print(coroutine.running())  -- thread:XXXXXX
            end
            coroutine.yield()
        end
    end
)

coroutine.resume(co2)  -- 1
coroutine.resume(co2)  -- 2
coroutine.resume(co2)  -- 3

print(coroutine.status(co2))  -- suspended
print(coroutine.runnning())

print('-----------')
</code></pre>

以上实例执行输出结果为：

<pre><code>1
dead
----------
1
----------
1
2
3
running
thread: 0x7fb801c05868    false
suspended
thread: 0x7fb801c04c88    true
----------
</code></pre>

> coroutine.running就可以看出来,coroutine在底层实现就是一个线程。

>当create一个coroutine的时候就是在新线程中注册了一个事件。

>当使用resume触发事件的时候，create的coroutine函数就被执行了，当遇到yield的时候就代表挂起当前线程，等候再次resume触发事件。

**协程的运行可能被两种方式终止： 正常途径是主函数返回 （显式返回或运行完最后一条指令）； 非正常途径是发生了一个未被捕获的错误。 对于正常结束， coroutine.resume 将返回 true， 并接上协程主函数的返回值。 当错误发生时， coroutine.resume 将返回 false 与错误消息。**


***

# 十五. 文件I/O

Lua I/O 库用于读取和处理文件。分为简单模式（和C一样）、完全模式。
* 简单模式（simple model）拥有一个当前输入文件和一个当前输出文件，并且提供针对这些文件相关的操作。   

* 完全模式（complete model） 使用外部的文件句柄来实现。它以一种面对对象的形式，将所有的文件操作定义为文件句柄的方法

打开文件操作语句如下：

<pre><code>file = io.open (filename [, mode])
</code></pre>

mode的值有：

|模式|描述|
|----|---|
|r|以只读方式打开文件，该文件必须存在|
|w|打开只写文件，若文件存在则文件长度清零，若文件不存在则创建|
|a|以附加的方式打开只写文件。若文件不存在，则会建立该文件，如果文件存在，写入的数据会被加到文件尾，即文件原先的内容会被保留。（EOF符保留）|
|r+|以可读写方式打开文件，该文件必须存在|
|w+|打开可读写文件，若文件存在则文件长度清零，若文件不存在则创建|
|a+|与a类似，但该文件可读写|
|b|二进制模式，如果文件是二进制文件，可以加上b|
|+|加号表示对文件既可以读又可以写|

## 1. 简单模式


***

# 十六. 错误处理

* 语法错误
* 运行错误

## 1. 错误处理

### 1.1 assert函数

<pre><code>local function add(a, b)
    assert(type(a) == "number", "a 不是一个数字")
    assert(type(b) == "number", "b 不是一个数字")
    return a+b
end
add(10)
</code></pre>

执行以上程序会出现如下错误：

<pre><code>lua: test.lua:3: b 不是一个数字
stack traceback:
    [C]: in function 'assert'
    test.lua:3: in local 'add'
    test.lua:6: in main chunk
    [C]: in ?
</code></pre>

实例中assert首先检查第一个参数，若没问题，assert不做任何事情；否则，assert以第二个参数作为错误信息抛出。

### 1.2 error函数
`error (message [, level])`

>功能：终止正在执行的函数，并返回message的内容作为错误信息(error函数永远都不会返回)

>通常情况下，error会附加一些错误位置的信息到message头部。

>Level参数指示获得错误的位置:

>* Level=1[默认]：为调用error位置(文件+行号)   

>* Level=2：指出调用error的函数的函数   

>* Level=0:不添加错误位置信息   

## 2. pcall和xpcall、debug

>Lua中处理错误，可以使用函数pcall（protected call）来包装需要执行的代码。   

>pcall接收一个函数和要传递个后者的参数，并执行，执行结果：有错误、无错误；返回值true或者或false, errorinfo。

<pre><code>if pcall(function_name, ….) then
-- 没有错误
else
-- 一些错误
end
</code></pre>
