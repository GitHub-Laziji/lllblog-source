---
title: Java中静态成员变量的加载
date: 2018-10-16 13:43:15
categories: 技术分享
tags: Java
---

# 前一个成员变量调用其后的成员变量
例如:
```Java
public static final Object A = B;
public static final Object B = "String B";
```
这种显然是不行的, 编译都通不过, 而下面一种就有点不一样了

# 前成员变量在静态方法中调用其后的成员变量
例如:
```Java
public class Test {

    public static final Object A = getB();
    public static final Object B = "String B";

    public static Object getB() {
        return B;
    }
}
```
此时编译正常但是, 打印一下A看看:
```Java
System.out.println(Test.A); // null
System.out.println(Test.B); // "String B"
System.out.println(Test.getB()); // "String B"
```
发现A是`null`, 因为此时B还未初始化, 但是当把B的类型改为`String`时
```Java
public class Test {

    public static final Object A = getB();
    public static final String B = "String B"; // B 的类型改为 String

    public static Object getB() {
        return B;
    }
}
```
此时A的值就不是`null`
```Java
System.out.println(Test.A); // "String B"
```
这是因为Java的静态成员变量加载时`类型为基本类型和String, 并且赋值为常量表达式的成员变量`的加载顺序优与其他
常量表达式指能立即得出的运算 包括`普通运算以及字符串运算`


当静态方法太复杂的时候就可能会碰到这种问题, 而且难找到原因所在

