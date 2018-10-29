---
title: Java中的继承实现方式与执行顺序
date: 2018-09-16 16:22:01
categories:
- jdk
tags:
- extends
---

  Java中的继承实现方式与执行顺序

---

# Java中的继承实现方式与执行顺序

## 概要

> 本文主要探究如何使用Java中的继承（`extends`）？以及子父类中，`static{}`、`{}`和构造器执行顺序。

## 执行顺序

`A:`

```Java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou xiazhaoyang Dev, Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2018/9/17 22:28
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.capsule.chapter.extend;

import java.util.Date;

/**
 * <p>
 *
 * </p>
 *
 * @author xiazhaoyang
 * @version V1.0
 * @date 2018/9/17 22:28
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018/9/17
 * @modify reason: {方法名}:{原因}
 * ...
 */
public class A {
    public A() {
        System.out.println("Class A");
    }
    {
        System.out.println("{} A");
        overrideMe("{}");
    }
    static{
        System.out.println("static A");
        overrideMeStatic("static");
    }

    public void overrideMe(String area){
        System.out.println("overrideMe A - " + area);
    }

    public static void overrideMeStatic(String area){
        System.out.println("overrideMeStatic A - " + area);
    }

    public static void main(String[] args) {
        new A();
        //static A
        //overrideMeStatic A - static
        //{} A
        //overrideMe A - {}
        //Class A
    }
}
```
> 分析

- 类在初始化时，方法块`{}`、静态方法块`static{}`、构造器`className()`的执行顺序依次是：static{} > {} > className()
- 静态方法只能在静态方法块中执行，所以静态方法的执行顺序和静态方法块一样，是最高的（除非静态方法快中并没有使用该静态方法）。

`B:`

```Java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou xiazhaoyang Dev, Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2018/9/17 22:29
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.capsule.chapter.extend;

/**
 * <p>
 *
 * </p>
 *
 * @author xiazhaoyang
 * @version V1.0
 * @date 2018/9/17 22:29
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018/9/17
 * @modify reason: {方法名}:{原因}
 * ...
 */
public class B extends A{

    public B() {
        System.out.println("Class B");
    }
    {
        //方法快中无论什么方法都可以执行
        System.out.println("{} B");
        isStatic();
        notStatic();
    }
    static{
        //静态方法块中只能执行静态方法
        System.out.println("static B");
        //notStatic(); 报错
        isStatic();//不报错，使用的是静态方法
    }

    public static void isStatic(){
        System.out.println("is static method B");
    }

    public void notStatic(){
        System.out.println("not static method B");
    }

    public static void overrideMe(){
        overrideMeStatic("static method B");//父类静态方法
        System.out.println("overrideMe B");
    }

    public static void main(String[] args) {
        B b = new B();
        b.overrideMe();
        B.overrideMe();
        //---父类static{}
        //static A
        //overrideMeStatic A - static
        //---子类static{}
        //static B
        //is static method B
        //---父类方法块
        //{} A
        //overrideMe A - {}
        //---父类构造器
        //Class A
        //---子类方法块
        //{} B
        //is static method B
        //not static method B
        //---子类构造器
        //Class B
        //--- 实例方法调用
        //overrideMeStatic A - static method B
        //overrideMe B
        //--- 子类静态方法调用
        //overrideMeStatic A - static method B
        //overrideMe B
    }

}

```

> 分析

- 子类初始化时，`父类static{}和其中的静态方法` > `子类static{}和其中的静态方法` > `父类方法块{}` > `父类构造方法` > `子类方法块` > `子类构造器`
- `static{}`方法块执行优先级最高(父>子)
- 单个类中，方法块调用在类初始化构造器之前，子父类中，子类方法块的执行在父类构造器方法之后执行   

## 注意事项

> 要么为继承而设计，并提供文档说明，要么就禁止继承

- 继承对于final变量域的修改

`Super:`
```Java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2018/9/12 08:50
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.capsule.chapter.extend;

/**
 * <p>
 *
 * </p>
 *
 * @author xiazhaoyang
 * @version V1.0
 * @date 2018/9/12 08:50
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018/9/12
 * @modify reason: {方法名}:{原因}
 * ...
 */
public class Super {

    public Super() {
        overrideMe();
    }

    public void overrideMe(){
        System.out.println("Super overrideMe!");
    }
}

```
`Sub:`
```Java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev, Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2018/9/12 08:52
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.capsule.chapter.extend;

import java.util.Date;

/**
 * <p>
 *
 * </p>
 *
 * @author xiazhaoyang
 * @version V1.0
 * @date 2018/9/12 08:52
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018/9/12
 * @modify reason: {方法名}:{原因}
 * ...
 */
public class Sub extends Super{

    private final Date date;

    Sub(){
        date = new Date();
    }
    @Override
    public void overrideMe(){
        System.out.println(date);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
        //null
        //Wed Sep 12 08:55:57 CST 2018
    }

}
```
继承之后，子类实例化之前会先执行被重载的父类方法overrideMe()，此时子类的实例化尚未完成，静态块也未执行，所以虽然是final修饰的字段，date变量仍然是null,实例化时打印null，实例化之后打印复制后的新时间。

---

## 微信公众号

<center>
<img src="https://images.gitee.com/uploads/images/2018/0717/215030_8e782063_912956.png" width="50%" height="50%"/>
</center>

扫码关注或搜索`架构探险之道`获取最新文章，不积跬步无以至千里，坚持每周一更，坚持技术分享。我和你们一起成长 ^_^ ！
