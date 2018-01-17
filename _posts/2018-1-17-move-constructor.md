---
# layout: post
title: 类移动操作注意事项
category: C++11 move
---
# 一. 移动操作和异常
## 问题
先看代码:

```c++
class Moveable
{
public:
	Moveable() :i(new int(2))
	{
		std::cout << "constructor." << std::endl;
	}
	~Moveable()
	{
		delete i;
		std::cout << "destructor." << std::endl;
	}
	Moveable(const Moveable& o)
	{
		i = new int(*o.i);
		std::cout << "copy constructor." << std::endl;
	}
	Moveable(Moveable&& mo)
	{
		i = mo.i;
		mo.i = nullptr;
		std::cout << "move copy constructor." << std::endl;
	}
	Moveable& operator=(Moveable&& mo)
	{
		std::cout << "move assignment" << std::endl;
		if (this != &mo)
		{
			i = mo.i;
			mo.i = nullptr;
		}
		return *this;
	}
	Moveable& operator=(const Moveable& mo)
	{
		std::cout << "assignment" << std::endl;
		if (this != &mo)
		{
			i = new int(*mo.i);
		}
		return *this;
	}
public:
	int* i;
};

void test()
{
	std::vector<Moveable> vec_mo;
	vec_mo.reserve(2);
	for (int idx = 0; idx < 3; ++idx)
	{
		Moveable able;
		vec_mo.push_back(std::move(able));
	}
}
```
那么上面的test输出是什么? 我们知道vector预留空间不足时候, 会重新申请空间把老空间的对象一一拷贝过来; 此时,有移动拷贝构造函数也有拷贝构造函数, 会调用哪一个?

上面代码输出为:

```
constructor.
move copy constructor.
destructor.
constructor.
move copy constructor.
destructor.
constructor.
move copy constructor.
copy constructor.
copy constructor.
destructor.
destructor.
destructor.
```
我们可以看到, 并没有用到我们实现的移动拷贝构造函数; 为什么?

## 原因
* 标准容器需要对发生异常时候,提供自身的行为提供保证;

	我们知道移动一个对象时候会修改它的值, 想象一下,如果vector内存不足,申请了新的内存, 正在进行对象的移动, 部分发生了异常, 此时老的数据也已经被修改了,新空间需要构造的元素也未完成, 此时vector算是废了; 
	而使用拷贝构造就不会出现这个不可逆的问题;

* 移动操作不会抛出异常, 但需要让标准容器知道;
	
	我们都知道,移动构造是"窃取"资源, 通常不会分配资源;因此移动操作不会抛出任何的异常,当编写一个不抛出异常的移动操作时候, 我们需要让标准容器库知道, 否则它会认为调用我们移动操作移动我们对象时候会抛出异常; 自然也不会调用我们的移动操作了;

## 解决方式

```c++
	Moveable(Moveable&& mo) noexcept
	{
		i = mo.i;
		mo.i = nullptr;
		std::cout << "move copy constructor." << std::endl;
	}
	Moveable& operator=(Moveable&& mo) noexcept
	{
		std::cout << "move assignment" << std::endl;
		if (this != &mo)
		{
			i = mo.i;
			mo.i = nullptr;
		}
		return *this;
	}
```

告诉标准容器库, 我们的移动操作是不会抛出异常的, 只需要在移动操作函数后面加上**noexcept**即可;
那么上面的测试test函数输出就不一样了! 如下:

```
constructor.
move copy constructor.
destructor.
constructor.
move copy constructor.
destructor.
constructor.
move copy constructor.
move copy constructor.
move copy constructor.
destructor.
destructor.
destructor.
```
如上, 在vector中内存不足申请新的内存后, 原来的拷贝构造全部换成了移动拷贝构造, 效率得到很大的提高; 

# 二. 默认的移动操作
* 和默认生成拷贝构造操作不一样, 在大多数情况下, 类不会帮我们生成默认的移动操作; 这时候一般使用拷贝操作代替移动操作;
* 当一个类没有定义任何自己的拷贝控制成员函数, 且类中每一个非static的数据成员都可以移动时候, 编译器才会为我们合成移动构造函数和移动赋值运算符; 除此之外都不会生成默认的移动操作;
	
	```c++

	//编译器会生成默认的移动操作
	struct X 
	{
		int d;
		std::string s;
	}
	
	```
* 如果我们显式的要求编译器生成"=default"移动操作; 且编译器不能移动所有成员, 则编译器会将移动操作定义为删除的函数;
* 定义了移动构造或者移动赋值的类, 必须自己定义拷贝操作, 否者默认生成的拷贝构造和拷贝赋值就会被定义为删除;
