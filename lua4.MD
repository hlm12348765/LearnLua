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
|coroutine.resume()|重启coroutine，和create配合使用|
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
