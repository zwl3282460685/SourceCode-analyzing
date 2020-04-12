# 一、String 源码解析 #

## 1. 数据结构 ##

以主流的 JDK 版本 1.8 来说，String 内部实际存储结构为 char 数组，源码如下：

```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
	private final char value[]; //用于存储字符串的值
	private int hash;           //缓存字符串的hash code
	...
}
```

## 2. 构造方法

String 字符串有以下 4 个重要的构造方法：

```java
//String为参数的构造方法
public String(String original){
	this.value = original.value;
	this.hash= original.hash;
}

//char[] 为参数的构造方法
public String(char value[]){
    this.value = Arrays.copyOf(value, value.length);
}

//StringBuffer 为参数的构造方法
public String(StringBuffer buffer){
    synchronized(buffer){
        this.value = Arrays.copyOf(buffer,getValue(), buffer.length());
    }
}

//StringBuilder 为参数的构造方法
public String(StringBuilder builder){
    synchronized(builder){
        this.value = Arrays.copyOf(builder.getValue(), builder.length());
    }
}
```

## 3. equals() 比较两个字符串是否相等

```Java
public boolean equals(Object anObject) {
        //两个对象引用相同直接返回true
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {//判断需要对比的值是否为 String 类型，如果不是则直接返回 false
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                // 把两个字符串都转换为 char 数组对比
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) { //循环对比两个字符串的每一个字符
                    // 如果其中有一个字符不相等就 true false，否则继续对比
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

​       String类型重写了Object中的equals()方法，equals()方法需要传递一个Object类型的参数值，在比较时会先通过instanceof 判断是否为String类型，如果不是则直接返回false。当判断参数为 String 类型之后，会循环对比两个字符串中的每一个字符，当所有字符都相等时返回 true，否则则返回 false。还有一个和 equals() 比较类似的方法 equalsIgnoreCase()，它是用于忽略字符串的大小写之后进行字符串对比。

## 4. CompareTo() 比较两个字符串

compareTo() 方法用于比较两个字符串，返回的结果为 int 类型的值，源码如下：

```java
 public int compareTo(String anotherString) {
        int len1 = value.length;
        int len2 = anotherString.value.length;
        int lim = Math.min(len1, len2);//获取两个字符串长度最短的那个int值
        char v1[] = value;
        char v2[] = anotherString.value;
        int k = 0;
        while (k < lim) { //循环对比每一个元素
            char c1 = v1[k];
            char c2 = v2[k];
            if (c1 != c2) { //有字符不相等就返回差值
                return c1 - c2;
            }
            k++;
        }
        return len1 - len2;
    }

```

​       从源码中可以看出，compareTo()方法会循环对比所有的字符，当两个字符串中有任意一个字符不相同时，则returnchar1-char2。比如，两个字符串分别存储的是1和2，则返回的值是 1。还有一个和compareTo()比较类似的方法compareToIgnoreCase()，用于忽略大小写后比较两个字符串。可以看出compareTo()方法和equals()方法都是用于比较两个字符串的，但它们有两点不同：

- equals() 可以接收一个 Object 类型的参数，而 compareTo() 只能接收一个 String 类型的参数；

- equals() 返回值为 Boolean，而 compareTo() 的返回值则为 int。


它们都可以用于两个字符串的比较，当 equals() 方法返回 true 时，或者是 compareTo() 方法返回 0 时，则表示两个字符串完全相同。

## 5. 其他重要的方法

```json
indexOf()：查询字符串首次出现的下标位置
lastIndexOf()：查询字符串最后出现的下标位置
contains()：查询字符串中是否包含另一个字符串
toLowerCase()：把字符串全部转换成小写
toUpperCase()：把字符串全部转换成大写
length()：查询字符串的长度
trim()：去掉字符串首尾空格
replace()：替换字符串中的某些字符
split()：把字符串分割并返回字符串数组
join()：把字符串数组转为字符串
```

# 二 、常见的面试题 #

## 1. == 和 equals 的区别

== 对于基本数据类型来说，是用于比较 “值”是否相等的；而对于引用类型来说，是用于比较引用地址是否相同的。查看源码我们可以知道 Object 中也有 equals()  方法，源码如下：

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

可以看出，Object 中的 equals() 方法其实就是 ==，而 String 重写了 equals() 方法把它修改成比较两个字符串的值是否相等。源码见上面的分析。

## 2. String 类型用 final 修饰的好处

​       从 String 类的源码我们可以看出 String 是被 final 修饰的不可继承类。好处：一是为它能够缓存结果，当你在传参时不需要考虑谁会修改它的值；如果是可变类的话，则有可能需要重新拷贝出来一个新值进行传参，这样在性能上就会有一定的损失。二是安全，当你在调用其他方法时，比如调用一些系统级操作指令之前，可能会有一系列校验，如果是可变类的话，可能在你校验过后，它的内部的值又被改变了，这样有可能会引起严重的系统崩溃问题，这是迫使String类设计成不可变类的一个重要原因。总结来说，使用 final 修饰的第一个好处是安全；第二个好处是高效。

## 3. String 和 StringBuilder、StringBuffer 的区别

因为String类型是不可变的，所以在字符串拼接的时候如果使用String的话性能会很低，因此我们就需要使用另一个数据类型StringBuffer，它提供了append和insert方法可用于字符串的拼接，它使用 synchronized 来保证线程安全，如下源码所示：

```java
@Override
public synchronized StringBuffer append(Objectobj){
	toStringCache=null;
	super append(String.valueOf(obj));
     return this;
}
@Override
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}
```

因为它使用了synchronized来保证线程安全，所以性能不是很高，于是在JDK1.5就有了StringBuilder，它同样提供了append和insert的拼接方法，但它没有使用 synchronized 来修饰，因此在性能上要优于 StringBuffer，所以在非并发操作的环境下可使用 StringBuilder 来进行字符串拼接。

## 4. String 类型在 JVM中是如何存储的？编译器对 String 做了哪些优化？

String常见的创建方式有两种，new String()的方式和直接赋值的方式，直接赋值的方式会先去字符串常量池中查找是否已经有此值，如果有则把引用地址直接指向此值，否则会先在常量池中创建，然后再把引用指向此值；而new String()的方式一定会先在堆上创建一个字符串对象，然后再去常量池中查询此字符串的值是否已经存在，如果不存在会先在常量池中创建此字符串，然后把引用的值指向此字符串，代码如下：

```java
String s1 = new String("Java");
String s2 = s1.intern();
String s3 = "Java";
System.out.println(s1 == s2); // false
System.out.println(s2 == s3); // true
```

它们在JVM 存储的位置，如下图所示：

![img](https://s0.lgstatic.com/i/image3/M01/0D/BE/Ciqah16RQbaAZ3QkAACUHPvF6fE928.png)

注：JDK 1.7 之后把永生代换成的元空间，把字符串常量池从方法区移到了 Java 堆上。除此之外编译器还会对 String 字符串做一些优化，例如以下代码：返回结果为true。代码"Ja"+"va" 会被直接编译成成"Java"。

```java
String s1 = "Ja" + "va";
String s2 = "Java";
System.out.println(s1 == s2);
```