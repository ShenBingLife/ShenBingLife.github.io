---
layout: post
title: ServiceLoader
category: Java SPI机制：ServiceLoader
tags: 
    - Java
---

## SPI
SPI ，全称为 Service Provider Interface，是一种服务发现机制。它通过在ClassPath路径下的META-INF/services文件夹查找文件，自动加载文件里所定义的类。

这一机制为很多框架扩展提供了可能，比如在Dubbo、JDBC、SpringBoot中都使用到了SPI机制。

用法：通过SPI，我们可以动态的根据导入的jar包，发现需要的接口实现类，尤其是策略模式下。
无需手动指定接口的实现类，通过ServiceLoader可以自动选择实现类并创建实例对象。
<!-- more --> 
## 使用

1. 定义接口
    ```java
    public interface SPIService {
        void execute();
    }
    ```
2. 定义实现类
    ```java
    public class SpiImpl1 implements SPIService{
        public void execute() {
            System.out.println("SpiImpl1.execute()");
        }
    }
    ```
3. 创建配置文件   
    在src/main/resources文件夹内创建META-INF/services/xxx.SPIService 文件  
    在文件内写入子类的全路径类名
    ```
    xxx.SpiImpl1
    ```


## 源码中的懒加载

ServiceLoader 内部使用了懒加载模式，只有当需要获取对象时，
会调用内部的LazyIterator类的方法去寻找和加载发现的类。  
查找实现类和创建实现类的过程，都在LazyIterator完成。当我们调用iterator.hasNext和iterator.next方法的时候，实际上调用的都是LazyIterator的相应方法。
```java
public final class ServiceLoader<S> implements Iterable<S> {
    private ServiceLoader(Class<S> svc, ClassLoader cl) {
            //要加载的接口
            service = Objects.requireNonNull(svc, "Service interface cannot be null");
            //类加载器
            loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
            //访问控制器
            acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
            //先清空
            providers.clear();
            //实例化内部类 
            LazyIterator lookupIterator = new LazyIterator(service, loader);
        }

    //对类的查询都是通过LazyIterator代理
    public Iterator<S> iterator() {
        return new Iterator<S>() {
            public boolean hasNext() {
                return lookupIterator.hasNext();
            }
            public S next() {
                return lookupIterator.next();
            }
        };
    }
}

```

分析LazyIterator的hasNextService方法，只有在第一次时，解析出pending对象获取所有的类，后续都是读取pending对象。
```java
private class LazyIterator implements Iterator<S>{
    Class<S> service;
    ClassLoader loader;
    Enumeration<URL> configs = null;
    Iterator<String> pending = null;
    String nextName = null; 
    private boolean hasNextService() {
        //第二次调用的时候，已经解析完成了，直接返回
        if (nextName != null) {
            return true;
        }
        if (configs == null) {
            //META-INF/services/ 加上接口的全限定类名，就是文件服务类的文件
            //META-INF/services/com.viewscenes.netsupervisor.spi.SPIService
            String fullName = PREFIX + service.getName();
            //将文件路径转成URL对象
            configs = loader.getResources(fullName);
        }
        while ((pending == null) || !pending.hasNext()) {
            //解析URL文件对象，读取内容，最后返回
            pending = parse(service, configs.nextElement());
        }
        //拿到第一个实现类的类名
        nextName = pending.next();
        return true;
    }
}
```

## JDBC示例
```java
public class DriverManager {
    static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }
}
```

阅读loadInitialDrivers方法，它在里面查找的是Driver接口的服务类，所以它的文件路径就是：META-INF/services/java.sql.Driver

```java
public class DriverManager {
    private static void loadInitialDrivers() {
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
                //很明显，它要加载Driver接口的服务类，Driver接口的包为:java.sql.Driver
                //所以它要找的就是META-INF/services/java.sql.Driver文件
                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator<Driver> driversIterator = loadedDrivers.iterator();
                try{
                    //查到之后创建对象
                    while(driversIterator.hasNext()) {
                        driversIterator.next();
                    }
                } catch(Throwable t) {
                    // Do nothing
                }
                return null;
            }
        });
    }
}
```

同时，com.mysql.cj.jdbc.Driver在DriverManager注册时，也完成了一件事，就是向DriverManager注册自己。
```java
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    static {
        try {
            //注册
            //调用DriverManager类的注册方法
            //往registeredDrivers集合中加入实例
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }
    public Driver() throws SQLException {
        // Required for Class.forName().newInstance()
    }
}
```





