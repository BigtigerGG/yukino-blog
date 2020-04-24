---
title: Java8 新特性总结
date: 2020-04-23 14:21:39
tags:
- Java
categories:
- Java
- Java8
---

现在大部分的 JDK 都是使用 JDK1.8，了解 Java8 的新特性是很有必要的。Java 8增加了很多语法糖，其中最重要的例如对 Lambda 语法的支持、函数式接口、Stream 等等。这篇文章结合我所学的对各种新特性进行总结。  

## Lambda 表达式和函数式接口  

熟悉 Javascript 的同学肯定知道，在 ES6 中增加了箭头函数，也就是我们所说的 Lambda 表达式。这种编程风格属于函数式编程，Java 8 中也对函数式编程提供了一系列的支持。箭头函数的概念和 C++ 中的函数指针有点类似，通过将函数作为引用（指针）传递给方法中的参数或者赋值给一个变量。Java中的箭头函数需要和函数式接口配合使用。  

函数式接口的概念是：一个接口如果只有一个抽象方法，但可以有多个非抽象方法（有 default 方法），那么这个接口就是一个函数式接口。  

我们可以将一个函数式接口通过箭头函数来实现它的抽象方法。基本用法如下：  

```java
// 这里首先定义一个函数式接口。
@FunctionalInterface
public interface Comparator {
   int compare(int t1, int t2);
}

public static void main(String[] args) {
    int num1 = 1, num2 = 2;
    // 这里将箭头函数赋值给一个函数式接口
    Comparator numCompare = (num1, num2) => {
        if(num1>num2) return 1;
        else if(num1<num2) return -1;
        else return 0;
    }
    System.out.println(numCompare.compare(num1, num2));
    foo(numCompare);
}

// 我们也可以在一个方法中接受一个函数式接口形参
public static void foo(Comparator comparator) {
    System.out.println(comparator.compare(1, 2));
}
```
## 方法和构造函数引用  

箭头函数可以以字面量的形式实现函数式接口。如果一个方法或者构造函数符合函数式接口的实现，那么我们可以将它的引用赋值给这个函数式接口。  

```java
// 这里假如 A 类中有一个符合上节的函数式接口的方法
public class A {
    int intCompare(int n1, int n2) {
        if(num1>num2) return 1;
        else if(num1<num2) return -1;
        else return 0;
    }

    public static void main(String[] args[]) {
        // 这里将 A 类中的 intComapre 方法赋值给 numCompare
        Comparator numCompare = A::intCompare;
        System.out.println(numCompare.compare(1, 2));
    }
}
```

正如代码中一样，Java 8 允许我们以 `::` 关键字传递方法或者构造函数的引用。关于构造函数，我们可以通过以 `Class::new` 的形式将一个符合条件的构造函数传递给一个工厂接口。如果一个类有多个构造方法，编译器会选择最合适的那一个作为引用传递。  

## 内建的函数式接口  

Java 8 中内建了很多函数式接口，同时之前很多诸如 Runnable, Comparator 等接口也加入了 `@FunctionalInterface` 的注解。这里介绍几个使用的比较多的内建函数式接口。  

1. Predicate（断言）  
这个接口是一个只有一个参数的返回布尔类型的 **断言式** 接口。该接口用于判断命题的真假，它内置了一系列默认方法用于组成复杂的与或非结构。  
该接口的源码如下:  

    ```java
    package java.util.function;
    import java.util.Objects;

    @FunctionalInterface
    public interface Predicate<T> {
        
        // 该方法是接受一个传入类型,返回一个布尔值.此方法应用于判断.
        boolean test(T t);

        //and方法与关系型运算符"&&"相似，两边都成立才返回true
        default Predicate<T> and(Predicate<? super T> other) {
            Objects.requireNonNull(other);
            return (t) -> test(t) && other.test(t);
        }
        // 与关系运算符"!"相似，对判断进行取反
        default Predicate<T> negate() {
            return (t) -> !test(t);
        }
        //or方法与关系型运算符"||"相似，两边只要有一个成立就返回true
        default Predicate<T> or(Predicate<? super T> other) {
            Objects.requireNonNull(other);
            return (t) -> test(t) || other.test(t);
        }
    // 该方法接收一个Object对象,返回一个Predicate类型.此方法用于判断第一个test的方法与第二个test方法相同(equal).
        static <T> Predicate<T> isEqual(Object targetRef) {
            return (null == targetRef)
                    ? Objects::isNull
                    : object -> targetRef.equals(object);
        }
    }  
    ```  
    这里是使用示例：  

    ```java
    public static void main(String[] args) {
        // 这里是 Predicates 的使用示例
        // Predicate 接口用于进行对象的判断
        Predicate<String> isNullString = String::isEmpty;
        // negate() 方法是该接口用于“非”运算的默认实现
        System.out.println(isNullString.negate().test(""));
        // and() 方法是该接口用于“与”运算的默认实现
        // 这里要求测试的字符串不是空串且以“https://”开头
        System.out.println(isNullString.negate().and(t->t.startsWith("https://")).test("https://www.baidu.com"));
        // or() 方法是该接口用于“或运算”
        // 这里要求测试的字符串是一个“”串或者仅有空格符的串
        System.out.println(isNullString.or(t->t.trim().equals("")).test(""));
    }
    ```  

