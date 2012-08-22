---
layout: post
title: "切分字符串"
date: 2012-08-22 17:31
comments: true
categories: 
---

##切分字符串

对于切分一个字符串的问题，是比较基础而且经常遇到的问题，下面就3种语言的实现方式
做一个比较

##C++

c++的方式，包括c没有在标准库中提供这样的一个函数。因此有很多第三方库或者
灵活运用标准库的方法


```c
#include <boost/algorithm/string.hpp>

using namespace boost;

int main()
{
	string s = "a,b, c ,,e,f,";
	vector <string> fields;
	split( fields, s, is_any_of( "," ) );
	}

```

Boost还支持正则表达式的方式切分字符串

```c
string s = "one->two->thirty-four";
vector <string> fields;
split_regex( fields, s, regex( "->" ) );
```

同时Boost Tokenizer库也可以完成这种工作。具体参见[ Boost.Tokenizer](http://www.boost.org/libs/tokenizer/index.html)

###QT

QT中的[QString](http://developer.qt.nokia.com/doc/qstring.html)可以完成unicode方式的解析
具体参见[ QString::split()](http://developer.qt.nokia.com/doc/qstring.html#split)

###GNU

在glib中有相应的切分函数

- [*g_strsplit()*](http://developer.gnome.org/glib/stable/glib-String-Utility-Functions.html#g-strsplit)
- [*g_strsplit_set()*](http://developer.gnome.org/glib/stable/glib-String-Utility-Functions.html#g-strsplit-set)

```c
const char* s = ",,three,,five,,";
char** fields = g_strsplit( s, ',', 0 );
for (char** field = fields; field; ++field, ++n)
{
	printf( "\"%s\"\n", *field );
}
g_strfreev( fields );
fields = NULL;
```
###stl::iostream

可以用std::getline()函数根据不同的切分符来分割行

```c
string s = "string, to, split";
istringstream ss( s );
while (!ss.eof())
{
	  string x;               // here's a nice, empty string
	    getline( ss, x, ',' );  // try to read the next field into it
		  cout << x << endl;      // print it out, even if we already hit EOF
}
```
###std::string

可以利用std::string中的*find_first_of()*函数循环处理字符串

```c
string s = "string, to, split";
string delimiters = " ,";
size_t current;
size_t next = -1;
do
{
	  current = next + 1;
	    next = s.find_first_of( delimiters, current );
		  cout << s.substr( current, next - current ) << endl;
}
while (next != string::npos);
```

另外还有其他非主流方式，参见参考1

##Java

相对c/c++来说，java的标准库要强大很多，比如split函数就是string里面的

```java
string s=abcdeabcdeabcde;
string[] sArray=s.Split('c') ;
foreach(string i in sArray)
Console.WriteLine(i.ToString());
```

还可以按照字符串分割

```java 
string s="abcdeabcdeabcde";
string[] sArray1=s.Split(new char[3]{'c','d','e'}) ;
foreach(string i in sArray1)
Console.WriteLine(i.ToString());
```

正则表达式也是支持的

```java 
string content=agcsmallmacsmallgggsmallytx;
string[]resultString=Regex.Split(content,small,RegexOptions.IgnoreCase)
foreach(string i in resultString)
Console.WriteLine(i.ToString());
```

##Python

类似java，python的字符串分割也是内置的函数

```py

str = 'a,b,c,d'
strlist = str.split(',')	# 用逗号分割str字符串，并保存到列表
```


##参考

[Split a string](http://www.cplusplus.com/faq/sequences/strings/split/)
[Splitting a string in C++](http://stackoverflow.com/questions/236129/splitting-a-string-in-c)
[python string](http://docs.python.org/library/string.html)
