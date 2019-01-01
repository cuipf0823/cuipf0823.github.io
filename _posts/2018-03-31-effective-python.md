---
# layout: post
title: Effective python读书笔记
category: python
---


[toc]

# 了解bytes, str与unicode的区别
1. python3有两种字符序列的类型:bytes和str ;
> bytes实例包含的就是二进制数据, str的实例包含的unicode字符; 
2. python2也包含两种字符序列的类型: str 和 unicode ;
> str的实例包含是二进制数据, unicode则包含的unicode字符;

## 转换函数

```python
def to_str(bytes_or_str):
    """
    python3 接受str或者bytes字符串, 返回str
    """
    if isinstance(bytes_or_str, bytes):
        value = bytes_or_str.decode('utf-8')
    else:
        value = bytes_or_str
    return value


def to_bytes(bytes_or_str):
    """
    python3 接受str或者bytes字符串, 返回bytes
    """
    if isinstance(bytes_or_str, str):
        value = bytes_or_str.encode('utf-8')
    else:
        value = bytes_or_str
    return value
```

## 建议
* python编码时候, 一定要把编码和解码放在外围来做, 程序的核心部分使用unicode字符类型(python3的str, python2的unicode);
* 从文件中读取二进制数据, 或向其中写入二进制数据, 总应该以'rb'或'wb'等二进制模式开启文件;


# 使用辅助函数代替复杂的表达式
* 避免特别复杂以及难以理解的单行表达式;
* 把复杂的表达式移入辅助函数之中, 如果要反复使用相同的逻辑, 更应该这么做;
* 使用"if/else"表达式, 要比用"or"和"and"这样的boolean操作符写成的表达式更加的清晰;

```python
//第一种
red = int(values.get('red', [''])[0] or 0)

//第二种 优于第一种
red = values.get('red', [''])
red = int(red[0]) if red[0] else 0

//第三种 优于前两种
def get_first_int(values, key, default=0):
    found = values.get(key, [''])
    if found[0]:
        found = int(found[0])
    else:
        found = default
    return found

red = get_first_int(values, 'red')
```

# 了解切割序列的办法

* python提供了一种把序列切成小块的写法, 使得开发者可以轻易的访问序列中某些元素所构成的子集; 主要是对内置的list, str和bytes进行切割;
* 对原列表进行切割后, 会产生另外一份全新的列表; 在切割后得到的新列表上进行修改, 不会影响原列表; 
* 在赋值时候对左侧列表使用切割操作, 会把列表中处于指定范围内的对象替换为新值, 位于切片范围前后的值保留不变, 列表会根据新值的个数相应的扩张或者收缩;

```python
In [14]: a
Out[14]: ['a', 'b', 'c', 'e', 'f', 'g', 'h']
In [15]: a[2:7] = [33, 44, 55]
In [16]: a
Out[16]: ['a', 'b', 33, 44, 55]
```
* 切片操作不会比较start和end是否越界, 这样很容易就能从序列的前端和后端开始, 对其进行范围固定的切片操作(如a[:20]或a[-20:]);
* 对于切片操作, 既有start和end, 又有stride, 可能会令人费解;
* 即使要用stride也要使用正数;避免使用负数;

# 用生成器表达式来改写数据量较大的列表推导
列表推导缺点: 在推导过程中, 对于输入列表中的每个值, 可能都要创建仅包含一项元素的全新列表, 当数据较少的时候, 没有问题, 但是数据量大的时候, 会消耗大量的内存, 并导致程序崩溃; 

## 解决方式
python提供了生成器表达式, 它是对列表推导和生成器的一种泛化, 生成器表达式在运行的时候, 并不会把整个输出序列呈现出来, 而是会估值为迭代器(iterator);

把实现列表推导所用的那种写法放在一对括号内, 就构成了生成器表达式; 

```python

it = (len(x) for x in open('c://111.ini'))
print(it)
<generator object <genexpr> at 0x072DB0F0>

print(next(it))
10

print(next(it))
18

```

* 由生成器表达式返回的迭代器, 可以逐次产生输出值, 从而避免了内存用量的问题; 

# 尽量使用enumerate取代range

## 问题

```python
for i in range(len(a)):
    print(a[i])

output:
a
b
c
e
f
g
h
```
使用range做遍历, 我们必须现获取长度, 然后通过下标来访问,  代码很生硬, 且不便于理解;

## 解决
使用enumerate代替range; 

```python
for i, item in enumerate(a):
    print('{}.{}'.format(i+1, item))
1.a
2.b
3.c
4.e
5.f
6.g
7.h
for i, item in enumerate(a, 3):
    print('{}.{}'.format(i+1, item))
4.a
5.b
6.c
7.e
8.f
9.g
10.h
```

## 说明
* 尽量使用enumerate来修改那种将range与下标访问结合的遍历;
* enumerate可以提供第二个参数, 以指定开始计数时所用的值(默认为0);

