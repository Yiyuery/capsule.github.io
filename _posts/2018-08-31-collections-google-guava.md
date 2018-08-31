---
title: Google Guava 集合工具类
date: 2018-08-31 23:22:00
categories:
- Guava
tags:
- guava
---

任何对JDK集合框架有经验的程序员,都熟悉和喜欢java.util.Collections包含的工具方法。Guava沿着这些路线提供了更多的工具方法：适用于所有集合的静态方法。这是Guava最流行和成熟的部分原因之一。

---

# Google Guava 集合工具类

## Guava中的集合方法扩展

> 任何对JDK集合框架有经验的程序员都熟悉和喜欢java.util.Collections包含的工具方法。Guava沿着这些路线提供了更多的工具方法：适用于所有集合的静态方法。这是Guava最流行和成熟的部分原因之一。

| 集合接口       | JDK/Guava | Guava工具类                                  |
| :--------- | :-------- | :---------------------------------------- |
| Collection | JDK       | `Collections2`：不要和java.util.Collections混淆 |
| List       | JDK       | Lists                                     |
| Set        | JDK       | Sets                                      |
| SortedSet  | JDK       | Sets                                      |
| Map        | JDK       | Maps                                      |
| SortedMap  | JDK       | Maps                                      |
| Queue      | JDK       | Queues                                    |
| Multiset   | Guava     | Multisets                                 |
| Multimap   | Guava     | Multimaps                                 |
| BiMap      | Guava     | Maps                                      |
| Table      | Guava     | Tables                                    |

## 静态工方法

`Person`:

```java
/*
 * @ProjectName: 编程学习
 * @Copyright:   2018 HangZhou Yiyuery Dev., Ltd. All Right Reserved.
 * @address:     http://xiazhaoyang.tech
 * @date:        2018/8/31 18:15
 * @email:       xiazhaoyang@live.com
 * @description: 本内容仅限于编程技术学习使用，转发请注明出处.
 */
package com.example.chapter1.guava;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.ToString;

/**
 * <p>
 *
 * </p>
 *
 * @author xiachaoyang
 * @version V1.0
 * @date 2018年08月30日 20:37
 * @modificationHistory=========================逻辑或功能性重大变更记录
 * @modify By: {修改人} 2018年08月30日
 * @modify reason: {方法名}:{原因}
 * ...
 */
@Data
@ToString
@AllArgsConstructor
public class Person {
    private String name;
}
```

`Ex.`:

```java
/**
 * 推断范型的静态工厂方法
 * - 构造新的范型集合
 * - 初始化起始元素
 * - 工厂法声明集合变量
 * - 工厂法初始化集合大小
 */
@Test
public void declareStaticFactory(){
    //构造新的范型集合
    List<Person> list = Lists.newArrayList();
    Map<String, Person> map = Maps.newLinkedHashMap();
    Map<String, Person> hsahMap = Maps.newHashMap();
    //...
    list.add(new Person("p1"));
    //初始化起始元素
    Set<Person> copySet = Sets.newHashSet(list);
    System.out.println(copySet);//[Person(name=p1)]
    List<String> theseElements = Lists.newArrayList("alpha", "beta", "gamma");
    System.out.println(theseElements);//[alpha, beta, gamma]
    //工厂方法命名（Effective Java第一条），我们可以提高集合初始化大小的可读性
    List<Person> exactly100 = Lists.newArrayListWithCapacity(100);
    List<Person> approx100 = Lists.newArrayListWithExpectedSize(100);
    Set<Person> approx100Set = Sets.newHashSetWithExpectedSize(100);
    //Guava引入的新集合类型没有暴露原始构造器，也没有在工具类中提供初始化方法。而是直接在集合类中提供了静态工厂方法
    Multiset<String> multiset = HashMultiset.create();
}
```

## Iterables

> 常规方法

| 集合接口                              | 描述                                                     | 示例                                                        |
| :-------------------------------- | :----------------------------------------------------- | :-------------------------------------------------------- |
| concat(Iterable<Iterable>)        | 串联多个iterables的懒视图                                      | concat(Iterable...)                                       |
| frequency(Iterable, Object)       | 返回对象在iterable中出现的次数                                    | 与Collections.frequency (Collection,Object)比较;Multiset     |
| partition(Iterable, int)          | 把iterable按指定大小分割，得到的子集都不能进行修改操作                        | Lists.partition(List, int);paddedPartition(Iterable, int) |
| getFirst(Iterable, T default)     | 返回iterable的第一个元素，若iterable为空则返回默认值                     | 与Iterable.iterator().next()比较;FluentIterable.first()      |
| getLast(Iterable)                 | 返回iterable的最后一个元素，若iterable为空则抛出NoSuchElementException | getLast(Iterable, T default);FluentIterable.last()        |
| elementsEqual(Iterable, Iterable) | 如果两个iterable中的所有元素相等且顺序一致，返回true                       | 与List.equals(Object)比较                                    |
| unmodifiableIterable(Iterable)    | 返回iterable的不可变视图                                       | 与Collections.unmodifiableCollection(Collection)比较         |
| limit(Iterable, int)              | 限制iterable的元素个数限制给定值                                   | FluentIterable.limit(int)                                 |
| getOnlyElement(Iterable)          | 获取iterable中唯一的元素，如果iterable为空或有多个元素，则快速失败              | getOnlyElement(Iterable, T default)                       |

