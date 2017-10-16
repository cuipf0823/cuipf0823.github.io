# boost::implicit_cast
## 问题
先看代码:

```c++
struct top 
{

};
struct mid_a : top 
{

};
struct mid_b : top 
{

};
struct bottom : mid_a, mid_b 
{
	
};
void foo(mid_a&) 
{
	
}
void foo(mid_b&) 
{
	
}

void bar(bottom &arg) 
{    
    //想要调用"void foo(mid_a&)"
	foo(arg); 
}

int main() 
{    
	bottom x;    
	bar(x);    
	return 0;
}
```

此时, 代码是无法通过编译, 因为foo存在重载解析的歧义;

## 解决方法
ok, 现在把bar的代码修改为C++风格, 而不是使用C风格的转换:

```c++
void bar(bottom &arg) 
{    
    //想要调用"void foo(mid_a&)"
    foo(static_cast<mid_a&>(arg)); 
}
```

现在一点问题没有, 可以通过编译, 可以等到正确的答案;

## 另一个问题
上面的代码看似没有任何问题, 然而.....过了一段时间上面代码修改如下:

```c++
//top, mid_a, mid_b声明都不变
//仅仅修改了如下部分
void bar(top &arg)
{
    //很长时间之后, 这里已经添加了很多代码.
    foo(static_cast<mid_a&>(arg));
}

int main() 
{
    //由原来bottom x 变为top x
    top x;
    bar(x);
    return 0;
}
```

此时, 代码可以编译通过, 但是, 一运行程序就挂掉了, why ?

**问题分析:**

原因就在于static_cast太强大了, 强大到可以进行”down-cast”. 于是编译器没有给你任何警告, 就把一个top类型的引用给强制转换成了min_a的引用, 这是不安全的; 

这个时候轮到boost::implicit_cast出场了;

## 优化解决方法

```c++
void bar(top &arg)
{
    //很长时间之后, 这里已经添加了很多代码.
    foo(boost::implicit_cast<mid_a&>(arg));
}
```
把bar里面的那句foo调用修改为上述, 于是问题就解决了, 再出现"down-cast", 代码就会编译不通过; 

原因在于隐式类型转换不允许”down-cast”, 只能”up-cast”.


## 分析

首先,这里说一下显式和隐式: 

* 在C++世界的英文里, 我们说”convert”通常指”implicit convert”, 而”cast”指”explicit cast”; 
* 隐式类型转换好理解, 就是你写了个a=b, 而ab不同类型, 编译又不报错, 就说明隐式类型转换发生了, 类似的情况还有在函数调用的参数传递时; 
* 而显式类型转换特指C 风格的强制转换 ((type)obj 或者C++ 中等价的 type(obj)), 以及C++风格的四个关键字(static_cast, const_cast, dynamic_cast, reinterpret_cast); 
    > 然而这个定义是相当模糊的, 比如一个int类型的x, bool(x)是显式的, 而!!x是隐式的, 其实效果上并没有区别, 只是字面上的不同罢了.


所以在bar里我们需要的仅仅是一个隐式类型转换, 然而直接把arg传递给foo的话会出现重载歧义, 于是我们需要告诉编译器到底要进行哪个隐式类型转换. 然而static_cast又太过强大, 它还能做隐式类型转换之外的事情(up-cast), 于是在日后代码演化的过程中留下了bug.

于是boost::implicit_cast应运而生, 它比static_cast弱, 正如它的名字一样, 它只能用来告诉编译器执行什么隐式类型转换.

## 源码分析
简单到令人发指:

```c++
template<typename T> 
struct identity 
{ 
    typedef T type; 
};

template <typename T>
inline T implicit_cast (typename mpl::identity<T>::type x) 
{
    return x;
}
```

这个identity很重要; 如果没有这个identity, 像**implicit_cast(obj)**这样的代码也能通过编译, 然而它其实什么也没做, obj的类型仍然没变. identity的存在使得函数模板的参数类型推导失效, 因为要推导出T, 首先得知道identity是什么, 而identity又是依赖于T的. 于是就形成了循环依赖, 参数类型推导就失效了. 于是编译器就要求你显式地指定T的类型.