# 使用zip函数同时遍历连个迭代器
* 在python3中zip函数, 可以把两个或两个以上的迭代器封装为生成器, 会在遍历过程中逐次产生元组, 而python2中zip则直接把这些元组完全生成好, 并一次性返回整个列表(python2中, 可能会占用大量内存并导致程序崩溃);
* 如果提供的生成器长度不等, zip会自动提前终止(按照短的长度);
* itertools内置模块中zip_longest函数可以平行地遍历多个迭代器, 而不用在乎他们长度是否相等;

```python
for name, alphy in zip(names, a):
    print('{}.{}'.format(name, alphy))

cipfi.a
yonyn.b
ddddd.c
ccc.e

for name, alphy in itertools.zip_longest(names, a):
     print('{}.{}'.format(name, alphy))

cipfi.a
yonyn.b
ddddd.c
ccc.e
None.f
None.g
None.h
```


# 不要在for和while循环后面写else块

* 在python中允许for以及while循环的内部语句块之后紧跟一个else块;
* 只要主循环体没有遇到break, 循环后面的else块才会执行; 
* 不要在循环后面使用else模块, 这种写法不直观且容易引起误解; 

# 尽量使用异常来表示特殊情况, 而不是返回None

## 问题
```python
def divide(a, b):
    try:
        return a/b
    except ZeroDivisionError:
        return None
```
编写工具函数时候, 我们喜欢给None赋予特殊意义, 有时候这么做是不合理的; 设想如果分子"a=0", 那么这个函数返回为0;
调用者在调用之后, 使用if判断无论是"0", "None"和"False"都是等效的; 我们一般不会去专门判断是否等于None. 这样编写函数, 存在很大问题;

## 解决

```python

def divide(a, b):
    try:
        return a/b
    except ZeroDivisionError as e:
        raise ValueError('Invalid inputs') from e

x, y = 5, 2
try:
    result = divide(x, y)
except ValueError:
    print('Invalid inputs')
else:
    print('result is {}'.format(result))

```
编写这样的函数, 我们可以不使用None, 而是把异常抛给上一级, 使得调用者必须应对它; 我们可以使用相应的注释解释清楚; 

## 说明

函数在遇到特殊情况下, 应该抛出异常, 而不是返回None. 调用者看到该函数的文档中描述的异常之后, 应该编写相应的代码处理它们; 

# 了解如何在闭包里使用外围作用域中的变量

## 问题

```python
def sort_priority(values, group):
    found = False

    def helper(x):
        if x in group:
            # 这里的found并不是上面的found，而是重新定义了found
            found = True
            return 0, x
        return 1, x
    values.sort(key=helper)
    return found
```
上述函数功能很简单, 对数字列表排序, 并把出现在group中的数字, 放在出现group外的那些数字之外; 

当然, 上述代码排序没有问题, 但是返回值found不对, 问题出在哪里?

在表达式中引用变量, python解释器将按如下顺序遍历各作用域, 以解析该引用:
1. 当前函数作用域; 
2. 包含外围作用域(例如: 包含当前函数的其他函数);
3. 包含当前代码的那个模块的作用域(也叫全局作用域, global scope);
4. 内置作用域(也就是包含len及str等函数的那个作用域);
如果上面这些地方都没有定义过名称相符的变量, 那就抛出NameError异常; 

给变量赋值时, 规则所有不同, 如果当前作用域内已经定义了这个变量, 那么该变量就会具备新值，若当前作用域没有这个变量，python则会把这次赋值视为该变量的定义； 
上面返回值错误问题，就是我们常常成为的作用域bug，这是python设计有意为之，防止函数中局部变量污染外部模块； 

## 解决
python3中有一种特殊的写法，能够获取闭包内的数据，我们可以使用**nolocal**语句表明我们的意图，也就是：给相关变量赋值的时候，应该在上层作用域查找该变量；
**nolocal**的唯一限制在于，他不能延伸到模块级别，这是为了防止它污染全局作用域；

```python
def sort_priority(values, group):
    found = False

    def helper(x):
        nolocal found
        if x in group:
            # 这里的found并不是上面的found，而是重新定义了found
            found = True
            return 0, x
        return 1, x
    values.sort(key=helper)
    return found
```

**注意**
* 千万不要乱用nolocal，特别是在复杂的函数，修饰某个变量的nolocal语句可能和赋值操作离的很远，很容易导致代码难以理解； 
* python2并不支持**nolocal**，程序可以使用可变值（eg：包含单个元素的列表）来实现与nolocal相仿的机制，如下;

```python

def sort_priority(values, group):
    found = [False]

    def helper(x):
        if x in group:
            found[0] = True
            return 0, x
        return 1, x
    values.sort(key=helper)
    return found[0]

```

# 考虑使用生成器来改写直接返回列表的函数

如果函数要产生一系列的结果，最简单的方式就是返回一个包含这些结果的列表；

```python 
def index_words(text):
    result = []
    if text:
        result.append(0)
    for index, letter in enumerate(text):
        if letter == ' ':
            result.append(index + 1)
    return result
```
这个函数有两个问题： 
1. 函数代码显得很臃肿；
2. 如果传递的参数text非常巨大，很可能会造成内存消尽并崩溃； 

