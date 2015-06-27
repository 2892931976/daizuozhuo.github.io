---
layout: post
title: C++11 读书笔记
---

###左值和右值
c++的表达式中，左值可以放在赋值语句的左侧，而右值则不能。
通过“&&”来获得右值引用，右值引用只能绑定到一个将要销毁的对象。
使用move函数可以得到一个左值的右值引用，然后可以直接将其中的资源拿出来而不是重新拷贝一份。

```  
  std::string foo = "foo-string";
  std::string bar = "bar-string";
  std::vector<std::string> myvector;

  myvector.push_back (foo);                    // copies
  myvector.push_back (std::move(bar));         // moves
```
###非类型参数模板
模板可以定义类型参数（type parameter），还可以定义非类型参数(nontype parameter).一个非类型参数表示一个值而不是类型。比如：

```
template<unsigned N, unsigned M>
int compare(const char(&p1)[N], const char (&p2)[M]) {
	return strcmp(p1, p2);
}
```

###可变参数模板

```
template<typename...Args> void g(Args...args) {
	cout << sizeof...(Args) << endl;
	cout << sizeof...(args) << endl;
}
```

###类模版的特例化
例如可以为hash特例化自定义class的模板。

```
template<>
struct hash<Sales_data> {
	typedef size_t result_type;
	typedef Sales_data argument_type;
	size_t operator()(const Sales_data& s)const;
}
```

###类模版的部分特例化
我们只能部分特例化模板，而不能部分特例化函数模板。

应用场景1:部分特例化引用

```
template<class T> struct remove_reference {
	typedef T type;
};
template<class T> struct remove_reference<T&> {
	typedef T type;
};
template<class T> struct remove_reference<T&&> {
	typedef T type;
};
```
应用场景2：部分特例化子类

```
template <class T2>
struct Goo
{
    void goo(T2 v)
    {
    }
};
 
template <class T1, class T2>
struct Foo : Goo<T2>
{
    void foo(T1 v)
    {
    }
};
 
template <class T2>
struct Foo<int, T2> : Goo<T2>
{
    void foo(int v)
    {
    }
};
```
		
