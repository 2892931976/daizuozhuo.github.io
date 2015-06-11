---
layout: post
title: Type conversions in C++类型转换
---

Implicit conversions隐式转换

* 可以在基本类型之间自由转换；
* 可以把任何类型的pointer转换为void pointer；
* 可以将子类pointer转换为基类pointer;
* 会产生warning

static_cast静态转换

* 类似于implicit cast
* 可以在子类基类pointer之间自由转换
* 没有运行时检查

dynamic_cast动态转换

* 一般用于polymorphism
* 会检查pointer是否指向valid object
* reinterpret_cast
* 可以在任意类型pointer之间进行转换
* 没有检查，非常危险

const_cast

* 可以在const 和 non const pointer 之间转换

C style cast

* (type) val 或 type(val), 按照以下优先顺序转换：

const_cast

* static_cast (though ignoring access restrictions
* static_cast (see above), then const_cast
* reinterpret_cast
* reinterpret_cast, then const_cast