我们可以使用生成器的方式替换上述方法，这样代码更加清晰，无论输入量和输出量有多大都不会影响执行时消耗的内存；
优化修改如下： 

```python
def index_words_iter(text):
    if text:
        yield 0
    for index, letter in enumerate(text):
        if letter == ' ':
            yield index + 1

# 调用该函数
list1 = list(index_words_iter(text))

```

# 使用数量可变的位置参数减少视觉杂讯

```python

def log_old(message, values):
    if not values:
        print(message)
    else:
        values_str = ','.join(str(x) for x in values)
        print('{}.{}'.format(message, values_str))

def log(message, *values):
    if not values:
        print(message)
    else:
        values_str = ','.join(str(x) for x in values)
        print('{}.{}'.format(message, values_str))

log_old('hi, there', [])
log_old('my scores are', [1, 2])

log('hi, there')
log('my scores are', 1, 2)
favor = [1, 2, 3, 4, 5, 6, 7]
#如果favor足够大，可能会存在消耗大量内存的情况，并导致程序崩溃
log('my favor:', *favor)
```
相比而言： 
1. log函数的调用比log_old更加的友好，没有第二参数时候，无需传入空的列表；

但是，log函数有哪些隐藏的问题呢？
1. 变长参数在传给函数时候，总是要先转化为元组（tuple），这意味着，如果使用带有“*”操作符的生成器为参数，来调用这种函数python会把该生成器完整的迭代一轮放入元组中，可能会存在消耗大量内存的情况，并导致程序崩溃；
2. 使用\*arg参数后，以后需要为该函数添加新的参数，就必须修改原有的代码，不方便扩展（*arg必须为函数的最后一个参数）；

# 使用None和文档字符串来描述具有动态默认值的参数

## 问题

有时候，我们会采用一种非静态的类型，来做关键字参数的默认值，如下示例：

```python

def log_default(message, when=datetime.now()):
    print('{}:{}'.format(when, message))
 
# 调用  
log_default('Hi there')
time.sleep(1)
log_default('Hi again')

# 输出
2018-04-12 15:52:44.755341:Hi there
2018-04-12 15:52:44.755341:Hi again
```
从输出可以看出两条消息的时间戳（timestamp）是一样的，这明显不是我们想要的；
也就是说“datetime.now()”只在函数定义的时候执行了一次；包含这段代码的模块一旦加载进来，参数的默认值就固定不变了，程序不会再次执行datetime。

## 解决
在python中，如果想要正确的实现默认值，习惯上是把默认值设为None，并在文档字符串里面把None所对应的实际行为描述出来即可；优化如下：

```python

def log_default_none(message, when=None):
    """
     log a message with a timestamp.
     args:
     message: Message to print
     when: datetime of when the message occured.
           Defaults to the present time.
    """
    when = datetime.now() if when is None else when
    print('{}:{}'.format(when, message))
```

## 注意
如果参数的实际默认值是可变（mutable）参数，那就一定要记得用None作为形式上的默认值。

```python 

def decode(data, default={}):
    try:
        return json.loads(data)
    except ValueError:
        return default

# 调用
foo = decode('bad data')
foo['stuff'] = 5
bar = decode('also bad')
bar['mess'] = 1
print('Foo:', foo)
print('Bar:', bar)

# 输出
Foo: {'stuff': 5, 'mess': 1}
Bar: {'stuff': 5, 'mess': 1}
```
foo和bar是同一个字典，这明显不是我们想要的，解决方式同样是：把关键字参数的默认值设为None，并在函数的文档字符串中描述它的实际行为；

# 尽量用辅助类来维护程序的状态，而不要用字典和元组
1. 代码中不要使用包含字典的字典；也不要使用过长的元组；
2. 保存内部状态的字典如果变得比较复杂，那就应该把这些代码拆解为多个辅助类；

# 使用super来初始化类
初始化父类的传统方式，是在子类的实例中调用父类的\__init\__方法；

```python
class BaseClass(object):
    def __init__(self, value):
        self._value = value


class ChildClass(BaseClass):
    def __init__(self):
        BaseClass.__init__(self, 5)
```

使用这种传统的方法，会有以下问题：
1. 调用父类的\__init__的顺序不固定，完全由程序员自己控制；
2. 在钻石继承体系中，使用这种传统的方法，会使得钻石继承顶部的那个公共基类多次执行\__init__多次调用，从而产生意向不到的结果；

在python2.2中，增加了内置的super函数，并定义了方法解析顺序（method resolution order MRO）；MRO以标准的流程来安排超类之间的初始化顺序(深度优先，从左到右)，他保证钻石继承顶部公共基类的\__init\__方法只会运行一次；

```python
# python2
class TimesFiveCorrect(BaseClass):
    def __init__(self, value):
        super(TimesFiveCorrect, self).__init__(value)
        self._value *= value


# python3
class Implicit(BaseClass):
    def __init__(self, value):
        super().__init__(value*2)

# python3
class Explicit(BaseClass):
    def __init__(self, value):
        super(__class__, self).__init__(value * 2)
```

* 从上面可以看出，python2中super函数写法稍微麻烦；
