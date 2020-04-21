---
layout: post
title: Java动态绑定和静态绑定
tags: Java
category: Java
description: 
---

# Java动态绑定和静态绑定

[TOC]

## 动态绑定

动态绑定是指编译器无法解析调用，仅仅在运行时绑定数据的功能。

Java的动态绑定，最直观的使用方法就是多态。

```java
class SuperClass {
    someMethod() {
        System.out.print("superclass say");
    }
}

class SubClass extends SuperClass｛
	someMethod() {
		System.out.print("subclass say");
	}
｝

...
SuperClass superClass1 = new SuperClass();
SuperClass superClass2 = new SubClass();
...

superClass1.someMethod(); // SuperClass version is called
superClass2.someMethod(); // SubClass version is called
....
```

因此，我们看到Java中的动态绑定只是绑定方法调用（只有在子类中可以覆盖的继承方法，因此编译器可能无法确定要调用的方法版本），具体取决于实际的对象类型和不在对象引用的声明类型上。 

## 静态绑定（或提前绑定）

如果编译器可以在编译时解析绑定的对象关系，那么称为静态绑定或提前绑定。

所有实例方法调用总是在运行时解析，但所有静态方法调用都在编译时自行解析，因此我们对静态方法调用进行静态绑定。因为静态方法是类方法，因此可以使用类名本身访问它们（实际上它们只是使用它们相应的类名而不是使用对象引用来使用）因此需要解析它们的访问权限在编译期间只使用编译时类型信息。这就是为什么静态方法实际上不能被覆盖的原因。阅读更多 - [您可以覆盖Java中的静态方法吗](http://geekexplains.blogspot.com/2008/06/can-you-override-static-methods-in-java.html)？ 



类似地，访问Java中的所有成员变量遵循静态绑定，因为Java不支持（事实上，它不鼓励）成员变量的多态行为。 

```java
class SuperClass{
...
public String someVariable = "Some Variable in SuperClass";
...
}

class SubClass extends SuperClass{
...
public String someVariable = "Some Variable in SubClass";
...
}
...
...
public class Main() {
    public static void say(SuperClass superClass) {
	    System.out.println(superClass.someVariable);
    }
    
    public static void main(String[] args) {
        SuperClass superClass1 = new SuperClass();
        SuperClass superClass2 = new SubClass();
        say(superClass1);
        say(supperClass2);
    }
}
...

```

```
Output:-
Some Variable in SuperClass
Some Variable in SuperClass
```

我们可以观察到，在这两种情况下，成员变量仅基于对象引用的声明类型进行解析，编译器只能在编译时查找，因此在这种情况下可以进行静态绑定。静态绑定的另一个例子是'private'方法，因为它们永远不会被继承，并且编译只能在编译时解析对任何私有方法的调用。

更新[2008年6月24日]：阅读更多内容以了解[Java中的字段隐藏是如何工作的？](http://geekexplains.blogspot.com/2008/06/field-hiding-in-java-fields-are-only.html)



转载：http://geekexplains.blogspot.com/2008/06/dynamic-binding-vs-static-binding-in.html