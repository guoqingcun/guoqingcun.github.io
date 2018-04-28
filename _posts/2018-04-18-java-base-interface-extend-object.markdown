---
layout: post
title:  "interface是否继承自Object对象"
date:   2018-04-28 17:00:17 +0800
categories: 公司
location: 
description: 创业公司的架构
---
---

### interface是否继承自Object对象?
    答案:NO，接口和类没有公共的根元素
	
### 为什么interface可以调用object中的方法？
    答案:接口中隐式的实现了object中的方法.
	
The members of an interface are:

- Those members declared in the interface.
- Those members inherited from direct superinterfaces.
- If an interface has no direct superinterfaces, then the interface implicitly declares a public abstract member method m with signature s, return type r, and throws clause t corresponding to each public instance method m with signature s, return type r, and throws clause t declared in Object, unless a method with the same signature, same return type, and a compatible throws clause is explicitly declared by the interface.

### 从字节码上看是存在歧义

举例说明，写一个空类，空接口，如下

空类

``` java

public class TestC {

}

```

字节码如下

<div align="center">
<img src="/images/java/base/interfaceAndClass/TestC.jpg" title="空类字节码分析"/>
</div>
<br>

空接口

``` java

public class TestI {

}

```

字节码如下

<div align="center">
<img src="/images/java/base/interfaceAndClass/TestI.jpg" title="空接口字节码分析"/>
</div>
<br>

其中对于super_class的解析如下

<b>For a class</b>, the value of the super_class item either must be zero or
must be a valid index into the constant_pool table. If the value of the
super_class item is nonzero, the constant_pool entry at that index must
be a CONSTANT_Class_info structure representing the direct superclass of the
class defined by this class file. Neither the direct superclass nor any of its
superclasses may have the ACC_FINAL flag set in the access_flags item of its
ClassFile structure.
If the value of the super_class item is zero, then this class file must represent
the class Object, the only class or interface without a direct superclass.

<b>For an interface</b>, the value of the super_class item must always be a valid
index into the constant_pool table. The constant_pool entry at that index
must be a CONSTANT_Class_info structure representing the class Object.

也就是说super_class并不能说明interface继承自Object


   




   















