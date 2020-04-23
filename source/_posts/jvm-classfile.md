---
title: jvm-类文件
date: 2020-04-13 16:15:39
tags:
- JVM
categories:
- JVM
- ClassFile
---

## 概述 

我们知道，Java 语言是通过 Java 编译器编译成字节码文件（扩展名为 .class) 然后通过让 JVM 运行这个编译生成的字节码文件来运行程序。这个字节码文件只针对 JVM，他不面向任何处理器。很多运行在 JVM 上的语言，例如 Lisp、Groovy、Scala 等都是通过相应的编译器编译成字节码文件然后运行在 JVM 上的，所以 .class 文件是不同基于 JVM 语言的桥梁，支撑着 Java 语言跨平台的特性。  

## Class 文件结构  

类文件的结构如下：  

```java
// u 代表一个字节，例如 u4 代表4个字节
ClassFile {
    u4             magic; //Class 文件的标志
    u2             minor_version;//Class 的小版本号
    u2             major_version;//Class 的大版本号
    u2             constant_pool_count;//常量池的数量
    cp_info        constant_pool[constant_pool_count-1];//常量池
    u2             access_flags;//Class 的访问标记
    u2             this_class;//当前类
    u2             super_class;//父类
    u2             interfaces_count;//接口
    u2             interfaces[interfaces_count];//一个类可以实现多个接口
    u2             fields_count;//Class 文件的字段属性
    field_info     fields[fields_count];//一个类会可以有个字段
    u2             methods_count;//Class 文件的方法数量
    method_info    methods[methods_count];//一个类可以有个多个方法
    u2             attributes_count;//此类的属性表中的属性数
    attribute_info attributes[attributes_count];//属性表集合
}
```  

## 类加载过程  

### 类的生命周期  

*加载 -> 连接 -> 初始化 -> 使用 -> 卸载*，其中连接的过程需要经过 *验证->准备->解析* 三个过程  

所有的类都由类加载器加载，加载的作用是将 .class 文件从磁盘加载到内存。  

JVM 内置了三个最重要的类加载器，它们分别是：  

1. BootstrapClassLoader（启动类加载器）：最顶层的加载类，由 C++ 实现，负责加载 %JAVA_HOME%/lib 目录下面的 jar 包或者被 -Xbootclasspath 参数指定的路径中的所有类文件。  
2. ExtensionClassLoader（扩展类加载器）：负责加载目录 %JRE_HOME%/lib/ext 下的所有 jar 包和类文件，或者被 java.ext.dirs 系统变量（这个系统是 JVM）指定路径下的 jar 包。
3. AppClassLoader（应用程序加载器）：面向用户的加载器，负责加载当前应用 classpath 下的所有 jar 包和类文件。  

### 双亲委派模型  

系统中的 ClassLoader 在加载类的时候会使用 **双清委派模型** 进行协同工作。原理是：  

在类加载的时候，系统会首先判断该类是否已经被加载，已经被加载的类会直接返回，否则才尝试进行加载。加载的时候会将该加载的请求委派给这个加载类的父类来加载，也就是由父类加载器的 `loadClass()` 方法来加载，如果父类加载器无法处理的时候，由自己来处理。当父类加载器为 `null` 的时候，会使用 `BootstrapClassLoader` 作为其父类加载器。  

双亲委派模型可以抽象为：  

![双亲委派模型](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/classloader_WPS%E5%9B%BE%E7%89%87.png)  

每一个类加载器都有一个父类加载器，其中 AppClassLoader 的父加载器是 ExtClassLoader，ExtClassLoader 的父加载器是 null，但是这并不代表它没有父加载器，它的父加载器是 BootstrapClassLoader。  

关于 “双亲” 的理解：  
一般地，所谓的双亲都是指父母，这里并不是指这个加载器类有一个“父加载器”和“母加载器”。这里的双亲更多的是指这个加载器的“父母这一辈”而已。另外，他们的父子关系也不是由继承来实现，而是由“优先级”来定义。  

双亲委派的源码分析：  

```java
    private final ClassLoader parent; 
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 首先，检查请求的类是否已经被加载过
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {//父加载器不为空，调用父加载器loadClass()方法处理
                        c = parent.loadClass(name, false);
                    } else {//父加载器为空，使用启动类加载器 BootstrapClassLoader 加载
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                   //抛出异常说明父类加载器无法完成加载请求
                }
                
                if (c == null) {
                    long t1 = System.nanoTime();
                    //自己尝试加载
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

双亲委派模型的好处：  

1. 保证了 Java 程序的稳定运行，避免类的重复加载（JVM 区分不同类的方式不仅仅根据类的全限定名，不同的类加载器加在同一个类文件产生的是两个类）。  
2. 保证了 Java 的核心 API 不被篡改。