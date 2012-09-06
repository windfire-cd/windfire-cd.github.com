---
layout: post
title: "c++的Traits技术"
date: 2012-09-06 13:58
comments: true
categories: c++
---

##简介

traits是一种特性萃取技术,它在Generic Programming中被广泛运用,常常被用于使不同的类型可以用于相同的操作,或者针对不同类型提供不同的实现
类型和类型的特性本是耦合在一起，通过traits技巧就可以将两者解耦。从某种意思上说traits方法也是对类型的特性做了泛化的工作，通过traits提供的类型特性是泛化的类型特性


##例子

###Example 1

traits在实现过程中往往需要用到以下三种C++的基本特性

- enum
- typedef
- template (partial) specialization

*enum*用于将在不同类型间变化的标示统一成一个,它在C++中常常被用于在类中替代*define*,你可以称*enum*为类中的*define*;
*typedef*则用于定义你的模板类支持特性的形式,你的模板类必须以某种形式支持某一特性,否则类型萃取器*traits*将无法正常工作.看到这里你可能会想,太苛刻了吧?
其实不然,不支持某种特性本身也是一种支持的方式(见示例2,我们定义了两种标示, *__xtrue_type*和 *__xfalse_type*,分别表示对某特性支持和不支持).
*template (partial) specialization*被用于提供针对特定类型的正确的或更合适的版本.
借助以上几种简单技术,我们可以利用*traits*提取类中定义的特性,并根据不同的特性提供不同的实现.你可以将从特性的定义到萃取,再到*traits*的实际使用统称为*traits*技术,但这种定义使得*traits*显得过于复杂,
我更愿意将traits的定义限于特性萃取,因为这种定义使得traits显得更简单,更易于理解

```cpp
#include <iostream>
using namespace std;

class CComplexObject // a demo class
{
	public:
		void clone() { cout << "in clone" << endl; }
};

// Solving the problem of choosing method to call by inner traits class
template <typename T, bool isClonable>
class XContainer
{
	public:
		enum {Clonable = isClonable};

		void clone(T* pObj)
		{
			Traits<isClonable>().clone(pObj);
		}

		template <bool flag>
			class Traits
			{
			};

		template <>
			class Traits<true>
			{
				public:
					void clone(T* pObj)
					{
						cout << "before cloning Clonable type" << endl;
						pObj->clone();
						cout << "after cloning Clonable type" << endl;
					}
			};

		template <>
			class Traits<false>
			{
				public:
					void clone(T* pObj)
					{
						cout << "cloning non Clonable type" << endl;
					}
			};
};

void main()
{
	int* p1 = 0;
	CComplexObject* p2 = 0;

	XContainer<int, false> n1;
	XContainer<CComplexObject, true> n2;

	n1.clone(p1);
	n2.clone(p2);
}
```

输出：

doing something non Clonable
before doing something Clonable
in clone
after doing something Clonable


