---
title: Java基础2
date: 2022-04-01  12:16:00 +0800
categories: [Java]
tags: [技术]
pin: true
author: 慕清辞

toc: true
comments: true
typora-root-url: ../../QingciMu.github.io
math: false
mermaid: true
---



# Java基础2

## 数据类型相关

### 包装类型

​	基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成

```java
Integer x = 2;     // 装箱 调用了 Integer.valueOf(2)
int y = x;         // 拆箱 调用了 X.intValue()
```



### 缓存池

new Integer(123) 与 Integer.valueOf(123) 的区别在于：

- new Integer(123) 每次都会新建一个对象；
- Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。

```java
Integer x = new Integer(123);
Integer y = new Integer(123);
System.out.println(x == y);    // false
Integer z = Integer.valueOf(123);
Integer k = Integer.valueOf(123);
System.out.println(z == k);   // true
```

valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容。

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

在 Java 8 中，Integer 缓存池的大小默认为 -128~127。

```java
static final int low = -128;
static final int high;
static final Integer cache[];

static {
    // high value may be configured by property
    int h = 127;
    String integerCacheHighPropValue =
        sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
    if (integerCacheHighPropValue != null) {
        try {
            int i = parseInt(integerCacheHighPropValue);
            i = Math.max(i, 127);
            // Maximum array size is Integer.MAX_VALUE
            h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
        } catch( NumberFormatException nfe) {
            // If the property cannot be parsed into an int, ignore it.
        }
    }
    high = h;

    cache = new Integer[(high - low) + 1];
    int j = low;
    for(int k = 0; k < cache.length; k++)
        cache[k] = new Integer(j++);

    // range [-128, 127] must be interned (JLS7 5.1.7)
    assert IntegerCache.high >= 127;
}
```

编译器会在自动装箱过程调用 valueOf() 方法，因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建，那么就会引用相同的对象。

```java
Integer m = 123;
Integer n = 123;
System.out.println(m == n); // true
```

基本类型对应的缓冲池如下：

- boolean values true and false
- all byte values
- short values between -128 and 127
- int values between -128 and 127
- char in the range \u0000 to \u007F

在使用这些基本类型对应的包装类型时，如果该数值范围在缓冲池范围内，就可以直接使用缓冲池中的对象。

在 jdk 1.8 所有的数值类缓冲池中，Integer 的缓冲池 IntegerCache 很特殊，这个缓冲池的下界是 - 128，上界默认是 127，但是这个上界是可调的，在启动 jvm 的时候，通过 -XX:AutoBoxCacheMax=<size> 来指定这个缓冲池的大小，该选项在 JVM 初始化的时候会设定一个名为 java.lang.IntegerCache.high 系统属性，然后 IntegerCache 初始化的时候就会读取该系统属性来决定上界。



## String

在 Java 8 中，String 内部使用 char 数组存储数据。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 `coder` 来标识使用了哪种编码。

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte[] value;

    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```

**String不可变的原因**

value 数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。



### 不可变的好处

**1.可以缓存hash值**

因为String的hash值经常被使用，力例如String用作HashMap的key。不可变的特性可以使得hash值也不可变，因此只需要进行一次计算。

**2.String Pool的需要**

如果一个String对象已经被创建了，那么就会从String Pool中取得引用。只有String是不可变的，才可能使用字符串常量池。

**3. 安全性**

String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 的那一方以为现在连接的是其它主机，而实际情况却不一定是。

**4. 线程安全**

String 不可变性天生具备线程安全，可以在多个线程中安全地使用。



### String、StringBuffer和StringBuilder

**1. 可变性**

- String 不可变
- StringBuffer 和 StringBuilder 可变

**2. 线程安全**

- String 不可变，因此是线程安全的
- StringBuilder 不是线程安全的
- StringBuffer 是线程安全的，内部使用 synchronized 进行同步



### new String 创建对象的问题

字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程将字符串添加到 String Pool 中。

当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

下面示例中，s1 和 s2 采用 new String() 的方式新建了两个不同字符串，而 s3 和 s4 是通过 s1.intern() 和 s2.intern() 方法取得同一个字符串引用。intern() 首先把 "aaa" 放到 String Pool 中，然后返回这个字符串引用，因此 s3 和 s4 引用的是同一个字符串。

```java
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
String s4 = s2.intern();
System.out.println(s3 == s4);           // true
```

如果是采用 "bbb" 这种字面量的形式创建字符串，会自动地将字符串放入 String Pool 中。

```java
String s5 = "bbb";
String s6 = "bbb";
System.out.println(s5 == s6);  // true
```

在 Java 7 之前，String Pool 被放在运行时常量池中，它属于永久代。而在 Java 7，String Pool 被移到堆中。这是因为永久代的空间有限，在大量使用字符串的场景下会导致 OutOfMemoryError 错误。

使用这种方式一共会创建两个字符串对象（前提是 String Pool 中还没有 "abc" 字符串对象）。

- "abc" 属于字符串字面量，因此编译时期会在 String Pool 中创建一个字符串对象，指向这个 "abc" 字符串字面量；
- 而使用 new 的方式会在堆中创建一个字符串对象。

创建一个测试类，其 main 方法中使用这种方式来创建字符串对象。

```java
public class NewStringTest {
    public static void main(String[] args) {
        String s = new String("abc");
    }
}
```

使用 javap -verbose 进行反编译，得到以下内容：

```java
// ...
Constant pool:
// ...
   #2 = Class              #18            // java/lang/String
   #3 = String             #19            // abc
