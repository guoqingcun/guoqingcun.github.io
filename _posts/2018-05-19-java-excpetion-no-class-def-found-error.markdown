---
layout: article-detail
title:  "NoClassDefFoundError与ClassNotFoundException两者的不同"
author: "郭清存"
date:   2018-05-19 11:06:17 +0800
categories: java
location: 
description: "不能找到类定义"
---
---

### NoClassDefFoundError
首先看看NoClassDefFoundError的继承关系。

<pre>
 java.lang.Object
      java.lang.Throwable
         java.lang.Error
            java.lang.LinkageError
                java.lang.NoClassDefFoundError
</pre>

了解NoClassDefFoundError之前，先看看它的大佬,老大及其自身的含义.

- Error:所谓的Error及其子类，代表的是不期望恢复的错误。
- LinkageError:LinkageError及其子类，代表编译时OK，之后被做了改动。
- NoClassDefFoundError:Java虚拟机或类加载器试着加载类定义时，未能找到类定义。这种错误是由于编译通过后，但运行时未能找到类定义导致。

从它的继承关系上看，可以得知NoClassDefFoundError是一个不能被恢复错误，究其原因是编译后的字节码加载时找不到类定义。

### ClassNotFoundException
<pre>
 java.lang.Object
     java.lang.Throwable
         java.lang.Exception
             java.lang.ReflectiveOperationException
                 java.lang.ClassNotFoundException
</pre>
同上,先看它的大佬,老大及其自身的含义
- Exception:所谓的Exception及其子类，代表的是期望恢复的异常。
- ReflectiveOperationException：反射操作时引起的异常
- ClassNotFoundException:调用Class.forname(),ClassLoader.findSystemClass(),ClassLoader.loadClass()方法时，加载指定的字符串类名，未找到类时抛出此类异常。

从继承关系可知，ClassNotFoundException是一个执行反射操作时引起且期望恢复的异常。


>
  <small>本文总阅读量<span id="busuanzi_value_page_pv"></span>次</small>
