---
title: 记录一次需求开发的坑（表格列总计新增计算方式）
date: 2019-05-29 11:49:03
tags:
---


开发两个自定义聚合函数 MAX_TO_VALUE OR MIN_TO_VALUE

## Problem 1: UDAF依赖class不对的问题

开发完成联调时发现执行SQL时，旧函数BOL指向的Java class不对

### 定位以及解决

Scala的Server程序会在启动时注册一遍所有UDF和UDAF，对于已经create过的函数将不会重复register，而开发中，发现BOL这个之前的函数对应的class命名不规范，修改了之前的class类名，而增加的MAX_TO_VALUE对应的class则用了之前函数的class，初始化时，BOL已经被创建了，指向的class不变，而由于实际上class名称已经被修改，所以出现了指向错误问题。

解决: 在Hive中删除之前已注册过的函数，重启Server，重新注册UDF后解决

## Problem 2: 新增UDAF运行结果一直为Null

PostMan模拟请求时，MAX_TO_VALUE结果一直是空

### 定位以及解决

Debug模式运行Server，断点调试发现UDAF在iterate和terminatePartial完成后，merge的时候传入的数据结构SimpleNormalValue的outputVal一直是null，而在iterate阶段是确认有值的，后续跟踪断点发现terminatePartial->merge的过程中，hive使用了Java反射去传递SimpleNormalValue这个类的结构，而SimpleNormalValue是从SimpleValue继承过来的，父类中的protected类型的变量outputVal对于子类是private类型，反射是获取不到private的变量

解决: 声明从protected变为public，或者不继承outputVal而是重新声明子类的outputVal

## Problem 3: 含对比时计算结果不对

有对比维度时，表格统计的值预期是(3,null,null)，实际为(3,3,3)

### 定位以及解决
包含对比时，查询的原始表的维度中包含了对比维度，另外结果集是用临时表做if判断然后再聚合的结果，因此实际表数据中没有的空记录，会被展示到查询结果里面，而列总计统计的是原始表的数据，而非结果数据或临时表数据（因为要统计limit之前的数据），另外对原始数据的维度idx计算也不准确

解决: 修复原始数据维度idx偏移的问题

TODO: 后续把包含对比列总计的逻辑，从直接对原始表进行GROUP SET分类聚合，改造为对对比拆分后的临时表进行GROUP SET分类聚合，改造工作大致有几点：1.临时表的维度不包含对比，因此GROUP SET需要包含空，且一个数值要根据xy_fields结构拆分成多列，同理在处理列总计查询到的数据时，之前的分类聚合是对不同对比做拆分的，而后续结果是要根据数值idx在拆分; 2.包含TOP的时候是Join了结果表和原始表，需要改造为Join结果表和临时表; 3.包含高级计算（同环比）时，注意兼容之前为了处理高级计算列总计的相关开发