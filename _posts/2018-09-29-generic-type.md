---
title: Java 泛型
date: 2018-09-29 16:22:01
categories:
- jdk
tags:
- generic-type
---

  Java 泛型

---

# Java 泛型

## 泛型方法

> Java 泛型

如果我们只写一个排序方法，就能够对整型数组、字符串数组甚至支持排序的任何类型的数组进行排序，这该多好啊。
Java泛型方法和泛型类支持程序员使用一个方法指定一组相关方法，或者使用一个类指定一组相关的类型。
Java泛型（generics）是JDK 5中引入的一个新特性,泛型提供了编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型。
使用Java泛型的概念，我们可以写一个泛型方法来对一个对象数组排序。然后，调用该泛型方法来对整型数组、浮点数数组、字符串数组等进行排序。该方法在调用时可以接收不同类型的参数。根据传递给泛型方法的参数类型，编译器适当地处理每一个方法调用。

下面是定义泛型方法的规则：

- 所有泛型方法声明都有一个类型参数声明部分（由尖括号分隔），该类型参数声明部分在方法返回类型之前（在下面例子中的<E>）。
- 每一个类型参数声明部分包含一个或多个类型参数，参数间用逗号隔开。一个泛型参数，也被称为一个类型变量，是用于指定一个泛型类型名称的标识符。
- 类型参数能被用来声明返回值类型，并且能作为泛型方法得到的实际参数类型的占位符。
- 泛型方法方法体的声明和其他方法一样。注意类型参数只能代表引用型类型，不能是原始类型（像int,double,char的等）。

### 全局泛型

```java
/**
 * 数组打印
 *
 * @param inputArray
 * @param <E>
 */
public <E> void printArray(E[] inputArray) {
    // 输出数组元素
    Arrays.asList(inputArray).forEach(p -> System.out.printf("%s ", p));
}

/**
 * 核心测试方法
 */
@Test
public void testCommonTypeFunctionDefine() {
    printArray(new String[]{"a", "b", "c"});
    //a b c
    System.out.println();
    printArray(new Integer[]{1, 2, 3, 4});
    //1 2 3 4
}

```

### 有界的类型参数

> 可能有时候，你会想限制那些被允许传递到一个类型参数的类型种类范围。例如，一个操作数字的方法可能只希望接受Number或者Number子类的实例。这就是有界类型参数的目的。要声明一个有界的类型参数，首先列出类型参数的名称，后跟extends关键字，最后紧跟它的上界。

```java
/**
  * 有界泛型 比较三个值并返回最大值
  * @param x
  * @param y
  * @param z
  * @param <T>
  * @return
  */
 public <T extends Comparable<T>> T maximum(T x, T y, T z)
 {
     T max = x; // 假设x是初始最大值
     if ( y.compareTo( max ) > 0 ){
         max = y; //y 更大
     }
     if ( z.compareTo( max ) > 0 ){
         max = z; // 现在 z 更大
     }
     return max; // 返回最大对象
 }

```

### 泛型类

> 泛型类

```java
package com.example.chapter3.function;

import lombok.Data;

/**
 * <p>
 *  泛型类
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年09月30日 11:17
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年09月30日
 * @modify reason: {方法名}:{原因}
 * ...
 */
@Data
public class Box<T> {
    private T t;
}

```

> 泛型有界类

```java
package com.example.chapter3.function;

import lombok.Data;

import java.util.Collection;

/**
 * <p>
 *  泛型有界类
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年09月30日 11:19
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年09月30日
 * @modify reason: {方法名}:{原因}
 * ...
 */
@Data
public class BoxExtends<T extends Collection> {
    private T t;
}

```

> 测试demo

`Ex:`

```java
//-------------------------------------------------------------
System.out.printf( "Max of %d, %d, %d is %d\n",
       3, 4, 5, maximum( 3, 4, 5 ) );
System.out.printf( "Maxm of %.1f, %.1f, %.1f is %.1f\n",
       6.6, 8.8, 7.7, maximum( 6.6, 8.8, 7.7 ) );
System.out.printf( "Max of %s, %s, %s is %s\n","pear",
       "apple", "orange", maximum( "pear", "apple", "orange" ) );
//-------------------------------------------------------------
Box<Integer> integerBox = new Box<>();
Box<String> stringBox = new Box<>();
integerBox.setT(new Integer(10));
stringBox.setT(new String("Hello World"));
System.out.printf("Integer Value :%d\n", integerBox.getT());
System.out.printf("String Value :%s\n", stringBox.getT());
//Integer Value :10
//String Value :Hello World
//-------------------------------------------------------------
BoxExtends<List> boxList = new BoxExtends<>();
boxList.setT(Lists.newArrayList("1", "2"));
System.out.println(boxList);
//BoxExtends(t=[1, 2])

```

## 类型安全

```java
/**
  * 优先考虑类型安全的异构容器
  */
 @Test
 public void effective29Test(){
     Favorites map = new Favorites();
     map.putFavorite(Integer.class,13);
     map.putFavorite(String.class,"12");
     map.putFavorite(Class.class,Favorites.class);
 }

 class Favorites{

     private Map<Class<?>,Object> favorites = new HashMap<>();

     public <T> void putFavorite(Class<T> type, T instance){
         if(type == null){
             throw new NullPointerException("Type is null!");
         }
         favorites.put(type,instance);
     }

     public <T> T getFavorite(Class<T> type){
         return type.cast(favorites.get(type));
     }
 }

```
- `public <T> void putFavorite(Class<T> type, T instance)` 对放入集合中的对象进行类型检查，检查失败编译器会报错。
- Class内置方法cast会对对象引用进行动态转换

## REFERENCES
1. [W3cSchool java泛型](https://www.w3cschool.cn/java/java-generics.html)
2. [Reflection API](http://www.51gjie.com/java/792.html)


---

## 更多

> 扫码关注“架构探险之道”，回复`文章名称`获取更多源码和文章资源

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222309957.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)

> 知识星球(扫码加入获取源码和文章资源链接)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403222322267.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzI4NjkwNDE3,size_16,color_FFFFFF,t_70)
