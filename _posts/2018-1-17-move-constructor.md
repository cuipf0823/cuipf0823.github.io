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

#三. 引用限定符号
通常情况下, 我们使用一个对象调用成员函数, 这个对象无论是左值还是右值都可以; 

```c++
std::string s1 = "a value";
std::string s2 = "another value";
auto iter = (s1 + s2).find('r');
//无论新老标准库都是允许的, 编译通过;
s1 + s2 = "nani";
```
即使我们直接向右值赋值, 编译器也不会报错, 当然我们自己实现类的是时候不希望这样; 这个时候我们就可以使用引用限定符号来限制这种行为;

```c++
//只能向可修改的左值赋值
Moveable& operator=(const Moveable& mo) & 
{
	std::cout << "assignment" << std::endl;
	if (this != &mo)
	{
		i = new int(*mo.i);
	}
	return *this;
}
```

**说明:**
1. 引用限定符号既可以是"&"或者"&&",分别指出this可以指向一个左值和右值;类似于const;
2. 引用限定符号只能用于(非static)成员函数;
3. 引用限定符号必须同时出现在函数的声明和定义中;
4. 对于"&"的限定的函数, 只能将它用于左值;而"&&"的限定只能用于右值;
5. 一个函数可以同时用const和引用限定符号; 但引用限定符号必须在const后面;


## 重载和引用函数
和const一样,引用限定符号也可以用来区分重载版本;

如下:
```c++
class Foo
{
public:
	Foo sorted() &&;	  //可用于可改变的右值;
	Foo sorted() const &; //可用于任何类型的Foo;
private:
	std::vector<int> datas;
};

//本对象为右值, 因此可以原址排序
Foo Foo::sorted() &&
{
	std::sort(datas.begin(), datas.end());
	return *this;
}

//本对象是const或者是一个左值 不能对其进行原址排序
Foo Foo::sorted() const &
{
	Foo ret(*this);
	std::sort(ret.datas.begin(), ret.datas.end());
	return ret;
}
```
说明:
当我们对一个右值执行sorted时候, 可以安全直接对data成员进行排序.对象是右值的时候意味着不会有其他用户, 所以可以尽情修改; 对象是一个const右值或者一个左值, 则不能改变对象; 因此就需要在排序前拷贝datas;

#四. 其他
* 如果一个类中有可用的拷贝构造而没有移动构造, 则所有的移动操作都是通过拷贝构造来"移动"的; 拷贝运算符和移动运算符情况类似;
	
	```c++
	class Foo
	{
		Foo() = default;
		Foo(const Foo& foo); //拷贝构造
 	}
	Foo x;
	Foo y(std::move(x)); //这里调用的是拷贝构造函数;
	```
