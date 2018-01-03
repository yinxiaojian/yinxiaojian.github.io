---
title: java类与成员访问控制
date: 2017-10-27 15:22:29
tags: java基础
categories: java
---

java访问控制是基础中的基础，有public/private/protected/default四个类型，一般分为类的访问控制和成员访问控制两个类型。
<!-- more -->

### 修饰类

修饰类只能使用public和default，不可以声明为protected或private。用public修饰的类任何情况下都可以访问。用default即不加任何修饰词，权限为包访问权限，在同一个包内的类可以访问。

在修饰类的时候有以下几点需要注意

* 每个编译单元（文件）只能有一个public类，如果有一个以上的public类，编译器会报错
* 实际上类可以既不是public也可以不失default，这涉及到内部类，此处不介绍。

### 修饰成员

| 权限修饰符     | 同类   | 同包   | 不同包的子类 | 不同包的非子类 |
| --------- | ---- | ---- | ------ | ------- |
| public    | Y    | Y    | Y      | Y       |
| protected | Y    | Y    | Y      | N       |
| default   | Y    | Y    | N      | N       |
| private   | Y    | N    | N      | N       |

### 有趣的类比

public：全世界共享
default：只属于中国这个国家，权限收缩
protected：属于中国这个国家，当然不在中国的国人也有权使用
private：只属于单一的中国人



