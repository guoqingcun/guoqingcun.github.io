---
layout: post
title:  "LinkedList remove方法陷阱"
date:   2018-04-02 16:12:17 +0800
categories: Java
location: 
description: 
---
---



## LinkedList定义


LinkedList是实现list和Deque接口的双向链表且非线程安全。LinkedList应该在什么场景下使用LinkedList呢?

熟悉双向链表的同学,首先想到的应该是随机增删等操作。没错，随机增删是双向链表所擅长的操作，但LinkedList果真如些吗？


## remove方法时间复杂度O(1)?

{% highlight java linenos %}

    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
{% endhighlight %}

从代码上我们可以对O(1)说NO，显然，LinkedList.remove(Object)方法的时间复杂度是O(n)+O(1)。最坏情况下要删除的元素是最后一个，你都要比较N-1次才能找到要删除的元素。

## 存在同样遍历的问题的方法
  {% highlight java linenos %}
  public int indexOf(Object o)  
  public void add(int index, E element)
  {% endhighlight %}  