[Traits初探](http://www.cppblog.com/woaidongmao/archive/2008/11/09/66387.html)


###Example 2

{%img http://p.blog.csdn.net/images/p_blog_csdn_net/sanlongcai/333954/o_type_traits.JPG %}

从图中可看出算法destroy不必关心具体的类型特性traits，client不用关心具体的destroy。destroy概念上存在的基类是通过参数多态实现的，traits概念上存在的基类是通过*type_traits*编程方法实现的。
另外得注意的是STL中的iterator相关*type_traits*的使用跟这里所说的有点不同，如果把类型特性从类中剥离出来看待，那就完全相同了。如何剥离，遇到*type_traits*相关的含有类型特性的类只看成是类型特性，跟类型特性无关的全都忽略掉

```cpp my_type_traits.h
#ifndef MY_TYPE_TRAITS_H
#define MY_TYPE_TRAITS_H

struct my_true_type {
};

struct my_false_type {
};

template <class T>
struct my_type_traits
{
	typedef my_false_type has_trivial_destructor;
};

template<> struct my_type_traits<int>
{
	typedef my_true_type has_trivial_destructor;
};

#endif
```

```cpp my_destruct.h
#ifndef MY_DESTRUCT_H
#define MY_DESTRUCT_H
#include <iostream>

#include "my_type_traits.h"

using std::cout;
using std::endl;

	template <class T1, class T2>
inline void myconstruct(T1 *p, const T2& value)
{
	new (p) T1(value);
}

	template <class T>
inline void mydestroy(T *p)
{
	typedef typename my_type_traits<T>::has_trivial_destructor trivial_destructor;
	_mydestroy(p, trivial_destructor());
}

	template <class T>
inline void _mydestroy(T *p, my_true_type)
{
	cout << " do the trivial destructor " << endl;
}

	template <class T>
inline void _mydestroy(T *p, my_false_type)
{
	cout << " do the real destructor " << endl;
	p->~T();
}

#endif
```

```cpp  test_type_traits.cpp
#include <iostream>
#include "my_destruct.h"

using std::cout;
using std::endl;

class TestClass
{
	public:
		TestClass()
		{
			cout << "TestClass constructor call" << endl;
			data = new int(3);
		}
		TestClass(const TestClass& test_class)
		{
			cout << "TestClass copy constructor call. copy data:"
				<< *(test_class.data) << endl;
			data = new int;
			*data = *(test_class.data) * 2;
		}
		~TestClass()
		{
			cout << "TestClass destructor call. delete the data:" << *data << endl;
			delete data;
		}
	private:
		int *data;
};

int main(void)
{
	{
		TestClass *test_class_buf;
		TestClass test_class;

		test_class_buf = (TestClass *)malloc(sizeof(TestClass));
		myconstruct(test_class_buf, test_class);
		mydestroy(test_class_buf);
		free(test_class_buf);
	}

	{
		int *int_p;
		int_p = new int;
		mydestroy(int_p);
		free(int_p);
	}
}
```

[type traits 之"本质论"](http://blog.csdn.net/sanlongcai/article/details/1786647)


###Example 3

首先假如有以下一个泛型的迭代器类，其中类型参数 T 为迭代器所指向的类型

```cpp
template <typename T>
class myIterator
{
	 ...
};
```

当我们使用myIterator时，怎样才能获知它所指向的元素的类型呢？我们可以为这个类加入一个内嵌类型，像这样

```cpp
template <typename T>
class myIterator
{
	      typedef  T value_type; 
		  ...
};
```

这样当我们使用myIterator类型时，可以通过 myIterator::value_type来获得相应的myIterator所指向的类型。

现在我们来设计一个算法，使用这个信息。

```cpp 
template <typename T>
typename myIterator<T>::value_type Foo(myIterator<T> i)
{
	 ...
}
```

这里我们定义了一个函数Foo，它的返回为为  参数i 所指向的类型，也就是T，那么我们为什么还要兴师动众的使用那个value_type呢？ 那是因为，当我们希望修改Foo函数，使它能够适应所有类型的迭代器时，我们可以这样写：

```cpp
template <typename I> //这里的I可以是任意类型的迭代器
typename I::value_type Foo(I i)
{
	 ...
}
```

现在，任意定义了 value_type内嵌类型的迭代器都可以做为Foo的参数了，并且Foo的返回值的类型将与相应迭代器所指的元素的类型一致。至此一切问题似乎都已解决，我们并没有使用任何特殊的技术。然而当考虑到以下情况时，新的问题便显现出来了：

原生指针也完全可以做为迭代器来使用，然而我们显然没有办法为原生指针添加一个value_type的内嵌类型，如此一来我们的Foo()函数就不能适用原生指针了，这不能不说是一大缺憾。那么有什么办法可以解决这个问题呢？ 此时便是我们的主角：类型信息榨取机 Traits 登场的时候了

```cpp
template <typename T>
class Traits
{
	      typedef typename T::value_type value_type;
};
```

```cpp
template <typename I> //这里的I可以是任意类型的迭代器
typename Traits<I>::value_type Foo(I i)
{
	 ...
}
```

偏特化原生指针

```cpp
template <typename T>
class Traits<T*> //注意 这里针对原生指针进行了偏特化
{
	      typedef typename T value_type;
};
```

```cpp test.cpp
int * p;
....
int i = Foo(p);
```

[C++ Traits](http://www.cnblogs.com/Hush/archive/2004/03/10/2717.html)

##参考

[C++之traits小记](http://www.pleee.com/archives/277.html)
