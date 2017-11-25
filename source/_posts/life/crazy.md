---
title: Crazy?
date: 2017-08-20 16:46:25
updated: 2017-11-25 09:40:00
categories: 学习
tags: [JS, ES6, JAVA]
---
有点迷茫
<!--more-->
最近感觉有点乱，在做管理后台的东西，开始大量接触`java`， 之前有动摇过到底是继续专注于前端，还是说也花一点功力在`java`，身边有支持前者的，也有同意后者的，然后自己做了决定开始学习`java`。

大学的时候有学过`java`， 一个学期学的`java`，第二个学期学的`JSF`和`EJB`，但是学习的并不好，现在感觉全都忘完了，只知道点渣渣了。

找大家借了几本书：《JAVA 核心技术 卷1》，《Effective java》，还有《修炼之道》，还有《Spring In Action》等好多本书等着我去借，我了个去啊，感觉自己的java学习之路，一定不轻松，*出来混，迟早是要还的*。

然后在接下来的近一个月的时间里，一直在学习java,但是又觉得，前端也不能停下来，那就一边学java，一边也继续搞前端，还好现在的工作内容是，前后端都要写的，似乎专门是为我准备的咯。但是工作稍微有点忙，自从办了新办公室，很少有准点下班的情况，要么工作，要么学习，当然工作似乎占了大部分。

java的，学习计划，打算采用扫盲的方式，不管以前学过什么，现在都从最基础的看起，防止自己遗漏一些细节，刚开始看得还算比较轻松，内心活动：毕竟还是学过的！但是后来就发现自己是too young too simple，最近看到了两个知识点：*反射*和*内部类*。。。

反射还算是有点印象，`getClass`嘛， 但是其他的就没有了，更别提怎么用了。

```java
Employee e;
...
System.out.println(e.getClass.getName+"  "+e.getName());
```
这里不同的子类就可以打印出不同的类名了，如果在包里，还可以打印出包名。

```java
public static void main(String[]agrs){
  Random g = new Random();
  Class cl = g.getClass();
  String name = cl.getName();//java.util.Random
  g.getClass().newInstance();//动态创建实例
  g.getClass() == Ramdom.class //比较
}
```
获取类名还有其他几种不同的方法.

```java

String className = "java.util.Random";
Class cl = Class.forName(className);
```
```java
Class cl1 = Random.class;
Class cl2 = int.class; // int
Class cl3 = Double[].class; //[Ljava.lang.Double;
```
反射能力最重要的功能——检查类的结构。

在java.lang.reflect包中有三个类，`Field`、`Method`、`Constructor`，分别用于描述类的域、方法和构造器。他们都有`getName`方法，返回名称。域还有个`getType`方法，返回域所属对象类型的`Class`对象，`Method`和`Constructor`能够报告参数类型的方法，`metho`d还有个报告返回类型的方法。这三个类还有个叫`getModified`的方法，返回一个整型数组，用不同的位开关描述`public` 、`static`等修饰符的使用状况。
还有`Modifier`类提供一些方法。

···················
写了点写不下去了，有时间再说吧。。。

暂时的解决方案就是，以前端为主，前后端学习时间分布6-4或7-3开吧，嗯，就酱。