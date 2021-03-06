---
layout: article-detail
title:  "一次排查MYSQL索引失效的过程"
author: "郭清存"
date:   2018-03-30 10:16:17 +0800
categories: 数据库
location: 
description: 
---
---



# 背景


由于一次系统升级，造成生产系统中订单商品金额记录错误。账务在计算销售成本时出现差额。订单商品表记录270W，则需要修复3K条记录。


# 修复过程中的问题

在生产环境修复一条记录花费时间大概7秒.这个速度远远超出我们的想象，慢的出奇。

# 排查

发现订单商品金额记录错误后，按排江勇（同事）处理此问题。同事很快的定位到问题，且给出了使用存储过程做为解决方案。但执行存储过程中发现速度出奇的慢，修复一条记录的时间大概7秒钟。如此这般，仅3K，则需要花费5个小时的时间。这是不可接受的。

但又百思不解，在where中的字段都有索引，为什么还会如此的慢。经过多轮尝试发现一个奇怪的现象，修复一条数据时做了大量的表扫码(Handler_read_rnd_next的数值210W)。这也说明在where中的条件未使用到索引。
小试了一把，将where条件中的变量@order_code由固定字符串替换后，Handler_read_rnd_next的数值为3.修复记录仅用了几十毫秒。这就断定问题是由变量@order_code引起的.初次断定是由于@order_code的编码问题导致。江勇通过select character(@order_code)得到的结果是:utf8mb4,而表中的字段编码是utf8.原来如此.

# 解决方案

把@order_code转为utf8编码

SELECT CONVERT(@order_code, CHAR CHARACTER SET utf8) INTO temp;

搞定

# 小结

SQL无法走索引常见的有如下9种情况：

- 统计信息不准确
- 索引列的值允许为NULL 
- 谓词使用了不等于（<>, !=） 
- LIKE前通配或全通配的查询 
- 索引列使用了函数、数学运算、其它表达式等 
- 使用了隐式类型转换 
- 查询转换失败 
- 左右字段编码不同
- 其它语句逻辑原因

![王江勇](/images/people/jiangyong.jpg "王江勇")

>
  <small>本文总阅读量<span id="busuanzi_value_page_pv"></span>次</small>
