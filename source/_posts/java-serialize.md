---
title: Java 序列化与反序列化
date: 2020-03-19 12:26:41
tags:
- Java
categories:
- Java
- Java SE
---

通常，程序中的对象是保存在内存中的，它的生命周期最多持续到程序结束。如果我们想要让一个对象 ”持久化“ 存在，可以使用 Java 的对象 *序列化* 机制，将对象的状态保存到磁盘中，当程序结束的时候，我们可以在下一次的访问中，将这个对象恢复。  

Session 对象通常就是需要序列化的，它是某一个用户访问服务器的会话对象，服务器根据这个会话对象来保证用户的访问权限。  

## ObjectInputStream 和 ObjectOutputStream  

Java 主要通过两个流来控制对象的序列化和反序列化。如果一个对象要能够正常序列化，那么它必须实现 `Serializable` 接口，这个接口只是一个标志，没有任何需要实现的方法。 下面是基本的用例：  

```java
    // 假设 DemoClass 已经实现了 Serializable 接口
    ObjectOutputStream out = new ObjectOutputStream(
        new FileOutputStream("./demo.txt"));
    DemoClass demo = new DemoClass();
    demo.setName("John");
    out.writeObject(demo);
    ObjectInputStream in = new ObjectInputStream(
        new FileInputStream("./demo.txt"));
    demo = in.readObject();
    System.out.println(demo.getNmae());
    // expected output : "John"
```

序列化机制是通过将对象映射成一串字节，这串字节代表了这个对象被序列化的哪一个瞬间的状态。当这个对象被反序列化的时候，对象的那个状态会被还原。  

需要注意的是，对象的反序列化可能会发生 `ClassNotFoundException`，这是因为反序列化的时候，JVM 会查找是否加载了这个对象的类的 Class 对象，如果没有加载，那么就会抛出这个运行时异常。  

## 序列化的控制  

如果我们想要控制一个对象的成员是否应该被序列化的时候，例如一个用户对象的密码，他不应该被序列化到文件中，我们可以控制对象的序列化。  

方法有3个：  

1. 让这个类实现 Externalizable 接口。  
    `Externalizable` 接口继承自 `Serializable` 接口，所以实现了这个接口也就相当于实现了 `Serializable` 接口。并且重写这个接口的方法：`readExternal(ObjectInput in)`, `writeExternal(ObjectOutput out)`。  
    在这两个方法中你可以完全控制哪些成员应该被被序列化。需要注意的是，这个类的构造方法必须是 `public` 的，因为在反序列化的过程，Java 会利用反射调用这个类的所有默认构造器，需要重新 new 一个该类的对象， 如果不是 public 的，那么会抛出一个异常。  
    与通常的序列化不同，这种序列化得到的对象不是同一个对象，也就是说它们的hashcode 不一样。  

2. 让这个类不需要被序列化的字段用 `transient` 关键字修饰  
    用 `transient` 修饰的成员在序列化的过程中，这个字段的不会被序列化。  

3. 添加（不是重写，不是重载）`readObject()`, `writeObject()` 方法  
    这里的两个方法不是重写，而且必须满足下面的函数签名：  

    ```java
    private void readObject(ObjectInputStream stream) throws Exception;
    private void writeObject(ObjectOutputStream stream) throws Exception;
    ```
    在这两个方法中我们可以自己定义对象成员的序列化。这种方法的原理是反射机制，Java 在调用 `ObjectInputSteram.readObject()` 的时候，通过反射调用所传递的 `Serializable` 对象，`writeObject()` 同理。  
 