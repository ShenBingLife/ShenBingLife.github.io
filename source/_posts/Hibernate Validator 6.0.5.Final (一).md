---
layout: post
title: Hibernate Validator 6.0.5.Final （一）
category: Java
number: false
---

# Hibernate Validator 6.0.5.Final （一）

## 前言

验证数据是所有应用程序层（从演示文稿到持久层）发生的常见任务。 通常在每个层中实现相同的验证逻辑，这是耗时且容易出错的。 为了避免重复这些验证，开发人员经常将验证逻辑直接捆绑到领域模型中，使用验证代码来混淆领域类，验证代码实际上是关于类本身的元数据。

<!-- more -->

![image](https://tva2.sinaimg.cn/large/007S2YVMgy1gdzj1klmqqj30i207sq3q.jpg)

JSR 380 - Bean Validation 2.0 - 为实体和方法验证定义了元数据模型和API。 默认的元数据源是注解，可以通过使用XML覆盖和扩展元数据。 该API不绑定到特定的应用程序层或编程模型。 它特别与Web或持久层无关，可用于服务器端应用程序编程以及富客户端Swing应用程序开发人员。

![image](https://tvax4.sinaimg.cn/large/007S2YVMgy1gdzj1qdvcqj30i209mjsh.jpg)

Hibernate Validator是JSR 380的参考实现。实现本身以及Bean Validation API和TCK都是在[Apache Software License 2.0](http://www.apache.org/licenses/LICENSE-2.0)下提供和分发的。

Hibernate Validator 6和Bean Validation 2.0需要Java 8或更高版本。

## 1.开始

本章将向您展示如何开始使用Bean Validation的参考实现（RI）Hibernate Validator。为了以下快速启动，您需要：

- 一个JDK 8
- [Apache Maven](http://maven.apache.org/)
- 互联网连接（Maven必须下载所有必需的库）

### 1.1。 项目建立

为了在Maven项目中使用Hibernate Validator，只需将下面的依赖添加到你的*pom.xml中*：

例1.1：Hibernate验证器Maven依赖

```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.5.Final</version>
</dependency>
```

这个传递性的依赖于Bean Validation API（ `javax.validation:validation-api:2.0.0.Final` ）。

#### 1.1.1。 统一的EL

Hibernate Validator需要实现统一表达式语言（ [JSR 341](http://jcp.org/en/jsr/detail?id=341) ）来评估约束违例消息中的动态表达式（参见[第4.1节“默认消息插值”](#section-message-interpolation) ）。 当您的应用程序在JBoss AS等Java EE容器中运行时，容器已经提供了一个EL实现。 但是，在Java SE环境中，必须将实现作为依赖项添加到POM文件。 例如，您可以添加以下依赖项来使用JSR 341 [参考实现](https://javaee.github.io/uel-ri/) ：

例1.2：统一EL参考实现的Maven依赖关系

```xml
<dependency>
    <groupId>org.glassfish</groupId>
    <artifactId>javax.el</artifactId>
    <version>3.0.1-b08</version>
</dependency>
```

| **   | 对于无法提供EL实现的环境，Hibernate Validator提供了[第12.9节“ `ParameterMessageInterpolator` ”](#non-el-message-interpolator) 。 但是，使用此插补器不符合Bean Validation规范。 |
| ---- | ---------------------------------------- |
|      |                                          |

#### 1.1.2。 CDI

Bean Validation定义了CDI的集成点（Contexts and Dependency Injection for Java  EE, [JSR 346](http://jcp.org/en/jsr/detail?id=346) ）。 如果您的应用程序在不提供此集成的环境中运行，您可以通过将以下Maven依赖项添加到您的POM来使用Hibernate Validator CDI可移植扩展：

例1.3：Hibernate Validator CDI可移植扩展Maven依赖

```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator-cdi</artifactId>
    <version>6.0.5.Final</version>
</dependency>
```

请注意，在Java EE应用程序服务器上运行的应用程序通常不需要添加此依赖项。 您可以在[第11.3节“CDI”中](#section-integration-with-cdi)了解更多关于Bean Validation和CDI的集成。

#### 1.1.3。 与安全管理器一起运行

Hibernate Validator支持在启用[安全管理器的](http://docs.oracle.com/javase/8/docs/technotes/guides/security/index.html)情况下运行。 为此，您必须为Hibernate验证器和Bean验证API代码库分配多个权限。 以下显示了如何通过由Java默认策略实现处理的[策略文件](http://docs.oracle.com/javase/8/docs/technotes/guides/security/PolicyFiles.html)来执行此操作：

例1.4：使用Hibernate Validator和安全管理器的策略文件

```groovy
grant codeBase "file:path/to/hibernate-validator-6.0.5.Final.jar" {
    permission java.lang.reflect.ReflectPermission "suppressAccessChecks";
    permission java.lang.RuntimePermission "accessDeclaredMembers";
    permission java.lang.RuntimePermission "setContextClassLoader";

    permission org.hibernate.validator.HibernateValidatorPermission "accessPrivateMembers";

    // Only needed when working with XML descriptors (validation.xml or XML constraint mappings)
    permission java.util.PropertyPermission "mapAnyUriToUri", "read";
};

grant codeBase "file:path/to/validation-api-2.0.0.Final.jar" {
    permission java.io.FilePermission "path/to/hibernate-validator-6.0.5.Final.jar", "read";
};
```

所有需要特殊权限的API调用都是通过特权操作完成的。 这意味着只有Hibernate Validator和Bean Validation API本身需要列出的权限。 您不需要为调用Hibernate Validator的其他代码库分配任何权限。

#### 1.1.4。 在WildFly中更新Hibernate验证器

[WildFly应用程序服务器](http://wildfly.org/)包含开箱即用的Hibernate Validator。 为了将Bean Validation API和Hibernate Validator的服务器模块更新到最新，可以使用WildFly的补丁机制。

您可以使用以下依赖项从[SourceForge](http://sourceforge.net/projects/hibernate/files/hibernate-validator)或Maven Central下载修补程序文件：

例1.5：Maven依赖于WildFly 11.0.0.Final补丁文件

```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator-modules</artifactId>
    <version>6.0.5.Final</version>
    <classifier>wildfly-11.0.0.Final-patch</classifier>
    <type>zip</type>
</dependency>
```

我们还为以前版本的WildFly提供了一个补丁：

例1.6：WildFly 10.1.0.Final补丁文件的Maven依赖关系

```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator-modules</artifactId>
    <version>6.0.5.Final</version>
    <classifier>wildfly-10.1.0.Final-patch</classifier>
    <type>zip</type>
</dependency>
```

下载了补丁文件后，可以运行以下命令将其应用于WildFly：

例1.7：应用WildFly补丁

```shell
 $JBOSS_HOME/bin/jboss-cli.sh patch apply hibernate-validator-modules-6.0.5.Final-wildfly-11.0.0.Final-patch.zip 
```

如果您想要取消修补程序并返回最初随服务器提供的Hibernate Validator版本，请运行以下命令：

例1.8：回滚WildFly补丁

```shell
 $JBOSS_HOME/bin/jboss-cli.sh patch rollback --reset-configuration=true 
```

一般来说，您可以在[这里](https://developer.jboss.org/wiki/SingleInstallationPatching/)和[这里](http://www.mastertheboss.com/jboss-server/jboss-configuration/managing-wildfly-and-eap-patches)了解更多关于WildFly修补程序的基础结构。

#### 1.1.5。 在Java 9上运行

从Hibernate Validator 6.0.5.Final开始，对Java 9和Java平台模块系统（JPMS）的支持就是实验性的。 目前还没有提供JPMS模块描述符，但Hibernate Validator可用作 automatic modules。

这些是使用`Automatic-Module-Name`头声明的模块名称 ：

- Bean验证API： `java.validation`
- Hibernate Validator核心： `org.hibernate.validator`
- Hibernate Validator CDI扩展： `org.hibernate.validator.cdi`
- Hibernate Validator测试实用程序： `org.hibernate.validator.testutils`
- Hibernate Validator注解处理器： `org.hibernate.validator.annotationprocessor`

这些模块名称是初步的，在未来版本中提供真实模块描述符时可能会更改。

当使用Bean验证XML描述符（ *META-INF / validation.xml* 和（或）约束映射文件）时，必须启用`java.xml.bind`模块。 通过追加`--add-modules java.xml.bind`到你的*java*调用。

| **   | 在CDI中使用Hibernate Validator时，请注意不要启用JDK的`java.xml.ws.annotation`模块。 该模块包含JSR 250 API的一个子集（“Commons Annotations”），但缺少一些注解，如`javax.annotation.Priority` 。 这会导致Hibernate Validator的方法验证拦截器不被注册，即方法验证将不起作用。<br/><br/>相反，将完整的JSR 250 API添加到未命名模块（即类路径）中，例如通过引入*javax.annotation：javax.annotation-api*依赖关系（这里已经存在JSR 250 API的依赖关系*当依赖 hibernate.validator：hibernate-validator-cdi* ）。<br>如果由于某种原因需要启用`java.xml.ws.annotation`模块，则应使用完整API的内容修补它， 通过附加`--patch-module java.xml.ws.annotation=/path/to/complete-jsr250-api.jar`到你的*java*调用。 |
| ---- | ---------------------------------------- |
|      |                                          |

### 1.2。 应用约束

让我们直接看一个例子，看看如何应用约束。

例1.9：带限制条件的Class Car

```java
package org.hibernate.validator.referenceguide.chapter01;

import javax.validation.constraints.Min;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

public class Car {

    @NotNull
    private String manufacturer;

    @NotNull
    @Size(min = 2, max = 14)
    private String licensePlate;

    @Min(2)
    private int seatCount;

    public Car(String manufacturer, String licencePlate, int seatCount) {
        this.manufacturer = manufacturer;
        this.licensePlate = licencePlate;
        this.seatCount = seatCount;
    }

    //getters and setters ...
}
```

`@Size` `@Min` ， `@Size`和`@Min`注释用于声明应该应用于Car实例字段的约束：

- `manufacturer`绝不能为`null`
- `licensePlate`不能为`null`且`licensePlate`必须在2到14个字符之间
- `seatCount`必须至少为2

| **   | 您可以在GitHub上的Hibernate Validator [源代码仓库](https://github.com/hibernate/hibernate-validator/tree/master/documentation/src/test)中找到本参考指南中使用的所有示例的完整源代码。 |
| ---- | ---------------------------------------- |
|      |                                          |

### 1.3。 验证约束

要执行这些约束的验证，请使用`Validator`实例。 我们来看一下`Car`的单元测试：

例1.10：Class CarTest显示验证示例

```java
package org.hibernate.validator.referenceguide.chapter01;

import java.util.Set;
import javax.validation.ConstraintViolation;
import javax.validation.Validation;
import javax.validation.Validator;
import javax.validation.ValidatorFactory;

import org.junit.BeforeClass;
import org.junit.Test;

import static org.junit.Assert.assertEquals;

public class CarTest {

    private static Validator validator;

    @BeforeClass
    public static void setUpValidator() {
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        validator = factory.getValidator();
    }

    @Test
    public void manufacturerIsNull() {
        Car car = new Car( null, "DD-AB-123", 4 );

        Set<ConstraintViolation<Car>> constraintViolations =
                validator.validate( car );

        assertEquals( 1, constraintViolations.size() );
        assertEquals( "must not be null", constraintViolations.iterator().next().getMessage() );
    }

    @Test
    public void licensePlateTooShort() {
        Car car = new Car( "Morris", "D", 4 );

        Set<ConstraintViolation<Car>> constraintViolations =
                validator.validate( car );

        assertEquals( 1, constraintViolations.size() );
        assertEquals(
                "size must be between 2 and 14",
                constraintViolations.iterator().next().getMessage()
        );
    }

    @Test
    public void seatCountTooLow() {
        Car car = new Car( "Morris", "DD-AB-123", 1 );

        Set<ConstraintViolation<Car>> constraintViolations =
                validator.validate( car );

        assertEquals( 1, constraintViolations.size() );
        assertEquals(
                "must be greater than or equal to 2",
                constraintViolations.iterator().next().getMessage()
        );
    }

    @Test
    public void carIsValid() {
        Car car = new Car( "Morris", "DD-AB-123", 2 );

        Set<ConstraintViolation<Car>> constraintViolations =
                validator.validate( car );

        assertEquals( 0, constraintViolations.size() );
    }
}
```

在`setUp()`方法中，从`ValidatorFactory`检索`Validator`对象。 `Validator`实例是线程安全的，可以重复使用多次。 因此，它可以安全地存储在一个静态字段中，并在测试方法中用于验证不同的`Car`实例。

`validate()`方法返回一组`ConstraintViolation`实例，您可以遍历该实例以查看发生了哪些验证错误。 前三种测试方法显示了一些预期的约束违规：

- `manufacturerIsNull()`违反了`manufacturerIsNull()`的`@NotNull`约束
- `@Size`约束违反了`licensePlateTooShort()`
- `@Min`约束在`seatCountTooLow()`被违反

如果对象验证成功， `validate()`将返回一个空集，如您在`carIsValid()`看到的。

请注意，仅使用包`javax.validation`中的类。 这些是从Javax Bean Validation API提供的。 没有从Hibernate Validator的类直接引用，编写出的代码具有移植性。

### 1.4。 下一步该去哪里？

通过Hibernate验证器和Bean验证的5分钟结束。 继续研究代码示例或查看[第14章“ *深入阅读”中*](#validator-further-reading)引用的更多示例。

要了解更多关于bean和属性的验证，请继续阅读[第2章*声明和验证bean约束*](#chapter-bean-constraints) 。 如果您有兴趣使用Bean验证来验证方法的前置和后置条件，请参阅[第3章*声明和验证方法约束*](#chapter-method-constraints) 。 如果您的应用程序具有特定的验证要求[，请*参阅*第6章“ *创建自定义约束”*](#validator-customconstraints) 。



## 2.声明和验证bean约束

在本章中，您将学习如何[声明bean](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.co.uk&sl=en&sp=nmt4&tl=zh-CN&u=https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/&usg=ALkJrhgD-MH0gmBhw9uAJj9NkshS0SvLog#section-declaring-bean-constraints) （见[第2.1节“声明bean约束”](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.co.uk&sl=en&sp=nmt4&tl=zh-CN&u=https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/&usg=ALkJrhgD-MH0gmBhw9uAJj9NkshS0SvLog#section-declaring-bean-constraints) ）和验证（参见[第2.2节“验证bean约束”](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.co.uk&sl=en&sp=nmt4&tl=zh-CN&u=https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/&usg=ALkJrhgD-MH0gmBhw9uAJj9NkshS0SvLog#section-validating-bean-constraints) ）。 [第2.3节“内置约束”](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.co.uk&sl=en&sp=nmt4&tl=zh-CN&u=https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/&usg=ALkJrhgD-MH0gmBhw9uAJj9NkshS0SvLog#section-builtin-constraints)提供了Hibernate Validator附带的所有内置约束的概述。

如果您有兴趣将约束应用于方法参数和返回值，请参阅[第3章*声明和验证方法约束*](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.co.uk&sl=en&sp=nmt4&tl=zh-CN&u=https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/&usg=ALkJrhgD-MH0gmBhw9uAJj9NkshS0SvLog#chapter-method-constraints) 。

### 2.1。 声明bean约束

Bean验证中的约束通过Java注解来表达。 在本节中，您将学习如何使用这些注解来增强对象模型。 有四种类型的bean约束：

- field constraints（字段约束）
- property constraints（属性约束）
- container element constraints（容器元素约束，Java泛型元素约束）
- class constraints（类约束）

| **   | 并不是所有的约束都可以放在所有这些层次上。 实际上，Bean Validation定义的默认约束都不能放在类级别。 约束注解中的`java.lang.annotation.Target`注解本身决定了哪些元素可以放置一个约束。 有关更多信息请参见[第6章*创建自定义约束*](#validator-customconstraints) 。 |
| ---- | ---------------------------------------- |
|      |                                          |

#### 2.1.1。 字段级别约束

约束可以通过注解一个类的字段来表示。 [示例2.1“字段级别约束”]()显示了字段级别的配置示例：

例2.1：字段约束

```java
package org.hibernate.validator.referenceguide.chapter02.fieldlevel;

public class Car {

    @NotNull
    private String manufacturer;

    @AssertTrue
    private boolean isRegistered;

    public Car(String manufacturer, boolean isRegistered) {
        this.manufacturer = manufacturer;
        this.isRegistered = isRegistered;
    }

    //getters and setters...
}
```

当使用字段级别约束时，字段访问策略用于访问要验证的值。 这意味着验证引擎直接访问实例变量，即使存在这样的访问器，也不会调用属性访问器方法。

约束可以应用于任何访问类型（公共，私人等）的字段。 不过，不支持静态字段的约束。

| **   | 在验证字节码增强对象时，应该使用属性级别约束，因为字节码增强库将无法通过反射来确定字段访问。 |
| ---- | ---------------------------------------- |
|      |                                          |

#### 2.1.2。 属性级别约束

如果您的模型类符合[JavaBean](http://www.oracle.com/technetwork/articles/javaee/spec-136004.html)标准，则也可以注释一个bean类的属性，而不是其字段。[例2.2“属性级约束”]()使用与[例2.1“字段级约束”中]()相同的实体，但是使用属性级约束。

例子2.2：属性级约束

```java
package org.hibernate.validator.referenceguide.chapter02.propertylevel;

public class Car {

    private String manufacturer;

    private boolean isRegistered;

    public Car(String manufacturer, boolean isRegistered) {
        this.manufacturer = manufacturer;
        this.isRegistered = isRegistered;
    }

    @NotNull
    public String getManufacturer() {
        return manufacturer;
    }

    public void setManufacturer(String manufacturer) {
        this.manufacturer = manufacturer;
    }

    @AssertTrue
    public boolean isRegistered() {
        return isRegistered;
    }

    public void setRegistered(boolean isRegistered) {
        this.isRegistered = isRegistered;
    }
}
```

| **   | 该属性的getter方法必须被注解，而不是其setter。 这样也可以限制当没有Getter方法时的只读属性。 |
| ---- | ---------------------------------------- |
|      |                                          |

使用属性级别约束时，使用属性访问策略访问要验证的值，即验证引擎通过属性访问器方法访问状态。

| **   | 建议在一个类中使用字段*或*属性级别注解。 不建议同时注解字段和getter方法，因为这会导致字段被验证两次。 |
| ---- | ---------------------------------------- |
|      |                                          |

#### 2.1.3。 容器元素约束

可以直接在泛型化参数的泛型参数中指定约束：这些约束称为容器元素约束。

这要求在约束定义中通过`@Target`指定`ElementType.TYPE_USE` 。 从Bean Validation 2.0开始，内置的Bean Validation以及Hibernate Validator特定的约束指定了`ElementType.TYPE_USE`的校验注解可以直接在这种情况下使用。

Hibernate Validator验证在以下标准Java容器上指定的容器元素约束：

- `java.util.Iterable`实现（例如`List` s， `Set` s），
- `java.util.Map`实现，支持键和值，
- `java.util.Optional` ， `java.util.OptionalInt` ， `java.util.OptionalDouble` ， `java.util.OptionalLong` ，
- JavaFX的`javafx.beans.observable.ObservableValue`的各种实现。

它还支持自定义容器类型的容器元素约束（请参见[第7章， *值提取*](#chapter-valueextraction) ）。

| **   | 在hibernate validator 6之前的版本中，支持容器元素约束的一个子集， 在容器级别需要`@Valid`注解来启用它们。 从Hibernate Validator 6开始，这不再是必需的。 |
| ---- | ---------------------------------------- |
|      |                                          |

下面介绍几个例子，说明各种Java类型的容器元素约束。

在这些示例中， `@ValidPart`是一个允许在`TYPE_USE`上下文中使用的自定义约束。

##### 2.1.3.1。 使用`Iterable`

在`Iterable`类型参数上应用约束时，Hibernate Validator会验证每个元素。 [例2.3，“`Set`上的容器元素约束”](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.co.uk&sl=en&sp=nmt4&tl=zh-CN&u=https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/&usg=ALkJrhgD-MH0gmBhw9uAJj9NkshS0SvLog#example-container-element-constraints-iterable)显示了一个带有容器元素约束的`Set`的例子。

例2.3： `Set`上的容器元素约束

```java
package org.hibernate.validator.referenceguide.chapter02.containerelement.set;

import java.util.HashSet;
import java.util.Set;

public class Car {

    private Set<@ValidPart String> parts = new HashSet<>();

    public void addPart(String part) {
        parts.add( part );
    }

    //...

}
```

```java
Car car = new Car();
car.addPart( "Wheel" );
car.addPart( null );

Set<ConstraintViolation<Car>> constraintViolations = validator.validate( car );

assertEquals( 1, constraintViolations.size() );

ConstraintViolation<Car> constraintViolation =
        constraintViolations.iterator().next();
assertEquals(
        "'null' is not a valid car part.",
        constraintViolation.getMessage()
);
assertEquals( "parts[].<iterable element>",
        constraintViolation.getPropertyPath().toString() );
```

请注意属性路径如何明确指出违规来自可迭代元素。

##### 2.1.3.2。 与`List`

当在`List`类型参数上应用约束时，Hibernate Validator会验证每个元素。 [例2.4，“`List`上的容器元素约束”]()显示了一个带容器元素约束的`List`的例子。

例2.4： `List`上的容器元素约束

```java
package org.hibernate.validator.referenceguide.chapter02.containerelement.list;

public class Car {

    private List<@ValidPart String> parts = new ArrayList<>();

    public void addPart(String part) {
        parts.add( part );
    }

    //...

}
```

```java
Car car = new Car();
car.addPart( "Wheel" );
car.addPart( null );

Set<ConstraintViolation<Car>> constraintViolations = validator.validate( car );

assertEquals( 1, constraintViolations.size() );

ConstraintViolation<Car> constraintViolation =
        constraintViolations.iterator().next();
assertEquals(
        "'null' is not a valid car part.",
        constraintViolation.getMessage()
);
assertEquals( "parts[1].<list element>",
        constraintViolation.getPropertyPath().toString() );
```

在这里，属性路径也包含无效元素的索引。

##### 2.1.3.3。 与`Map`

容器元素约束也在Map键和值上进行验证。 [例2.5，“映射键和值的容器元素约束”]()显示了一个关于键和值约束的`Map`的例子。

例2.5：映射键和值的容器元素约束

```java
package org.hibernate.validator.referenceguide.chapter02.containerelement.map;

import java.util.HashMap;
import java.util.Map;

import javax.validation.constraints.NotNull;

public class Car {

    public enum FuelConsumption {
        CITY,
        HIGHWAY
    }

    private Map<@NotNull FuelConsumption, @MaxAllowedFuelConsumption Integer> fuelConsumption = new HashMap<>();

    public void setFuelConsumption(FuelConsumption consumption, int value) {
        fuelConsumption.put( consumption, value );
    }

    //...

}
```

```java
Car car = new Car();
car.setFuelConsumption( Car.FuelConsumption.HIGHWAY, 20 );

Set<ConstraintViolation<Car>> constraintViolations = validator.validate( car );

assertEquals( 1, constraintViolations.size() );

ConstraintViolation<Car> constraintViolation =
        constraintViolations.iterator().next();
assertEquals(
        "20 is outside the max fuel consumption.",
        constraintViolation.getMessage()
);
assertEquals(
        "fuelConsumption[HIGHWAY].<map value>",
        constraintViolation.getPropertyPath().toString()
);
```

```java
Car car = new Car();
car.setFuelConsumption( null, 5 );

Set<ConstraintViolation<Car>> constraintViolations = validator.validate( car );

assertEquals( 1, constraintViolations.size() );

ConstraintViolation<Car> constraintViolation =
        constraintViolations.iterator().next();
assertEquals(
        "must not be null",
        constraintViolation.getMessage()
);
assertEquals(
        "fuelConsumption<K>[].<map key>",
        constraintViolation.getPropertyPath().toString()
);
```

违反约束的属性路径特别有趣：

- 无效元素的key包含在属性路径中（在第二个示例中，key为`null` ）。
- 在第一个例子中，违规涉及`<map value>` ，第二个是`<map key>` 。
- 在第二个例子中，您可能已经注意到泛型参数`<K>`的存在，稍后会有更多介绍。

##### 2.1.3.4。 用`java.util.Optional`

当在`Optional`的type参数上应用一个约束时，Hibernate Validator会自动打开类型并验证内部值。 [例2.6，“可选的容器元素约束”]()显示了带有容器元素约束的可选项。

示例2.6：可选的容器元素约束

```java
package org.hibernate.validator.referenceguide.chapter02.containerelement.optional;

public class Car {

    private Optional<@MinTowingCapacity(1000) Integer> towingCapacity = Optional.empty();

    public void setTowingCapacity(Integer alias) {
        towingCapacity = Optional.of( alias );
    }

    //...

}
```

```java
Car car = new Car();
car.setTowingCapacity( 100 );

Set<ConstraintViolation<Car>> constraintViolations = validator.validate( car );

assertEquals( 1, constraintViolations.size() );

ConstraintViolation<Car> constraintViolation = constraintViolations.iterator().next();
assertEquals(
        "Not enough towing capacity.",
        constraintViolation.getMessage()
);
assertEquals(
        "towingCapacity",
        constraintViolation.getPropertyPath().toString()
);
```

在这里，属性路径只包含属性的名称，因为我们正在考虑将`Optional` 作为“透明”容器。

##### 2.1.3.5。 使用自定义容器类型

容器元素约束也可以用于自定义容器。

必须为自定义类型注册`ValueExtractor` ，以允许检索要验证的值（有关如何实现自己的`ValueExtractor`以及如何注册的更多信息，请参阅[第7章， *值提取*](#chapter-valueextraction) ）。

[例2.7，“自定义容器类型的容器元素约束”]()显示了一个具有泛型参数约束的自定义泛型类型。

例2.7：自定义容器类型的容器元素约束

```java
package org.hibernate.validator.referenceguide.chapter02.containerelement.custom;

public class Car {

    private GearBox<@MinTorque(100) Gear> gearBox;

    public void setGearBox(GearBox<Gear> gearBox) {
        this.gearBox = gearBox;
    }

    //...

}
```

```java
 package org.hibernate.validator.referenceguide.chapter02.containerelement.custom;

public class GearBox<T extends Gear> {

    private final T gear;

    public GearBox(T gear) {
        this.gear = gear;
    }

    public Gear getGear() {
        return this.gear;
    }
}
```

```java
package org.hibernate.validator.referenceguide.chapter02.containerelement.custom;

public class Gear {
    private final Integer torque;

    public Gear(Integer torque) {
        this.torque = torque;
    }

    public Integer getTorque() {
        return torque;
    }

    public static class AcmeGear extends Gear {
        public AcmeGear() {
            super( 100 );
        }
    }
}
```

```java
package org.hibernate.validator.referenceguide.chapter02.containerelement.custom;

public class GearBoxValueExtractor implements ValueExtractor<GearBox<@ExtractedValue ?>> {

    @Override
    public void extractValues(GearBox<@ExtractedValue ?> originalValue, ValueExtractor.ValueReceiver receiver) {
        receiver.value( null, originalValue.getGear() );
    }
}
```

```java
Car car = new Car();
car.setGearBox( new GearBox<>( new Gear.AcmeGear() ) );

Set<ConstraintViolation<Car>> constraintViolations = validator.validate( car );
assertEquals( 1, constraintViolations.size() );

ConstraintViolation<Car> constraintViolation =
        constraintViolations.iterator().next();
assertEquals(
        "Gear is not providing enough torque.",
        constraintViolation.getMessage()
);
assertEquals(
        "gearBox",
        constraintViolation.getPropertyPath().toString()
);
```

##### 2.1.3.6。 嵌套的容器元素

嵌套的容器元素也支持约束。

在[例2.8“嵌套容器元素](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.co.uk&sl=en&sp=nmt4&tl=zh-CN&u=https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/&usg=ALkJrhgD-MH0gmBhw9uAJj9NkshS0SvLog#example-container-element-nested)约束条件中验证`Car`对象时， `Part`和`Manufacturer`上的`@NotNull`约束条件都将被强制执行。

例子2.8：嵌套的容器元素的约束

```java
package org.hibernate.validator.referenceguide.chapter02.containerelement.nested;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

import javax.validation.constraints.NotNull;

public class Car {

    private Map<@NotNull Part, List<@NotNull Manufacturer>> partManufacturers =
            new HashMap<>();

    //...
}
```

#### 2.1.4。 类级别约束

最后但并非最不重要的，还可以在类上设置一个约束。 在这种情况下，单个属性并不是验证的目标，而是完整的对象。 如果验证依赖于对象的多个属性之间的相关性，则类级约束是有用的。

[例2.9中的类级别约束“]() `Car`具有`seatCount`和`passengers`这两个属性，应该确保乘客名单没有比可用座位更多的条目。 为此， `@ValidPassengerCount`约束被添加到类级别上。 该约束的验证者可以访问完整的`Car`对象，从而可以比较座位和乘客的数量。

请参阅[第6.2节“类级约束”](#section-class-level-constraints)以详细了解如何实现此自定义约束。

例2.9：类级约束

```java
package org.hibernate.validator.referenceguide.chapter02.classlevel;

@ValidPassengerCount
public class Car {

    private int seatCount;

    private List<Person> passengers;

    //...
}
```

#### 2.1.5。 约束继承

当一个类实现了一个接口或者扩展了另一个类的时候，在父类型上声明的所有约束注解的应用方式与在类本身上指定的约束相同。 为了使事情更清晰，让我们看看下面的例子：

例2.10：约束继承

```java
package org.hibernate.validator.referenceguide.chapter02.inheritance;

public class Car {

    private String manufacturer;

    @NotNull
    public String getManufacturer() {
        return manufacturer;
    }

    //...
}
```

```java
package org.hibernate.validator.referenceguide.chapter02.inheritance;

public class RentalCar extends Car {

    private String rentalStation;

    @NotNull
    public String getRentalStation() {
        return rentalStation;
    }

    //...
}
```

在这里类`RentalCar`是`Car`一个子类，并添加属性`rentalStation` 。 如果`RentalCar`的实例被验证，则不仅会校验`rentalStation`的`@NotNull`约束，还会对父类的`manufacturer`的约束进行校验。

如果`Car`不是超类，而是由`RentalCar`实现的接口，情况如上相同 。

如果方法被覆盖，约束注解将被聚合。 因此，如果`RentalCar`覆盖了`Car`的`getManufacturer()`方法，则除了超类的`@NotNull`约束之外，还会校验在重写方法中注解的任何约束。

#### 2.1.6。 对象级联校验

Bean验证API不仅允许验证一个类的实例，也能实现完整的对象关联校验（级联验证）。为了实现级联校验，只要在字段或属性上增加注解`@Valid`。[2.11，“级联验证”]()。

实施例2.11：级联验证

```java
package org.hibernate.validator.referenceguide.chapter02.objectgraph;

public class Car {

    @NotNull
    @Valid
    private Person driver;

    //...
}
```

```java
package org.hibernate.validator.referenceguide.chapter02.objectgraph;

public class Person {

    @NotNull
    private String name;

    //...
}
```

如果实例`Car`被验证，被引用的`Person`对象也将被验证，因为`driver`字段被`@Valid`注解。因此`Person`的name为null，Car的校验也会失败。

对象关联的验证是递归的，也就是说，如果对级联验证点标记的对象本身具有有`@Valid`注解的属性，则这些引用也将由验证引擎跟踪。验证引擎将确保在级联验证期间不会出现无限循环，例如，如果两个对象互相持有引用。

需要注意的是`null`值在级联验证过程中被忽略。

作为约束条件，对象关联的验证也适用于容器的元素。这意味着可以对任意泛型参数的泛型类型使用`@Valid`，这将导致父对象被校验时每个容器内的元素被校验。

| **   | 级联验证还支持约束嵌套容器内的元素。 |
| ---- | ------------------ |
|      |                    |

实施例2.12：容器的级联验证

```java
package org.hibernate.validator.referenceguide.chapter02.objectgraph.containerelement;

public class Car {

    private List<@NotNull @Valid Person> passengers = new ArrayList<Person>();

    private Map<@Valid Part, List<@Valid Manufacturer>> partManufacturers = new HashMap<>();

    //...
}
```

```java
package org.hibernate.validator.referenceguide.chapter02.objectgraph.containerelement;

public class Part {

    @NotNull
    private String name;

    //...
}
```

```java
package org.hibernate.validator.referenceguide.chapter02.objectgraph.containerelement;

public class Manufacturer {

    @NotNull
    private String name;

    //...
}
```

当验证的实例`Car`中所示的类[实施例2.12，“级联容器的验证”]()，一个`ConstraintViolation`将被创建：

- 如果任何的`Person`对象有一个`null`名称;
- 如果任何Map中的`Part`对象有一个`null`名称;
- 如果任何的`Manufacturer`对象有一个`null`名字。

| **   | 在之前的6个版本，Hibernate验证支持的级联验证的容器元素的子集，它是在容器级别实现的（例如，你会用`@Valid private List<Person>`启用级联验证`Person`）。这仍然是支持，但不建议使用。请使用容器元素级别`@Valid`的注解，因为它更具表现力。 |
| ---- | ---------------------------------------- |
|      |                                          |

### 2.2。 验证Bean约束

`Validator`接口在Bean验证中最重要的对象。下一节将展示如何获取一个`Validator`实例。之后，您将学习如何使用`Validator`接口中的不同的方法。

#### 2.2.1。 获取`Validator`实例

校验对象实例的第一步是获取一个`Validator`实例。实现这个的过程，就是通过`Validation`类和`ValidatorFactory`。最简单的方法是使用静态方法`Validation#buildDefaultValidatorFactory()`：

实施例2.13： `Validation#buildDefaultValidatorFactory()`

```java
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
validator = factory.getValidator();
```

这个启动步骤可以创建一个默认配置的验证器。请参阅[第9章，*引导*](#chapter-bootstrapping)更多地了解不同的引导方法和如何获得特定配置的`Validator`实例。

#### 2.2.2。 验证方法

`Validator`接口包含验证整个实体或实体单一属性的三种方法。

这三种方法返回`Set<ConstraintViolation>`。如果`Set<ConstraintViolation>`为空，则验证成功。否则一个`ConstraintViolation`实例中添加了每个违反约束。

在执行校验时，所有的验证方法都有方法参数来约定校验分组。如果未指定参数，则默认验证组（`javax.validation.groups.Default`使用）。验证组的主题进行了详细讨论[第5章，*分组的约束*](#chapter-groups)。

##### 2.2.2.1。 `Validator#validate()`

使用`validate()`方法执行给定的bean的所有约束验证。[例2.14，“使用`Validator#validate()`”]()展示了例2.2属性校验的代码，其不能满足该`@NotNull`上约束`manufacturer`属性。因此，验证调用返回一个`ConstraintViolation`对象。

例如2.14：使用 `Validator#validate()`

```java
Car car = new Car( null, true );

Set<ConstraintViolation<Car>> constraintViolations = validator.validate( car );

assertEquals( 1, constraintViolations.size() );
assertEquals( "must not be null", constraintViolations.iterator().next().getMessage() );
```

##### 2.2.2.2。 `Validator#validateProperty()`

使用`validateProperty()`，你可以验证一个给定对象的指定名称的属性。属性名称必须符合JavaBeans的规范。

例如2.15：使用 `Validator#validateProperty()`

```java
Car car = new Car( null, true );

Set<ConstraintViolation<Car>> constraintViolations = validator.validateProperty(
        car,
        "manufacturer"
);

assertEquals( 1, constraintViolations.size() );
assertEquals( "must not be null", constraintViolations.iterator().next().getMessage() );
```

##### 2.2.2.3。 `Validator#validateValue()`

通过使用该`validateValue()`方法，您对校验指定对象的属性赋予指定的值，判断是否能校验通过：

实施例2.16：使用 `Validator#validateValue()`

```java
Set<ConstraintViolation<Car>> constraintViolations = validator.validateValue(
        Car.class,
        "manufacturer",
        null
);

assertEquals( 1, constraintViolations.size() );
assertEquals( "must not be null", constraintViolations.iterator().next().getMessage() );
```

| **   | `@Valid`不会被触发`validateProperty()`或`validateValue()`。 |
| ---- | ---------------------------------------- |
|      |                                          |

`Validator#validateProperty()`例如在集成Bean验证的成JSF 2中使用（见[第11.2节，“JSF＆Seam”](#section-presentation-layer)），校验那些在表单中的属性值，但是尚未被传入模型对象之前。

#### 2.2.3。 `ConstraintViolation`

##### 2.2.3.1。`ConstraintViolation`方法

现在是时候细看`ConstraintViolation`。使用`ConstraintViolation`中不同的方法可以获得很多有关验证失败的原因的有用信息。下面给出了这些方法的概述。下面示例中错误信息的值参考自[例2.14，“使用`Validator#validate()`”]()。

- `getMessage()`

  插值后的错误信息

  "must not be null"

- `getMessageTemplate()`

  尚未插值的错误信息字符串

  "{… NotNull.message}"

- `getRootBean()`

  最顶层被校验的对象

  car

- `getRootBeanClass()`

  顶层被校验的对象的类

  `Car.class`

- `getLeafBean()`

  如果一个bean约束，返回bean的实例; 如果一个属性约束，则返回托管属性的bean实例的

- `getPropertyPath()`

  属性路径从根bean的到被校验的对象

  返回一个节点`PROPERTY`，并命名为“manufacturer”

- `getInvalidValue()`

  返回未通过校验的值

- `getConstraintDescriptor()`

  返回校验约束的元数据

  `@NotNull`

##### 2.2.3.2。 利用属性路径

为了确定触发约束的元素，你需要使用`getPropertyPath()`的方法确定约束元素的路径。

返回的`Path`是校验元素表示的`Node` 的路径组成的。

`Path`和各种类型的`Node`的详细信息参考`ConstraintViolation`部分。

### 2.3。 内置的约束

Hibernate验证包括一组基本的通用的约束。这些是最重要的由Bean验证规范中定义的约束条件（见[第2.3.1节“Bean验证约束”](#validator-defineconstraints-spec)）。此外，Hibernate验证提供了有用的自定义约束条件（见[第2.3.2节“附加约束”](#validator-defineconstraints-hv-constraints)）。

#### 2.3.1。 Bean验证约束

下面可以找到所有的Bean Validation API定义的约束。所有这些约束只能在字段或属性上使用，没有支持类级别的校验约束。如果你使用了Hibernate ORM，一些约束可以直接在DDL模型操作时使用。

| **   | Hibernate验证允许一些限制约束超出标准的校验规范（例如`@Max`可应用于字符串）。依托这个功能会影响Bean验证提供者之间的应用程序的可移植性。 |
| ---- | ---------------------------------------- |
|      |                                          |

- `@AssertFalse`

  检查被注解的元素是假的。支持的数据类型`Boolean` ， `boolean`。 Hibernate的元数据影响没有

- `@AssertTrue`

  检查被注解的元素是真实的。支持的数据类型`Boolean` ， `boolean`。 Hibernate的元数据影响没有

- `@DecimalMax(value=, inclusive=)`

  检查被注解的值是否小于规定的最大值，当`inclusive`=false。否则，校验该值是否小于或等于指定的最大值。参数值是根据所述最大值的字符串表示`BigDecimal`字符串表示。支持的数据类型`BigDecimal`，`BigInteger`，`CharSequence`，`byte`，`short`，`int`，`long`和原始类型的相应的包装; 通过HV另外支持：任何`Number`和`javax.money.MonetaryAmount`的子类（如果[JSR 354 API]()在项目中被引入）。Hibernate的元数据影响没有

- `@DecimalMin(value=, inclusive=)`

  检查注释的值是否大于指定的最小值，当较大`inclusive`=false。否则判断该值是否大于或等于指定的最小值。参数值是根据所述最小值的字符串表示`BigDecimal`字符串表示。支持的数据类型`BigDecimal`，`BigInteger`，`CharSequence`，`byte`，`short`，`int`，`long`和原始类型的相应的包装; 通过HV另外支持：任何`Number`和`javax.money.MonetaryAmount`的子类。Hibernate的元数据影响没有

- `@Digits(integer=, fraction=)`

  检查注释的值的整数位数是否到达`integer`大小和小数位数是否到达`fraction`。支持的数据类型`BigDecimal`，`BigInteger`，`CharSequence`，`byte`，`short`，`int`，`long`和原始类型的相应的包装; 通过HV另外支持：任何`Number`的子类型。Hibernate的元数据影响：定义数据库列的精度和规模（precision and scale）

- `@Email`

  检查指定的字符序列是否是一个有效的电子邮件地址。可选参数`regexp`和`flags`允许指定附加的正则表达式（包括正则表达式标志）。支持的数据类型`CharSequence`。 Hibernate的元数据影响没有

- `@Future`

  检查注释的日期是否在将来。支持的数据类型`java.util.Date`，`java.util.Calendar`，`java.time.Instant`，`java.time.LocalDate`，`java.time.LocalDateTime`，`java.time.LocalTime`，`java.time.MonthDay`，`java.time.OffsetDateTime`，`java.time.OffsetTime`，`java.time.Year`，`java.time.YearMonth`，`java.time.ZonedDateTime`，`java.time.chrono.HijrahDate`，`java.time.chrono.JapaneseDate`，`java.time.chrono.MinguoDate`，`java.time.chrono.ThaiBuddhistDate`; 通过HV另外的支持，如果[Joda Time]()的日期/时间API被引入类路径：那么任何`ReadablePartial`和`ReadableInstant`的实现类都会支持。Hibernate的元数据影响没有

- `@FutureOrPresent`

  检查注释的日期是否在当前或将来。支持的数据类型`java.util.Date`，`java.util.Calendar`，`java.time.Instant`，`java.time.LocalDate`，`java.time.LocalDateTime`，`java.time.LocalTime`，`java.time.MonthDay`，`java.time.OffsetDateTime`，`java.time.OffsetTime`，`java.time.Year`，`java.time.YearMonth`，`java.time.ZonedDateTime`，`java.time.chrono.HijrahDate`，`java.time.chrono.JapaneseDate`，`java.time.chrono.MinguoDate`，`java.time.chrono.ThaiBuddhistDate`; 通过HV另外的支持，如果[Joda Time]()的日期/时间API被引入类路径：那么任何`ReadablePartial`和`ReadableInstant`的实现类都会支持。Hibernate的元数据影响没有

- `@Max(value=)`

  检查注释的值是否小于或等于指定的最大值。支持的数据类型`BigDecimal`，`BigInteger`，`byte`，`short`，`int`，`long`和原始类型的相应的包装; 通过HV另外支持：任何的`CharSequence`（可以被转为数字的字符串），`Number`和`javax.money.MonetaryAmount`的子类型。Hibernate的元数据影响：添加列检查约束

- `@Min(value=)`

  检查注释的值是否大于或等于规定的最小值。支持的数据类型`BigDecimal`，`BigInteger`，`byte`，`short`，`int`，`long`和原始类型的相应的包装; 通过HV另外支持：任何的`CharSequence`（可以被转为数字的字符串），`Number`和`javax.money.MonetaryAmount`的子类型。Hibernate的元数据影响：添加列检查约束

- `@NotBlank`

  检查该被注解的字符序列不为null且长度是大于0。与`@NotEmpty`不同的是，这个约束只可用于字符序列，并且尾部空格都被忽略。支持的数据类型`CharSequence`。 Hibernate的元数据影响没有

- `@NotEmpty`

  检查注释元素是否不为null，也不是空字符串。支持的数据类型`CharSequence`，`Collection`，`Map`和数组。Hibernate的元数据影响没有

- `@NotNull`

  检查批注的值不为 `null`。支持的数据类型：任何类型。Hibernate的元数据影响：定义列不可为空

- `@Negative`

  检查该元素是严格为负。零值被视为非法。支持的数据类型`BigDecimal`，`BigInteger`，`byte`，`short`，`int`，`long`和原始类型的相应的包装; 通过HV另外支持：任何`CharSequence`（可以被转为数字的字符串），任何的`Number`和`javax.money.MonetaryAmount`的子类型。Hibernate的元数据影响没有

- `@NegativeOrZero`

  检查该元素是负的或零。支持的数据类型`BigDecimal`，`BigInteger`，`byte`，`short`，`int`，`long`和原始类型的相应的包装; 通过HV另外支持：任何`CharSequence`（可以被转为数字的字符串），任何的`Number`和`javax.money.MonetaryAmount`的子类型。Hibernate的元数据影响没有

- `@Null`

  检查批注的值是 `null`。支持的数据类型：任何类型。Hibernate的元数据影响没有

- `@Past`

  检查注释的日期是否过去。支持的数据类型`java.util.Date`，`java.util.Calendar`，`java.time.Instant`，`java.time.LocalDate`，`java.time.LocalDateTime`，`java.time.LocalTime`，`java.time.MonthDay`，`java.time.OffsetDateTime`，`java.time.OffsetTime`，`java.time.Year`，`java.time.YearMonth`，`java.time.ZonedDateTime`，`java.time.chrono.HijrahDate`，`java.time.chrono.JapaneseDate`，`java.time.chrono.MinguoDate`，`java.time.chrono.ThaiBuddhistDate`; 通过HV此外支持，如果[Joda Time]()的日期/时间API被引入类路径：那么任何`ReadablePartial`和`ReadableInstant`的实现类都会支持。Hibernate的元数据影响没有

- `@PastOrPresent`

  检查注释的日期是否是在过去或当前日期。支持的数据类型`java.util.Date`，`java.util.Calendar`，`java.time.Instant`，`java.time.LocalDate`，`java.time.LocalDateTime`，`java.time.LocalTime`，`java.time.MonthDay`，`java.time.OffsetDateTime`，`java.time.OffsetTime`，`java.time.Year`，`java.time.YearMonth`，`java.time.ZonedDateTime`，`java.time.chrono.HijrahDate`，`java.time.chrono.JapaneseDate`，`java.time.chrono.MinguoDate`，`java.time.chrono.ThaiBuddhistDate`; 通过HV此外支持，如果[Joda Time]()的日期/时间API被引入类路径：那么任何`ReadablePartial`和`ReadableInstant`的实现类都会支持。Hibernate的元数据影响没有

- `@Pattern(regex=, flags=)`

  如果被注解的字符串匹配正则表达式检查。 支持的数据类型`CharSequence`。 Hibernate的元数据影响没有

- `@Positive`

  检查该元素是严格为正。零值被视为非法。支持的数据类型`BigDecimal`，`BigInteger`，`byte`，`short`，`int`，`long`和原始类型的相应的包装; 通过HV另外支持：任何`CharSequence`（可以被转为数字的字符串），任何的`Number`和`javax.money.MonetaryAmount`的子类型。Hibernate的元数据影响没有

- `@PositiveOrZero`

  检查该元件是正数或零。支持的数据类型`BigDecimal`，`BigInteger`，`byte`，`short`，`int`，`long`和原始类型的相应的包装; 通过HV另外支持：任何`CharSequence`（可以被转为数字的字符串），任何的`Number`和`javax.money.MonetaryAmount`的子类型。Hibernate的元数据影响没有

- `@Size(min=, max=)`

  检查已注释元素的大小在`min`和`max`（含）之间。支持的数据类型`CharSequence`，`Collection`，`Map`和数组。Hibernate的元数据影响：定义列长度将被设置为 `max`

| **   | 上面列出的每个约束都至少有3个参数：message，groups，payload。这是Bean验证规范的要求。 |
| ---- | ---------------------------------------- |
|      |                                          |

#### 2.3.2。 附加约束

除了由Bean验证API规范定义的约束，Hibernate验证提供了在下面列出了一些有用的自定义约束。这些约束适用于字段/属性的水平，仅`@ScriptAssert`是一个类级别的约束。

- `@CreditCardNumber(ignoreNonDigitCharacters=)`

  判断被注解的字符序列是否通过卢恩校验和测试的检查。注意，这种验证的目的是检查用户的错误，而不是信用卡的有效性！又见[解剖学信用卡号码]()。`ignoreNonDigitCharacters`允许忽略非数字字符。默认值是`false` 。

  支持的数据类型`CharSequence`。 

  Hibernate的元数据影响没有

- `@Currency(value=)`

  检查该注解值的货币单位。`javax.money.MonetaryAmount`是指定的货币单位的一部分。

  支持的数据类型任何的`javax.money.MonetaryAmount`的子类型（如果[JSR 354 API]()或其实现在项目路径中）。

  Hibernate的元数据影响没有

- `@DurationMax(days=, hours=, minutes=, seconds=, millis=, nanos=, inclusive=)`

  该被注解的`java.time.Duration`元素不大于从注解参数构成的一个过程范围。如果相等是允许的，则`inclusive`标志设置为`true`。

  支持的数据类型`java.time.Duration`。 

  Hibernate的元数据影响没有

- `@DurationMin(days=, hours=, minutes=, seconds=, millis=, nanos=, inclusive=)`

  该被注解的`java.time.Duration`元素不小于于从注解参数构成的一个过程范围。如果相等是允许的，则`inclusive`标志设置为`true`。

  支持的数据类型`java.time.Duration`。 

  Hibernate的元数据影响没有

- `@EAN`

  检查批注的字符序列是一个有效的[EAN](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.co.uk&sl=en&sp=nmt4&tl=zh-CN&u=http://en.wikipedia.org/wiki/International_Article_Number_%2528EAN%2529&usg=ALkJrhhu4j_3PTtfRnDuhHEepZysGKGc5w)条码。类型决定条形码类型。默认为EAN-13。

  支持的数据类型`CharSequence`。 

  Hibernate的元数据影响没有

- `@Length(min=, max=)`

  验证该注释字符序列长度在`min`和`max`之间， 包含最大最小值。支持的数据类型`CharSequence`。 Hibernate的元数据影响：定义列长度将被设置为最大值

- `@CodePointLength(min=, max=, normalizationStrategy=)`

  验证注释字符序列的代码点长度在`min`和`max`之间，包括最大最小值。验证标准值由`normalizationStrategy`设定。

  支持的数据类型`CharSequence`。

  Hibernate的元数据影响没有

- `@LuhnCheck(startIndex= , endIndex=, checkDigitIndex=, ignoreNonDigitCharacters=)`

  检查该注释的字符序列中的数字传递卢恩校验和算法（也参见[Luhn算法](http://en.wikipedia.org/wiki/Luhn_algorithm)）。`startIndex`和`endIndex`允许仅在指定范围内的子字符串校验。`checkDigitIndex`允许于该字符序列作为校验位中使用的任意的数字。如果没有指定假设校验码在指定范围的一部分。最后但并非最不重要的，`ignoreNonDigitCharacters`可以忽略非数字字符。

  支持的数据类型`CharSequence`。

  Hibernate的元数据影响没有

- `@Mod10Check(multiplier=, weight=, startIndex=, endIndex=, checkDigitIndex=, ignoreNonDigitCharacters=)`

  检查该注释的字符序列中的数字传递的一般模10校验和算法。`multiplier`确定用于奇数乘法器（默认为3），`weight`为偶数（默认为1）的重量。`startIndex`并`endIndex`允许运行仅在指定的子串的算法。`checkDigitIndex`允许于该字符序列作为校验位中使用的任意的数字。如果没有指定假设校验码在指定范围的一部分。最后但并非最不重要的，`ignoreNonDigitCharacters`可以忽略非数字字符。

  支持的数据类型`CharSequence`

  Hibernate的元数据影响没有

- `@Mod11Check(threshold=, startIndex=, endIndex=, checkDigitIndex=, ignoreNonDigitCharacters=, treatCheck10As=, treatCheck11As=)`

  检查该注释的字符序列中的数字传递模11校验和算法。`threshold`指定用于MOD11乘数生长的阈值; 如果没有指定值的乘数将无限增长。`treatCheck10As`和`treatCheck11As`指定的检验数字时要使用的模11的校验和等于10或11，分别。默认为X和0，分别。`startIndex`，`endIndex` `checkDigitIndex`并`ignoreNonDigitCharacters`携带相同的语义`@Mod10Check`。

  支持的数据类型`CharSequence`

  Hibernate的元数据影响没有

- `@Range(min=, max=)`

  检查注解值是否位于（含）指定的最小值和最大值之间。

  支持的数据类型`BigDecimal`，`BigInteger`，`CharSequence`，`byte`，`short`，`int`，`long`和原始类型的相应的包装。

  Hibernate的元数据影响没有。

- `@SafeHtml(whitelistType= , additionalTags=, additionalTagsWithAttributes=, baseURI=)`

  检查注释的值是否包含潜在的恶意片段如`<script/>`。为了使用此约束，jsoup库必须在类路径中。通过`whitelistType`预定义的白名单类型和`additionalTags`或`additionalTagsWithAttributes`可以精确控制校验过程。前者允许添加标签没有任何属性，而后者则允许指定标签和可选允许的属性以及使用注解属性接受的协议`@SafeHtml.Tag`。此外，`baseURI`允许指定URI用于解析相对URI的基础。

  支持的数据类型`CharSequence`。 

  Hibernate的元数据影响没有

- `@ScriptAssert(lang=, script=, alias=, reportOn=)`

  检查是否给定的脚本可以成功地对注解的元素进行评估。为了使用此约束，在Java脚本API的实现如由JSR 223（“脚本用于Java定义TM平台”）必须是类路径的一部分。待评估的表达式可以写成任何脚本或表达式语言，其中JSR 223兼容引擎可以在类路径上找到。虽然这是一类级别的约束，可以使用`reportOn`属性上的特定属性，而不是整个对象报告约束冲突。

  支持的数据类型任何类型。

  Hibernate的元数据影响没有

- `@UniqueElements`

  检查批注的集合只包含独特的元素。判断元素是否相等使用的是`equals()`方法。默认的消息不包含重复元素的列表，但是你可以通过重写信息并使用它包含`{duplicates}`消息参数。也包括在违反约束的动态负载重复元素的列表。

  支持的数据类型`Collection`。 

  Hibernate的元数据影响没有

- `@URL(protocol=, host=, port=, regexp=, flags=)`

  检查注释的字符序列是根据RFC2396有效的URL。如果任何可选参数`protocol`，`host`或`port`指定时，对应的URL片段必须在指定的值相匹配。可选参数`regexp`和`flags`允许指定附加的正则表达式（包括正则表达式标志），其的URL必须匹配。每默认此约束使用的`java.net.URL`构造函数来验证一个给定的字符串是否代表一个合法的URL。正则表达式基于版本也可用- `RegexpURLValidator`-其可以通过XML来配置（参见[第8.2节“经由映射约束`constraint-mappings`”]()）或编程API（见[第12.13.2“添加约束定义编程”]()）。

  支持的数据类型`CharSequence`。 

  Hibernate的元数据影响没有

##### 2.3.2.1。具体国家限制

Hibernate验证还提供了一些国家的具体限制，例如对于社会安全号码的验证。

| **   | 如果你必须实现一个国家相关的特定的约束，可以考虑向Hibernate Validator提出贡献。 |
| ---- | ---------------------------------------- |
|      |                                          |

- `@CNPJ`

  批注的字符序列代表的巴西企业纳税人登记号检查（Cadastro德佩索阿Juríeddica）支持的数据类型`CharSequence`Hibernate的元数据影响没有国家巴西

- `@CPF`

  批注的字符序列代表巴西个体纳税人登记号检查（Cadastro德佩索阿Fídsica）支持的数据类型`CharSequence`Hibernate的元数据影响没有国家巴西

- `@TituloEleitoral`

  被注解字符序列代表巴西选举人ID卡号码检查（[性标题Eleitoral](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.co.uk&sl=en&sp=nmt4&tl=zh-CN&u=http://www.exceldoseujeito.com.br/2008/12/19/validar-cpf-cnpj-e-titulo-de-eleitor-parte-ii/&usg=ALkJrhhHHVCUUpeVpVzAxXIwMH7v3Drrng)）支持的数据类型`CharSequence`Hibernate的元数据影响没有国家巴西

- `@NIP`

  检查该注释字符序列代表波兰VAT标识号（[NIP](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.co.uk&sl=en&sp=nmt4&tl=zh-CN&u=https://pl.wikipedia.org/wiki/NIP&usg=ALkJrhhg0v-xCoR1qFGHD9kzAkmmyUWHUw)）支持的数据类型`CharSequence`Hibernate的元数据影响没有国家波兰

- `@PESEL`

  检查批注的字符序列代表波兰国家识别号码（[PESEL](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.co.uk&sl=en&sp=nmt4&tl=zh-CN&u=https://pl.wikipedia.org/wiki/PESEL&usg=ALkJrhjM6oo5wSOngPLa9fFtNDXWwLG7Mg)）支持的数据类型`CharSequence`Hibernate的元数据影响没有国家波兰

- `@REGON`

  检查该注释字符序列代表一个波兰纳税人识别号（[REGON](https://translate.googleusercontent.com/translate_c?depth=1&hl=zh-CN&ie=UTF8&prev=_t&rurl=translate.google.co.uk&sl=en&sp=nmt4&tl=zh-CN&u=https://pl.wikipedia.org/wiki/REGON&usg=ALkJrhhE9cpJ9MqZ2Kw0wVL02RJHEZO4Tg)）。可同时适用于9个14位版本的REGON支持的数据类型`CharSequence`Hibernate的元数据影响没有国家波兰

| **   | 在某些情况下，无论是Bean验证约束，还输Hibernate验证所提供的自定义约束，都不能满足您的要求。在这种情况下，你可以很容易地编写你自己的约束。你可以找到更多信息[第6章，*创建自定义的约束*]()。 |
| ---- | ---------------------------------------- |
|      |                                          |