`>注:`懒视图意味着如果还没访问到某个iterable中的元素，则不会对它进行串联操作

```java
/**
 * 在可能的情况下，Guava提供的工具方法更偏向于接受Iterable而不是Collection类型。
 * 在Google，对于不存放在主存的集合（比如从数据库或其他数据中心收集的结果集）
 * 因为实际上还没有获取全部数据，这类结果集都不能支持类似size()的操作，通常都不会用Collection类型来表示。
 */
@Test
public void  testGuavaIterables(){
    Set<Integer> linkedHashSet = Sets.newLinkedHashSet();
    linkedHashSet.add(7);
    Iterable<Integer> iterable = Iterables.concat(Ints.asList(1,2,3),Ints.asList(4,5,6),linkedHashSet);
    Integer last = Iterables.getLast(linkedHashSet);
    System.out.println(iterable);
    System.out.println(last);
    //[1, 2, 3, 4, 5, 6, 7]
    //7
    Integer element = Iterables.getOnlyElement(linkedHashSet);
    System.out.println(element);
    //7
    linkedHashSet.add(8);
    element = Iterables.getOnlyElement(linkedHashSet);
    System.out.println(element);
    //java.lang.IllegalArgumentException: expected one element but was: <7, 8> linkedHashSet 如果不是单元素就会报错！
}
```

> 与Collection方法相似的工具方法

`Iterables`

| 方法                                                    | 类似的Collection方法                  | 等价的FluentIterable方法             |
| :---------------------------------------------------- | :------------------------------- | :------------------------------ |
| addAll(Collection addTo,   Iterable toAdd)            | Collection.addAll(Collection)    |                                 |
| contains(Iterable, Object)                            | Collection.contains(Object)      | FluentIterable.contains(Object) |
| removeAll(Iterable   removeFrom, Collection toRemove) | Collection.removeAll(Collection) |                                 |
| retainAll(Iterable   removeFrom, Collection toRetain) | Collection.retainAll(Collection) |                                 |
| size(Iterable)                                        | Collection.size()                | FluentIterable.size()           |
| toArray(Iterable, Class)                              | Collection.toArray(T\[])         | FluentIterable.toArray(Class)   |
| isEmpty(Iterable)                                     | Collection.isEmpty()             | FluentIterable.isEmpty()        |
| get(Iterable, int)                                    | List.get(int)                    | FluentIterable.get(int)         |
| toString(Iterable)                                    | Collection.toString()            | FluentIterable.toString()       |

`>注:`上面的方法中，如果传入的Iterable是一个Collection实例，则实际操作将会委托给相应的Collection接口方法。例如，往Iterables.size方法传入是一个Collection实例，它不会真的遍历iterator获取大小，而是直接调用Collection.size。

```java
    //通常来说 Collection的实现天然支持操作其他Collection，但却不能操作Iterable。
    List<Person> list = Lists.newArrayList();
    list.add(new Person("p1"));
    System.out.println(Iterables.size(list));//1
```

`源码:`

```java
/** Returns the number of elements in {@code iterable}. */
public static int size(Iterable<?> iterable) {
  return (iterable instanceof Collection)
      ? ((Collection<?>) iterable).size()
      : Iterators.size(iterable.iterator());
}
```

> FluentIterable 还有一些便利方法用来把自己拷贝到 不可变集合

| 名称                 | 方法                               |
| :----------------- | :------------------------------- |
| ImmutableSet       | toSet()                          |
| ImmutableSortedSet | toImmutableSortedSet(Comparator) |