// ...
  #18 = Utf8               java/lang/String
  #19 = Utf8               abc
// ...

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=1
         0: new           #2                  // class java/lang/String
         3: dup
         4: ldc           #3                  // String abc
         6: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
         9: astore_1
// ...
```

在 Constant Pool 中，#19 存储这字符串字面量 "abc"，#3 是 String Pool 的字符串对象，它指向 #19 这个字符串字面量。在 main 方法中，0: 行使用 new #2 在堆中创建一个字符串对象，并且使用 ldc #3 将 String Pool 中的字符串对象作为 String 构造函数的参数。

以下是 String 构造函数的源码，可以看到，在将一个字符串对象作为另一个字符串对象的构造函数参数时，并不会完全复制 value 数组内容，而是都会指向同一个 value 数组。

```java
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```




## 面向对象(OOP)

​	面向过程

- 步骤清晰简单，第一步、第二步...
- 面向过程适合处理一些简单问题



​	面向对象

- 物以类聚，分类的思维模式，先思考解决问题需要哪些分类，然后对分类进行单独思考，最后对某个分类的细节进行面向过程的思索。
- 适合处理复杂的问题、需要多人协作的问题。



**什么是面向对象**

本质就是：以类的方式组织代码，以对象的组织封装数据

抽象



三大特性：

- 封装

​	从形式上看，封装就是将数据和行为组合在一个包中，并对对象的使用者隐藏具体的实现方式。对外提供调用的接口。



- 继承

​	不同类型的对象，相互之间经常有一定数量的共同点。继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承，可以快速地创建新的类，可以提高代码的重用，程序的可维护性，节省大量创建新类的时间 ，提高我们的开发效率。

**关于继承如下 3 点请记住：**

1. 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问，**只是拥有**。
2. 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。
3. 子类可以用自己的方式实现父类的方法。



- 多态

​	多态，顾名思义，表示一个对象具有多种的状态，具体表现为父类的引用指向子类的实例。

**多态的特点:**

- 对象类型和引用类型之间具有继承（类）/实现（接口）的关系；
- 引用类型变量发出的方法调用的到底是哪个类中的方法，必须在程序运行期间才能确定；
- 多态不能调用“只在子类存在但在父类不存在”的方法；
- 如果子类重写了父类的方法，真正执行的是子类覆盖的方法，如果子类没有覆盖父类的方法，执行的是父类的方法。



## 用户自定义类

​	编写复杂的应用程序需要很多主力类，通常这些类都没有main方法，但是有自己的实例字段和实例方法。想要构建一个完整的程序，会结合使用多个类，其中只有一个类有main方法。

### 简单示例

下面是一个自定义的二叉树类：

```java
class TreeNode{
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(){}
    TreeNode(int val){
        this.val = val;
    }
    TreeNode(int val,TreeNode left,TreeNode right){
        this.val = val;
        this.left=left;
        this.right=right;
    }