2. Function（函数计算）  
这个接口接收一个参数并生成结果。它一系列默认方法可以将多个函数连接在一起。  
    下面是源码和解析：   

    ```java
    package java.util.function;
    import java.util.Objects;
    
    @FunctionalInterface
    public interface Function<T, R> {
        
        //将Function对象应用到输入的参数上，然后返回计算结果。
        R apply(T t);
        //将两个Function整合，并返回一个能够执行两个Function对象功能的Function对象。
        default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
            Objects.requireNonNull(before);
            return (V v) -> apply(before.apply(v));
        }
        // 让一个函数调用的结果作为另一个函数的参数并返回结果
        default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
            Objects.requireNonNull(after);
            return (T t) -> after.apply(apply(t));
        }
        
        static <T> Function<T, T> identity() {
            return t -> t;
        }
    }
    ```  
    用例： 

    ```java
    // Function 接口用于函数运算的连接组合运算
    Function<String, Integer> stringToInt = Integer::parseInt;
    System.out.println(stringToInt.apply("123") + 1);
    // compose() 方法用于将一个前置函数的返回值作为调用函数的参数进行运算
    // 这里是首先将字符串去掉首尾的空串，再进行数字转换
    System.out.println(stringToInt.compose(s -> s.toString().trim()).apply("   123   "));
    // andThen() 方法用于将调用函数的返回值作为后置函数的参数进行运算
    // 这里首先将字符串去掉首位空格，进行数字转换后将得到的数字加一
    System.out.println(stringToInt.compose(s -> s.toString().trim()).andThen(integer -> integer + 1).apply(" 123 "));
    ```  

## Optional 接口（防止 NullPointerException）  

NullPointerException 是 Java 中臭名昭著的一个异常了。因为程序的变化性，发生这个异常很难去跟踪到它进而调试程序。  

为了解决这个问题，我们经常性的做法是为我们的对象增加一个空对象，利用空对象模式来避免发生这个异常。但是这样的做法会导致程序的臃肿，我们不可能为每一个对象都去定义一个空对象。  

Java 8 中，增加了 Optional 接口来解决这个问题。Optional 是一个容器，其值可能是 null 也可能不是 null。它有一系列方法帮助我们更好的对值进行返回操作。  

下面是 Optional 的基本使用：  

```java
// of() : 为非 null 的值创建一个 Optional 对象，如果指定的值是 null，则抛出 NullPointerException
Optional<String> optional = Optional.of("hello");
// ofNullable：为一个指定的值创建一个 Optional 对象，如果指定的值为 null，则返回一个空的 Optional 对象
// isPresent(): 判断当前容器中的值是否为 null，不为空则返回 true，反之返回 false
optional.isPresent() // true
// get(): 如果 Optional 有值则将其返回，否则抛出 NoSuchElementException
optional.get() // "hello"
// orElse(some value): 如果 Optional 有值则将其返回，否则返回指定的值
optionl.orElse("world") // "hello"
// ifPresent(Consumer): 如果 Optional 有值则对这个值调用传入的 Consumer，否则不做处理
optional.ifPresent(s->System.out.println(s.trim())); // "hello"
```  
更多的 Optional 方法请参考 jdk 文档。  

下面是使用 Optional 来处理参数的 null 值判断处理的程序：  

