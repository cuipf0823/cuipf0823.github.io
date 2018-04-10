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