    @Override
    public String toString() {
        return "TreeNode{" +
                "val=" + val +
                ", left=" + left +
                ", right=" + right +
                '}';
    }
}
```



### 访问权限修饰符

| 修饰符类型 | 当前类 | 子类 | 当前包 | 其他包 |
| ---------- | ------ | ---- | ------ | ------ |
| public     | ☑️      | ☑️    | ☑️      | ☑️      |
| protected  | ☑️      | ☑️    | ☑️      |        |
| private    | ☑️      |      |        |        |



### 使用null引用

​	如果对null值应用一个方法，会产生一个NullPointerException异常。

```java
LocalDtae birthday = null;
String s = birthday.toString();//NullPointerException
```



### final实例字段

​	可以将实例字段定义为final。这样的字段必须在构造对象时初始化。确保在每一个构造器执行之后，这个字段的值已经设置，并且以后不能再修改这个字段。

​	final修饰符对于类型为基本类型或者不可变类的字段有用。（如果类中的所有方法都不会改变其对象，这样的类就是不可变的类，eg：String类）。

​	对与可变的类，使用final修饰符可能会造成混乱。

```java
	private fianl StringBuffer evaluations;
	evaluations = new StringBuffer();
```



final关键字知识表示存储在evaluations变量中的对象引用不会再指向另一个不同的StringBuffer对象，不过这个对象可以更改。



## 静态字段与静态方法

### 静态字段

​	使用static关键字修饰的字段，每个类只有一个这样的字段，只属于类，而不属于任何单个对象。对于非静态的实例字段，每个对象都有自己的一个副本。



一般会使用静态常量。例如：static final double PI = 3.14



### 静态方法

​	静态方法是不在对象上执行的方法。例如：Math类的方法，都是静态方法。静态方法的调用使用（类名.方法名）。

静态方法的使用场景：

- 方法不需要访问对象的状态，因为它所需要的所有参数都通过显式参数提供。
- 方法只需要访问类的静态字段。



### 静态语句块

静态语句块在类初始化时运行一次

```java
public class A {
    static {
        System.out.println("123");
    }

    public static void main(String[] args) {
        A a1 = new A();
        A a2 = new A();
    }
}
```



### 静态内部类

非静态



### 初始化顺序

​	存在继承的情况下，初始化顺序为

- 父类（静态变量、静态语句块）
- 子类（静态变量、静态语句块）
- 父类（实例变量、普通语句块）
- 父类（构造函数）
- 子类（实例变量、普通语句块）
- 子类（构造函数）



### 工厂方法

​	静态方法还有另外一种常见的用途。类似LocalDate和NumberFormat的类使用静态工厂方法来构造对象。

NumberFormat类生成不同风格的格式化对象如下：

```java
NumberFormat currencyFormatter = NumberFormat.getCurrencyInstance();
NumberFormat perrcentFormatter = NumberFormat.getPercentInstance();
double x = 0.1;
System.out.println(currencyFormatter.format(x));
System.out.println(percentFormatter.format(x));
```



不利用NumberFormat类构造器完成这些操作的原因：

- 无法命名构造器。构造器的名字必须与类名相同。但是，这里希望有两个不同名字，分别得到货币实例和百分比实例。
- 使用构造器时，无法改变所构造对象的类型。而工厂方法实际上将返回DecimalFormat类的对象，这是NumberFormat 的一个子类。



## 继承

### 类、超类和子类

​	

###  重载

​	如果多个方法有相同的名字、不同的参数，便出现了重载。编译器必须挑选出具体调用哪个方法。它用各个方法首部中的参数类型与特定方法调用中所使用的值类型进行匹配，来选出正确的方法。匹配不到，会产生编译时错误。

​	<font color="#dd000">注：返回类型不是方法签名的一部分，不能有两个名字相同、参数类型也相同却有不同返回值类型的方法</font>



## 异常

​	<font color="#dd000">Java语言将派生于Error和RuntimException类的所有异常称为非检查型异常。</font>

### Unchecked Exception

- Error

​	描述了Java运行时系统的内部错误和资源耗尽错误。

- RuntimeException

​	一般规则是：由编程错误导致的异常属于RuntimeException

1. 错误的强制类型转换
2. 数组访问越界
3. 访问null指针



在Java代码编译过程中，即使不处理不受检查异常也可以正常通过编译。



### Checked Exception

- I/OException

1. 试图超越文件末尾继续读区数据
2. 试图打开一个不存在的文件
3. 试图根据给定的字符串查找class对象，而这个字符串表示的类并不存在



在Java代码的编译过程中，如股票受检查异常没有被catch/throws处理的话，就没办法通过编译。