```java
public String parseString(String target) {
    return Optional.ofNullable(target)
                   .map(target->new StringBuffer(target).reverse().toString())
                   .orElse("Unknow");
}
// 不适用 Optional 这段程序一般这样写
public String parseString(String target) {
    if (target == null) return "Unknow";
    return new StringBuffer(target).reverse().toString();
}
```  

设计 Optional 的目的在于让程序流自动处理未知传入值，它提供了把若干依赖前一步结果的处理结合起来的途径。  

下面是使用 Optional 场景的应用建议：  

1. 适用层级处理（依赖上一步操作）的场合。
2. 产生对象的方法若可能产生 null ，则可以用 Optional 包装。
3. 尽可能延后处理 null 的时机，在过程中使用 Optional 以保持不确定性。
4. 尽量避免使用 Optional 作为字段类型。  

## Stream（流操作）  

Java 8 中，为了增强对序列的操作，引入了流的概念。流相当于一个迭代器，但是它有更多的 API 可供操作，同时提供了并发流利用多线程加快序列的操作速度。  

流操作的基本用法如下：  

从数据源中获取流 -> 数据转换 -> 执行操作 -> 获取最终结果。流操作分为中间操作和最终操作，中间操作返回处理后的流对象，可以供下一个中间操作或者最终操作调用。也就意味着我们可以对流进行一系列的链式操作，但是这一系列的操作中只能有一个最终操作。  

可以作为数据源的对象主要有：  
- Collection 或 数组
    - Collection.stream() 
    - Collection.parallelStream()（并行流）
    - Arrays.stream(T array)
    - Stream.of(T... array)
- BufferedReader
    - BufferedReader.lines()
- 静态工厂
    - java.util.IntStream.range()
    - java.nio.file.Files.walk()

下面是流操作的用例：  

```java
// Stream 表示应用在一组元素上，一次性执行的操作序列
// 流上的操作分为：中间操作和最终操作，中间操作返回处理好的 Stream
// 我们可以将多个操作一次串起来来达到我们的目的
List<String> list = Arrays.asList("http://www.baidu.com", "https://www.baidu.com", "https://www.bilibili.com");
// 1. 利用流进行过滤操作，中间操作
// filter(Predicate) 提供一个 Predicate 接口可以对流进行过滤
list.stream().filter(s -> s.startsWith("https://"))
                .forEach(System.out::println);
// 2. 利用流进行排序操作，中间操作
// sorted() 和 sorted(Comparator)
// 该方法有两个重载函数，不提供参数则使用默认比较方法
// 提供一个比较器接口 Comparator 接口可以按照我们的要求进行排序
list.stream().sorted()
                .forEach(System.out::println);
// 3. 利用流进行映射操作，中间操作
// 通过 map(Function) 可以对流进行映射操作
list.stream().map(s -> s.replaceAll("http[s]?://", ""))
                .forEach(System.out::println);
// 4. 利用流进行匹配操作，最终操作
// anyMatch(Predicate) 只要流中有任意一个匹配，则返回 true，否则返回 false
boolean hasUrl = list.stream().anyMatch(s->s.startsWith("http://") || s.startsWith("https://"));
System.out.println(hasUrl);
// allMatch(Predicate) 流中所有的元素都需要匹配才返回 true，否则返回 false
boolean isAllUrl = list.stream().anyMatch(s->s.startsWith("http://") || s.startsWith("https://"));
System.out.println(isAllUrl);
// noneMatch(Predicate) 流中没有元素匹配返回ture，否则返回 false
boolean hasNoneUrl = list.stream().noneMatch(s -> s.startsWith("http://") || s.startsWith("https://"));
System.out.println(hasNoneUrl);
// 5. 利用流进行计数，最终操作
long urlCount = list.stream().filter(s -> s.startsWith("http://") || s.startsWith("https://"))
                            .count();
System.out.println(urlCount);
// 6. 利用流进行规约操作，最终操作
// 这个操作会将流中所有的元素按照一定的运算规则（二元运算）合并成一个元素
String totalUrl = list.stream().reduce((s1, s2)->s1+"#"+s2).orElse("Nothing");
System.out.println(totalUrl);

int[] ints = {'A', 'B', 'C'};
Arrays.stream(ints).forEach(System.out::println);
String[] strings = (String[]) list.toArray();
Stream.of(strings).forEach(System.out::println);
```  

## Date API（日期相关的 API）  

Java 8 在 `java.time` 包下新增了一个全新的日期和时间 API。