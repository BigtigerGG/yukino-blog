---
title: Java 正则表达式
date: 2020-03-15 20:49:35
tags:
- Java
categories:
- Java
- Java SE
---

正则表达式是一个非常强大的工具，Java 对正则表达式的支持主要利用了 `java.util.regex` 中的一些类。

## 基础使用  

String 类的对象的成员方法利用了正则表达式来更灵活的处理字符串。  

1. `matches()` 

   这个方法用来进行字符串的模式匹配，例如邮箱的匹配:  
   
	```Java
	boolean isMatch = "yukino@stu.cdut.edu.cn".matches("\S+@[a-zA-Z]+(.[a-zA-Z]+)+");
	System.out.println(isMatch);
	// expected output : true
	```
	
2. `split()` 

   这个方法用来进行字符串的分割，按照正则表达式将一个字符串分割为一个数组。  

   例如：
   
   ```java
   for (String s : "Hello world".split(" ")) {
     System.out.println(s)
   }
   // expected output :
   // Hello 
   // world
   ```
   
3. `replaceFirst()`, `replaceAll()` 

   这个方法用来进行字符串的替换，前者只替换第一个匹配到的字符，后者则全部替换。

## 模式类，匹配类（Pattern，Matcher）

Java 底层使用这个两个类来进行字符串的匹配的，基础用法如下：

```java
Pattern p = Pattern.compile("[abc]+"); // 这里将一个正则表达式的字符串形式编译成一个 Pattern 对象
Matcher m = p.matcher("ababab"); // 这里用 p 返回一个指定要匹配的字符串的匹配器对象
boolean isMatch = m.matches(); // 最后判断是否匹配

isMatch = Pattern.matches("[abc]+", "ababab"); // 这是更简单的做法，但是失去了灵活性
```

## 量词

正则表达式中的量词描述了一个模式匹配文本的次数。

常用的几种量词：

量词|用例|用法
:-:|:-:|:-:
?|[abc]?|匹配0个或1个模式
+|[abc]+|匹配1个或多个模式
*|[abc]*|匹配0个或多个模式
{n}|[abc]{2}|恰好n个模式
{n,}|[abc]{2,}|至少n个模式
{n,m}|[abc]{2,3}|至少n个，但不超过m个


量词在 Java 中有三种类型：

- 贪婪型：量词总是贪婪的，除非有其他的选项被设置。贪婪表达式会为所有可能的模式发现尽可能多的匹配。也就是，这种匹配模式会导致匹配尽可能长的字符串。通常会导致一种问题：如果我们的模式仅能匹配第一个可能的字符组，如果它是贪婪的，那么他就会继续往下匹配。  
例如，可能会想用"<.+>"去匹配“a\<tr\>aaa\</tr\>"中的"\<tr\>", 贪婪模式会导致匹配到"\<tr\>aaa\</tr>"。
- 勉强型：用 ? 来指定，这种模式只会匹配模式所需要的最小字符数。
- 占有型：用 + 来指定，这种模式是 Java 特有的，它可以避免匹配过程重的回溯，当匹配失败的时候就不再匹配了。