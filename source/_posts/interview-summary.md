---
title: Java后端面试题整理
date: 2018-12-18 21:47:55
tags: [interview,Java]
toc: true
---
![惠灵顿](/asset/img/interview-summary/Mac.jpeg)

<!-- more -->
这个冬天有点冷，冬天了还是储藏点过冬的粮食，用倒推法来总结相关知识，这篇博客将会长期更新，收集网上的面试题，然后自己整理思路，以后自己肯定会用的上，也为以后博客积累素材。

## 1. Java基础
### 1.1 语法糖
>1. String类有看过源码吗？和StringBuffer，StringBuilder有什么区别？

String类是一个final不可修改的类,说明String类不能被继承，StringBuffer和StringBuilder都是用来拼接字符串用的，只有调用toString()方法的时候才会真正的new String()对象，StringBuffer是线程安全的，StringBuilder是线程不安全的，因此StringBuilder的性能会更好一点，有并发场景的话还是使用StringBuffer来比较合适。
### 1.2 集合
### 1.3 多线程
### 1.4 IO
## 2. MySQL数据库
## 3. 框架
## 4. 计算机网络
## 5. 算法
