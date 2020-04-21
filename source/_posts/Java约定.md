---
layout: post
title: Java开发约定
tags: 约定
category: Java
description: 定义一些Java开发常见的约定，统一开发人员的编码
---

# Java约定

[TOC]

### 必须遵循Maven项目的基本约定

### 常规

* 方法名，简短并准确表达语义，几乎所有的方法名，都应该是**动名词形式**。
* 方法参数，2-5个可以不用封装成对象。超过5个，考虑封装成对象。

### DAO层

* DAO层的接口名，以DAO或Mapper结尾。
* 遵循CRUD的命名规则，selectByXX，insert，update，delete。
* 添加或删除外键（关联关系）时，方法命名：insertXXRelation，deleteXXRelation
* 额外允许的命名：countXX，sumXX等
* 不允许的命名：validateXX
* 用户DAO，它的列表、分页方法永远不要查询出密码。提供单独的一个接口查询密码，仅仅在校验的时候使用。

### Service层

* Service层接口名，以Service结尾。
* 命名方式：getByXX, getPageByXX，save，update, delete等；按当前业务类名称可以省略方法名，如`UserService`的`getUserById`方法可以省略为`getById`。
* 其他命名方式：add/remove，bind/unbind，validateXX等
* 尽量不要给关联关系表，建立Service接口，可以将其功能移到具体的业务对象的服务类中。

### Controller层

* 类名以Controller结尾
* 方法命名参考Service层
* 方法长度尽量简短，可以写一些提取参数的代码段，具体业务转到Service处理。
* 永远不要将Request、Response、Session对象传递到Service层。

### Entity层

* 实体类尽量简单，但是可以写一些辅助方法，例如`User`类中增加`isEnabled`方法直接判断status属性字符串。
* 重新实现`toString`方法
* 可以增加数据库不存在的额外字段，但是要有说明或注解。

### Utils

* 包名以utils命名
* 类名以Utils结尾
* 辅助类可以不以utils结尾，例如：`MailSender`， 多个辅助类按情形可以都放在utils包中，或者重新建包。
* 私有化Utils类构造方法

### 值对象、业务对象分包、命名

* Java界常见对象缩写

  >PO(persistant object) 持久对象
  >
  >DO（Domain Object）领域对象
  >
  >TO(Transfer Object) ，数据传输对象
  >
  >DTO（Data Transfer Object）数据传输对象
  >
  >VO(view object) 视图对象
  >
  >BO(business object) 业务对象
  >
  >POJO(plain ordinary java object) 简单无规则 java 对象
  >
  >DAO(data access object) 数据访问对象

* 由于强类型带来的繁琐的数据格式转换问题，导致了很多的逻辑混淆。这里约定：

  * 定义vo包为业务拓展数据类型的包。
  * 拓展类以VO结尾

### Contants常量

* 数据库类型字段，可以用常量表示的一定用常量表示。
* 常量名称必须全部大写。
* 常量类可以嵌套表示，例如`A1.B1.XXX="xxx"`
* 用class定义常量，而不是interface。
* 常量可以定义在常量包中，如果使用范围不广，也可以定义在实体类中。
* 可以用枚举表示常量，增强常量的信息和行为。

​