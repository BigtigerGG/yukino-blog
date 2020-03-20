---
title: Java NIO
date: 2020-03-16 22:01:49
tags:
- Java
categories:
- Java
- Java SE
---

JDK 1.4 引进了新的 Java I/O 库，目的在于提高速度。同时，旧 I/O 的一部分也用 NIO 重新实现过，所以旧 I/O 也从中收益，即便没有使用新 I/O。  

## 通道和缓冲器  

新的 I/O 之所以提升了速度，是因为使用了更加接近操作系统执行 I/O 的方式。即**通道和缓冲器**。这两个概念可以这么理解：  

*通道* 即将输入输出源和程序连接起来，我们将它抽象成一道路。在 Java 中用 Channel 类来表示。程序不会直接和通道交互，而是使用 *缓冲器* 来运送数据，缓冲期在通道上将数据获取或者写入，数据经由通道与源进行交互。  

唯一能和通道交互的缓冲器是 `ByteBuffer` ，也就是说，通道上面只能传送字节数据。   

### 通道的基本使用
使用的最多的通道应该是 `FileChannel` 了，它是一种文件通道，用于和文件数据交互。通常可以从旧 I/O 中获取文件通道：  

```java
    FileChannel fsi = new FileInputStream("./demo.txt").getChannel();
    FileChannel fso = new FileOutputStream("./demo.txt").getChannel();
    FileChannel fsra = new RandomAccessFile("./demo.txt").getChannel();
```

通过使用 `getChannel` 方法来获取对应 I/O 的文件通道。文件通道可以通过使用 `read(ByteBuffer)`, `write(ByteBuffer)` 获取（放上）字节缓冲器来进行输入输出。这就是通道的基本用法。  

### 缓冲器的数据结构

缓冲器由数据和可以高效访问这些数据的4个索引组成，这四个索引分别是：  

1. mark 标记  
    用于给位置留下标记，便于位置信息的保存。可以使用 `mark()` 方法将当前的 position 设置为 mark。同时也可通过 `reset()` 方法将 position 还原为 mark 的值。  
2. position 位置  
    用于表示当前的缓冲器位置，当读取或者写入的时候，这个值会相应的改变
3. limit 界限  
    标识了缓冲器能读取或者能写入的界限，position 的值永远不会超过 limit 的值，否则将会抛出 BufferUnderflowException
4. capacity 容量  
    标识了缓冲器的最大容量。容量在初始化这个缓冲器的时候就被定义好了，之后不能更改。

### 缓冲器的基本使用方法  

1. `get()`   
    将一个字节数据从缓冲器当前的位置读取出来，position 加一。

2. `put()`  
    将一个字节数据写入到当前的缓冲器位置，position 加一。

> 这两个方法还有一系列的重载，详细内容请参考 JDK 文档

3. `filp()`  
    当从通道中读取字节数据到缓冲器中，一旦准备好要读取这个缓冲器中的值的时候，应该立即使用：`flip()` 方法让这个缓冲器做好被读取的准备。这个方法会将 limit 索引设置成当前的 position 值，并且 position 会被设置为 0。这样这个缓冲器被读取的时候，就只会被读取到 limit-position 之间的数据，防止数据被错误读取。  
4. `ByteBuffer.wrap()` (这是一个静态方法) 
    将一个字节缓冲器直接包装到通道里面, 缓冲器中的索引为自动设置好，所以不用调用 `flip()`
5. `ByteBuffer.allocate(int), ByteBuffer.allocateDirect(int)`  
    返回一个分配了指定长度的字节缓冲器，后者是“直接的”缓冲器，与操作系统更加耦合，速度也更快。

下面是缓冲器从通道中读取或者写入的范例：  

```java
    // 写入
    FileChannel fso = new FileOutputStream("./demo.txt").getChannel();
    fso.write(ByteBuffer.wrap("hello world".getBytes()));
    fso.close();
    // 或者这样写
    // ByteBuffer buf = ByteBuffer.allocate(1024);
    // buf.put("hello world".getBytes());
    // buf.flip();
    // fso.write(buf);
    // fc.close()

    // 读取
    FileChannel fsi = new FileInputStream("./demo.txt").getChannel();
    ByteBuffer buf = ByteBuffer.allocate(1024);
    fsi.read(buf);
    buf.flip();
    while(buf.hasRemaining()){
        System.out.print((char)buf.get());
    }
```

### 视图缓冲器  

视图缓冲器可以通过某个特定的基本数据类型的视窗查看其底层的字节缓冲器。这相当于在字节缓冲器上面添加了一个装饰器，我们可以直接通过这些视图缓冲器通过操纵基本数据类型来映射到字节缓冲器的改变。  

最常用的视图缓冲器是 CharBuffer ，通过它可以直接操作字符来达到对字节缓冲器的控制。当我们从通道中获取字符类型的数据的时候，需要有一个字节数据到字符数据的转换。而有了字符缓冲器，我们可以直接获取字节缓冲器中的字符型数据。例如上面的读文件操作我们可以这样写：  

```java
    System.out.println(buf.asCharBuffer()); 
    // CharBuffer 重写了toString()方法，可以直接输出字节缓冲其中的所有字符串
```

## 内存映射 MappedByteBuffer  

当想读取的文件太大，我们不能将一整个文件全部读取到内存中。这时候可以使用字节映射缓冲器。这个缓冲器通过文件通道获取，获取的方法为：  

```java
    MappedByteBuffer out = new RandomAccessFile("./demo.txt", "rw")
        .getChannel()
        .map(FileChannal.MapMode.READ_WRITE, 0, 0x8FFFFF);
    out.put((byte)'x'); // 这里直接写入了文件，不需要通道的写方法    
```

## 文件加锁  

在 Java 中，我们可以使用文件锁，让一个文件同步访问。但是，竞争这个锁的两个线程可能不再同一个 JVM 上面。或者是一个 Java 线程，另一个是操作系统中的其他的某个本地线程。文件锁对于其他操作系统进程是可见的，Java 的文件锁直接映射到本地系统的加锁工具。基本使用如下：  

```java
    FileOutputStream fout = new FileOutputStream("./demo.txt");
    FileLock fl = fout.getChannel().tryLock();
    if (fl != null) {
        System.out.println("Locked file!");
        TimeUnit.MILLISECONDS.sleep(1000);
        fl.release();
        System.out.println("Released Lock!");
    }
    fout.close();
```

可以看到，使用文件通道来调用 `tryLock()` 方法来尝试对文件进行加锁。如果成功获取了锁，则会返回一个 `FileLock` 对象，如果没有获得锁，则返回 `null` 。

`tryLock()` 方法是非阻塞式的，当调用这个方法会尝试获取文件的锁，他会尝试去获取文件锁。与之对应的是 `lock()` 方法，这个方法是阻塞式的，它会设法获取文件的锁，如果无法获得，它会一直阻塞当前线程直到能得到，或者将这个线程中断，又或者调用这个方法的通道关闭。  

使用 `FileLock` 对象的 `release()` 方法可以释放锁。  

这两个方法还有一个重载形式用于对文件的某一部分加锁。详细请参考 JDK 文档。