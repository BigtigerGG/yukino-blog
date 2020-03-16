---
title: Java I/O
date: 2020-03-15 17:10:49
tags:
- Java
categories:
- Java
- Java SE
---

​	Java I/O 系统是用来处理 Java 程序中的各种输入/输出问题。Java 标准库中有大量的类用来处理输入输出，本文就我的学习情况对一些关键的 I/O 使用进行总结。  

## File 类

File （文件）类并不是字面意义上指的某一个文件，它既能代表一个特定文件的名称，又能代表一个目录下的一组文件的名称。如果它指的是一个文件集，也就是说是一个文件夹，那么可以对这个 File 对象调用 `list()` 方法，这个方法会返回一个字符数组。  

### 使用 FilenameFilter 接口对目录下的文件进行过滤  

我们可以使用 `list()` 方法获取一个是目录的文件的子文件，他会返回所有的子文件列表，当我们想要获取一个指定规则的子文件列表的时候，可以在该方法传入一个 `FilenameFilter` 接口的实现类，或者 lambada，用来过滤这个列表。

`FilenameFilter` 接口是一个函数式接口，所以我们可以传入一个 lambada 表达式。这个接口只有一个 `accept()` 方法，用来在 `list()` 方法中回调。

`accept()` : `public boolean accept(File dir, String name)`
`  

例如，当前目录的子文件有：a.java, b.py, c.cpp, 我们想要在当前目录中获取以 .java 为后缀的文件名。代码如下：  

```java
File currentDir = new File(".");
Pattern pattern = Pattern.compile(".+\\.java");
String[] filteredSubFiles = currentDir.list(
    (dir, name)->pattern.matcher(name).matches());
for(String s : filteredSubFiles){
    System.out.println(s)
}
// expected output:
// a.java
```
## 目录的检查和创建  

File 对象有很多实用的方法，可以用 `file.isDirectory()`, `file.isFile()` 来检查这个路径是一个文件还是一个目录，用 `file.exists()` 检查是否存在，用 `file.getAbsolutePath()` 获取它的绝对路径，用 `file.mkdirs()` 创建这个路径下的目录等等。通过 File 类，我们可以对系统的文件进行各种各样的操作。  

## InputStream 类和 OutputStream 类  

这两个类作为输入输出的基本类，提供了流操作的基本方法。它面向字节流，将数据的字节抽象为字节流，对数据进行输入和输出。  

任何数据在计算机中的表示都是一个个的字节。因此，可以通过这两个类，对各种数据进行底层的操作。包括文本文件，图片文件，视频文件等等，它们都是由字节构成的数据。  

为了方便的操作各种各样的数据，Java 从这两个类中派生出一系列操作某一类数据的派生类以提供对这些数据更加具体的输入和输入。这些数据源包括：  

1. 字节数组，对应的类：ByteArrayInputStream（OutPutStream）
2. 字符串对象，对应的类：StringBufferInputStream（OutPutStream）,最新的 jdk 不再推荐被使用
3. 文件，对应的类：FileInputStream（OutPutStream）
4. 管道，对应的类：PipedInputStream（OutPutStream）
5. 由其他流组成的序列（便于组合为一个流）：SequenceInputStream（OutPutStream）
6. 其他数据源：例如网络流，对网络进行支持。  

这些流对象使用的最多的就是文件流，文件流对象将文件的数据抽象为字节流，对文件进行字节上的输入和输出。  

### 针对 InputStream 和 OutputStream 的装饰器类

Java 的 I/O 系统使用装饰器模式，为输入输出流提供更多的属性和接口。这些装饰器类都继承自：FilterInputStream（OutputStream），但是使用的最多的还是 BufferedInputStream（OutputStream），它可以让输入输出变得更加快速，通过缓冲区来降低流操作的性能损耗。  

用法：

```java
    BufferedInputStream bfi = new BufferedInputStream(new FileInputStream(new File("./bigFile.mp4")));
    byte[] buff = new byte[1024];
    bfi.read(buff, 0, 1024);
    // 此方法获取该文件的第一个KB的字节数组
    // 用 BufferedInputStream 可以提高输入输出的速度
```  
## Reader 和 Writer 类

与 InputStream 和 OutputStream 一样，Reader 和 Writer 类同样提供了 I/O 功能，区别在于 Reader 和 Writer 是面向**字符流**的。这使得 Reader 和 Writer 对 Unicode 提供了支持。  

Reader 和 Writer 的派生类都有 InputStream 和 OutputStream 的对应。但是它们对处理文件中的字符处理提供了更加完善的方法。所以，当处理字符的时候，应该使用 Reader 和 Writer 类。  

有些时候，需要让一个 InputStream 和 OutputStream 获得 Reader 和 Writer的特性的时候，我们可以使用两个适配器类：   

1. InputStreamReader
2. OutputStreamWriter

它们可以将作为构造参数的输出输出流转换成 Reader 和 Writer。

## 标准 I/O 流

标准输入输出流是指程序中所使用的单一信息流。也就是说，可以认为程序中的所有输入都来源于标准输入，程序中的所有输出都可以发送到标准输出。  

Java 中的标准 I/O 主要有一下3种：  

1. System.out
2. System.in
3. System.err

其中，标准输出和标准错误输出都是一个被包装好的对象，我们可以直接使用。标准输入只是一个 InputStream ，需要进一步的包装这个类才能有效的使用。  

标准输出和标准错误输出都是以控制台为输出源。标准输入默认以键盘输入作为默认输入源。  

### 标准输入输出的重定向

重定向是指改变标准输入输出的输入源和输出源。System 类提供了一系列静态方法以改变标准输入输出的源头。  

1. System.setIn(InputStream)
2. System.setOut(PrintStream)
3. System.(PrintStream) 