```java
/**
 * FluentIterable还有一些便利方法用来把自己拷贝到不可变集合
 * 为什么要用immutable对象？immutable对象有以下的优点：
 * 　1.对不可靠的客户代码库来说，它使用安全，可以在未受信任的类库中安全的使用这些对象
 * 　2.线程安全的：immutable对象在多线程下安全，没有竞态条件
 * 　3.不需要支持可变性, 可以尽量节省空间和时间的开销. 所有的不可变集合实现都比可变集合更加有效的利用内存 (analysis)
 * 　4.可以被使用为一个常量，并且期望在未来也是保持不变的
 * Guava提供了对JDK里标准集合类里的immutable版本的简单方便的实现，以及Guava自己的一些专门集合类的immutable实现。当你不希望修改一个集合类，或者想做一个常量集合类的时候，使用immutable集合类就是一个最佳的编程实践
 * 注意：每个Guava immutable集合类的实现都拒绝null值。我们做过对Google内部代码的全面的调查，并且发现只有5%的情况下集合类允许null值，而95%的情况下都拒绝null值。万一你真的需要能接受null值的集合类，你可以考虑用Collections.unmodifiableXXX。
 * Immutable集合使用方法,一个immutable集合可以有以下几种方式来创建：
 * 　　1.用copyOf方法, 譬如, ImmutableSet.copyOf(set)
 * 　　2.使用of方法，譬如，ImmutableSet.of("a", "b", "c")或者ImmutableMap.of("a", 1, "b", 2)
 * 　　3.使用Builder类
 *
 */
@Test
public void testGuavaFluentIterable(){
   Set<Integer> linkedHashSet = Sets.newLinkedHashSet();
   linkedHashSet.add(7);
   ImmutableSet<Integer> immutableSet = ImmutableSet.copyOf(linkedHashSet);
   System.out.println(immutableSet);//[7]
   immutableSet = ImmutableSet.of(4,5,6);
   System.out.println(immutableSet);//[4, 5, 6]
   ImmutableMap<String, Integer> immutableMap = ImmutableMap.of("a", 1, "b", 2);
   System.out.println(immutableMap);//{a=1, b=2}
   ImmutableSet<Person> personImmutableSet =
           ImmutableSet.<Person>builder()
                   .add(new Person("p1"))
                   .add(new Person("p2"))
                   .build();
   System.out.println(personImmutableSet);//[Person(name=p1), Person(name=p2)]
   //拷贝到不可变集合
   immutableSet = FluentIterable.of(7,8,9).toSet();
   System.out.println(immutableSet);//[7, 8, 9]
   //拷贝到不可变集合(排序)
   Comparator<Integer> comparator = (h1, h2) -> h1.compareTo(h2);
   immutableSet = FluentIterable.of(7, 8, 9, 5, 4, 7, 9).toSortedSet(comparator);
   System.out.println(immutableSet);//[4, 5, 7, 8, 9]
   //反转排序
   immutableSet = FluentIterable.of(7, 8, 9, 5, 4, 7, 9).toSortedSet(comparator.reversed());
   System.out.println(immutableSet);//[9, 8, 7, 5, 4]
}
```

## Lists

> 静态工厂方法

| 名称         | 方法(根据入参类型不同区分)                                                                                          |
| :--------- | :------------------------------------------------------------------------------------------ |
| ArrayList  | basic, with elements, from Iterable, from Iterator, with exact capacity, with expected size |
| LinkedList | basic, from Iterable                                                                        |

`>注:`
  - `basic` : 无参构造器
  - `with elements` : E... elements
  - `from Iterable` : Iterable<? extends E> elements
  - `from Iterator` : Iterator<? extends E> elements
  - `with exact capacity` : int initialArraySize
  - `with expected size` : int estimatedSize

> 除了静态工厂方法和函数式编程方法，Lists为List类型的对象提供了若干工具方法。

| 方法                   | 描述                                                        |
| :------------------- | :-------------------------------------------------------- |
| partition(List, int) | 把List按指定大小分割                                              |
| reverse(List)        | 返回给定List的反转视图。注: 如果List是不可变的，考虑改用ImmutableList.reverse() |

```java
/**
  * Lists 方法测试
  */
  @Test
  public void testLists(){
     //反转排序
     List list = Ints.asList(1, 2, 3, 4, 5);
     System.out.println(Lists.reverse(list));//[5, 4, 3, 2, 1]
     //指定大小分割
     List<List> parts = Lists.partition(list, 2);
     System.out.println(parts);//[[1, 2], [3, 4], [5]]
     List<Integer> iList = Lists.newArrayList();
     List<Integer> iList2 = Lists.newArrayList(1, 2, 3);
     List<Integer> iList3 = Lists.newArrayList(iList2.iterator());
     List<Integer> iList4 = Lists.newArrayList(Iterables.concat(iList2));
     List<Integer> iList5 = Lists.newArrayListWithCapacity(1);
     List<Integer> iList6 = Lists.newArrayListWithExpectedSize(1);
     System.out.println(iList);//[]
     System.out.println(iList2);//[1, 2, 3]
     System.out.println(iList3);//[1, 2, 3]
     System.out.println(iList4);//[1, 2, 3]
     System.out.println(iList5);//[]
     System.out.println(iList6);//[]
     iList5.addAll(iList2);
     System.out.println(iList5);//[1, 2, 3]
     iList6.addAll(iList2);
     System.out.println(iList6);//[1, 2, 3]
}
```

## REFRENCES

1.  [\[Google Guava\] 2.3-强大的集合工具类：java.util.Collections中未包含的集合工具](http://ifeve.com/google-guava-collectionutilities/)
2.  [Guava学习笔记：Immutable(不可变)集合](http://www.cnblogs.com/peida/p/Guava_ImmutableCollections.html)
3.  [guava翻译系列之Collections](https://yq.aliyun.com/articles/71082)
4.  [Java8：Lambda表达式增强版Comparator和排序](http://www.importnew.com/15259.html)
5. [guava之Lists常用示例及newArrayListWithExpectedSize()和newArrayListWithCapacity()详细对比](https://blog.csdn.net/qq_27093465/article/details/53116390)

## 微信公众号

<center>
<img src="https://images.gitee.com/uploads/images/2018/0717/215030_8e782063_912956.png" width="50%" height="50%"/>
</center>
