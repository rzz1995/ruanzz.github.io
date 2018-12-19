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
>1.String类有看过源码吗？和StringBuffer，StringBuilder有什么区别？

String类是一个final不可修改的类,说明String类不能被继承，StringBuffer和StringBuilder都是用来拼接字符串用的，只有调用toString()方法的时候才会真正的new String()对象，StringBuffer是线程安全的，StringBuilder是线程不安全的，因此StringBuilder的性能会更好一点，有并发场景的话还是使用StringBuffer来比较合适。

>2.Throwable、Error、Exception区别和联系。受查异常和非受查异常都用过哪些，谈一下对他们的理解

Throwable 是异常的顶层类，Error和Exception都继承了这个类，当然还有其他一些子类，在我们开发过程中常接触到也就是这两个类的子类，算是两种比较常用的异常处理。Error代表出现了非常严重的错误，JVM虚拟机无法处理，只能Crash，比如OutOfMemoryError。Exception是一般性的异常，这个一般性指的是不会造成虚拟机宕机，但是对开发人员来说，最复杂的也就是这块，对异常的处理是根据业务来的，是抛出去还是try起来，怎么对异常分类？这个都要根据具体的业务来决定，跟业务相关复杂性肯定不会小，因此如何处理好异常也是很有门道的。
Exception和Error本来是互不相关的，但是Exception中有一个叛徒RuntimeException，这个叛徒却是最受开发人员欢迎的，基本上自定义业务一样都会继承它，为什么说他是叛徒呢？它和Error一样，都是都是unchecked的，程序都会进行中止处理，除RuntimException及其子类外的Exception都是checked的，受查异常和非受查异常在在编译过程中就有体现出来，受查异常必须在代码层面做处理，不然编译不通过。常用的非受查异常有OutOfMemoryError,ClassCastException,NullPointerException等，常用的受查异常有：FileNotFoundException，NumberFormatException，NoSuchMethodException，IOException，ClassNotFoundException，InterruptedException等

### 1.2 集合
### 1.3 多线程
### 1.4 IO
### 1.5 JVM
## 2. MySQL数据库
## 3. 框架
## 4. 计算机网络
## 5. 算法
