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

| 名称         | 方法(根据入参类型不同区分)                                                                              |
| :--------- | :------------------------------------------------------------------------------------------ |
| ArrayList  | basic, with elements, from Iterable, from Iterator, with exact capacity, with expected size |
| LinkedList | basic, from Iterable                                                                        |

`>注:`

-   `basic` : 无参构造器
-   `with elements` : E... elements
-   `from Iterable` : Iterable&lt;? extends E> elements
-   `from Iterator` : Iterator&lt;? extends E> elements
-   `with exact capacity` : int initialArraySize
-   `with expected size` : int estimatedSize

> 除了静态工厂方法和函数式编程方法，Lists为List类型的对象提供了若干工具方法。

| 方法                   | 描述                                                       |
| :------------------- | :------------------------------------------------------- |
| partition(List, int) | 把List按指定大小分割                                             |
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

> `newArrayListWithCapacity` 和 `newArrayListWithExpectedSize` 对比

下面的这个写法呢是在初始化list的时候，说明容器的扩容界限值。使用条件：你确定你的容器会装多少个，不确定就用一般形式的。说明：这个容器超过`10`个还是会自动扩容的。不用担心容量不够用。默认是分配一个容量为`10`的数组，不够将扩容。执行数组数据迁移操作：新建新数组，复制就数组数据到新数组（包括开辟新空间，copy数据等等，耗时，耗性能）。对数字 `10` 的说明：若直接new一个list的话，默认大小是`10`的数组，下面的方式则是 `5L + x + x/10 = 16L(x = 10)`，在`17`的时候扩容。整个来说的优点有：节约内存，节约时间，节约性能。代码质量提高(10为下方例子中的数字，可设置为其他数字)。

```java
List<String> list = Lists.newArrayListWithExpectedSize(10);
```

`Lists.newArrayListWithExpectedSize(10)源码`

```java
//静态工厂方法
@GwtCompatible(serializable = true)
public static <E> ArrayList<E> newArrayListWithExpectedSize(int estimatedSize) {
  return new ArrayList<>(computeArrayListCapacity(estimatedSize));
}
//计算容量
@VisibleForTesting
static int computeArrayListCapacity(int arraySize) {
  checkNonnegative(arraySize, "arraySize");

  // TODO(kevinb): Figure out the right behavior, and document it
  return Ints.saturatedCast(5L + arraySize + (arraySize / 10));
}
//校验非负数
@CanIgnoreReturnValue
static int checkNonnegative(int value, String name) {
  if (value < 0) {
    throw new IllegalArgumentException(name + " cannot be negative but was: " + value);
  }
  return value;
}
//容量大小范围控制
public static int saturatedCast(long value) {
    if (value > Integer.MAX_VALUE) {
      return Integer.MAX_VALUE;
    }
    if (value < Integer.MIN_VALUE) {
      return Integer.MIN_VALUE;
    }
    return (int) value;
  }
```

这个方法就是直接返回一个10的数组。

```java
List<String> list_ = Lists.newArrayListWithCapacity(10);
```

`Lists.newArrayListWithCapacity(10)源码`

```java
//静态工厂方法
@GwtCompatible(serializable = true)
public static <E> ArrayList<E> newArrayListWithCapacity(int initialArraySize) {
  checkNonnegative(initialArraySize, "initialArraySize"); // for GWT.
  return new ArrayList<>(initialArraySize);
}
```

`ArrayList源码里面的add方法以及如何扩容`

```java
public boolean add(E e) {
     ensureCapacityInternal(size + 1);  // Increments modCount!! (Increments 翻译-- 增量; 增长( increment的名词复数 ); 增额; 定期的加薪;)
     elementData[size++] = e;
     return true;
}

private void ensureCapacityInternal(int minCapacity) {// Internal 翻译--  内部的; 国内的; 体内的; 内心的;   Capacity  翻译--  容量; 性能; 才能; 生产能力;
     ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private void ensureExplicitCapacity(int minCapacity) {//Explicit  翻译-- 明确的，清楚的; 直言的; 详述的; 不隐瞒的;
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)// 扩容条件，就是minCapacity = size + 1 大于当前数组的长度。就扩容.
        grow(minCapacity);
}

private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private void grow(int minCapacity) {
     // overflow-conscious code
     int oldCapacity = elementData.length;
     int newCapacity = oldCapacity + (oldCapacity >> 1);//新容量相当于是原来容量的1.5倍(old + old>>1右移一位即除以2)
     if (newCapacity - minCapacity < 0)
         newCapacity = minCapacity;
     if (newCapacity - MAX_ARRAY_SIZE > 0)
         newCapacity = hugeCapacity(minCapacity);
     // minCapacity is usually close to size, so this is a win:
     elementData = Arrays.copyOf(elementData, newCapacity);// 复制旧数组数据到新数组
 }
```

`ArrayList 的 modCount 属性`

```java
/**
  * The number of times（...的次数） this list has been <i>structurally modified（结构修改）</i>.
  * Structural modifications(结构修改) are those that change the size of the
  * list, or otherwise（其他） perturb（使混乱） it in such a fashion（以这样的方式） that iterations（迭代） in
  * progress（过程，流程） may yield（产生） incorrect results（不正确的结果）.
  *
  * <p>This field（字段，成员变量，域） is used by the iterator and list iterator implementation
  * returned by the {@code iterator} and {@code listIterator} methods.
  * If the value of this field changes unexpectedly（非预期的，意外的）, the iterator (or list
  * iterator) will throw a {@code ConcurrentModificationException} in
  * response to the {@code next}, {@code remove}, {@code previous},
  * {@code set} or {@code add} operations.  This provides
  * <i>fail-fast</i> behavior, rather than（而不是） non-deterministic behavior（非确定的行为） in
  * the face of（在面对...时）concurrent modification（并发性的修改） during iteration（在迭代过程中）.
  *
  * <p><b>Use of this field by subclasses（子类们） is optional（可选的）.</b> If a subclass
  * wishes to provide fail-fast iterators（希望提供快速失败的迭代器） (and list iterators), then it
  * merely（仅仅） has to increment this field in its {@code add(int, E)} and
  * {@code remove(int)} methods (and any other methods that it overrides
  * that result in structural modifications to the list).  A single call（一个单一的调用） to
  * {@code add(int, E)} or {@code remove(int)} must add no more than（不超过）
  * one to this field, or the iterators (and list iterators) will throw
  * bogus {@code ConcurrentModificationExceptions}.  If an implementation
  * does not wish to provide fail-fast iterators, this field may be
  * ignored（忽略）.
  */
 protected transient int modCount = 0;
```

## Sets

> Sets工具类包含了若干好用的方法,它提供了很多标准的集合运算（Set-Theoretic）方法，这些方法接受Set参数并返回SetView，可用于：直接当作Set使用，因为SetView也实现了Set接口；用`copyInto(Set)`拷贝进另一个可变集合；用`immutableCopy()`对自己做不可变拷贝。

| 方法                           | 描述              |
| :--------------------------- | :-------------- |
| union(Set, Set)              | ANB             |
| intersection(Set, Set)       | ANB             |
| difference(Set, Set)         | AN(!B)          |
| symmetricDifference(Set,Set) | (AUB)&(!(ANB))  |
| newCopyOnWriteArraySet(Set)  | 返回一个复制所有元素的集合对象 |
| cartesianProduct(List<Set>)  | 返回所有集合的笛卡儿积     |
| powerSet(Set)                | 返回给定集合的所有子集     |

`>注`:

-   `N`:数学集合操作中的"交集";
-   `U`:数学集合操作中的"并集";
-   `!`:非，表示不在此集合中

```java
/**
  *  Sets 方法测试
  */
 @Test
 public void testSets(){
     HashSet<Integer> iSets = Sets.newHashSet(1, 2, 3, 4);
     HashSet<Integer> iSets2 = Sets.newHashSet(4, 5, 6, 7);
     System.out.println(Sets.union(iSets,iSets2));//[1, 2, 3, 4, 5, 6, 7]
     System.out.println(Sets.intersection(iSets,iSets2));//[4]
     System.out.println(Sets.difference(iSets,iSets2));//[1, 2, 3]
     System.out.println(Sets.symmetricDifference(iSets,iSets2));//[1, 2, 3, 5, 6, 7]
     System.out.println(Sets.newCopyOnWriteArraySet(iSets));//[1, 2, 3, 4]
     System.out.println(iSets.equals(Sets.newCopyOnWriteArraySet(iSets)));//true
     System.out.println(Sets.cartesianProduct(iSets,iSets2));//[[1, 4], [1, 5], [1, 6], [1, 7], [2, 4], [2, 5], [2, 6], [2, 7], [3, 4], [3, 5], [3, 6], [3, 7], [4, 4], [4, 5], [4, 6], [4, 7]]
     System.out.println(Sets.cartesianProduct(iSets));//[[1], [2], [3], [4]]
     System.out.println(Sets.powerSet(iSets));//powerSet({1=0, 2=1, 3=2, 4=3})
     HashSet<Person> pSets = Sets.newHashSet(new Person("p1"),new Person("p2"),new Person("p3"));
     System.out.println(Sets.powerSet(pSets));//powerSet({Person(name=p1)=0, Person(name=p2)=1, Person(name=p3)=2})
     Set<String> sSets = ImmutableSet.of("one", "two", "three", "six", "seven", "eight");
     Set<String> sSets2 = ImmutableSet.of("two", "three", "five", "seven");
     Set<String> sSets3 = Sets.newCopyOnWriteArraySet(sSets);
     Sets.SetView<String> setView = Sets.intersection(sSets,sSets2);
     System.out.println(setView);//[two, three, seven]
     System.out.println(setView.immutableCopy());//[two, three, seven] 可以使用交集，但不可变拷贝的读取效率更高
     System.out.println(sSets3);//[one, two, three, six, seven, eight]
     System.out.println(setView.copyInto(sSets));//java.lang.UnsupportedOperationException
 }
```

> 静态工厂方法

| 类型            | 工厂方法                                                                   |
| :------------ | :--------------------------------------------------------------------- |
| HashSet       | basic, with elements, from Iterable, with expected size, from Iterator |
| LinkedHashSet | basic, from Iterable, with expected size                               |
| TreeSet       | basic, with Comparator, from Iterable                                  |

`>注:`

-   `basic` : 无参构造器
-   `with elements` : E... elements
-   `from Iterable` : Iterable&lt;? extends E> elements
-   `from Iterator` : Iterator&lt;? extends E> elements
-   `with expected size` : int expectedSize

## Maps

| 方法                                  | 描述                                                               |
| :---------------------------------- | :--------------------------------------------------------------- |
| Maps.uniqueIndex(Iterable,Function) | 方法返回一个Map，键为Function返回的属性值，值为Iterable中相应的元素，因此我们可以反复用这个Map进行查找操作 |
| Maps.difference(Map, Map)           | 用来比较两个Map以获取所有不同点。该方法返回MapDifference对象，把不同点的维恩图分解成键值对            |

`维恩图分解`

| 方法                   | 描述                                                                |
| :------------------- | :---------------------------------------------------------------- |
| entriesInCommon()    | 两个Map中都有的映射项，包括匹配的键与值                                             |
| entriesDiffering()   | 键相同但是值不同值映射项。返回的Map的值类型为MapDifference.ValueDifference，以表示左右两个不同的值 |
| entriesOnlyOnLeft()  | 键只存在于左边Map的映射项                                                    |
| entriesOnlyOnRight() | 键只存在于右边Map的映射项                                                    |

```java
/**
 * Maps 方法测试
 */
@Test
public void testMaps(){
    //我们有一堆字符串，这些字符串的长度都是独一无二的，而我们希望能够按照特定长度查找字符串
    List<String> strs = Lists.newArrayList("a","ab","abc");
    ImmutableMap<Integer, String> sMap = Maps.uniqueIndex(strs,(p)->p.length());
    System.out.println(sMap);//{1=a, 2=ab, 3=abc}
    strs.add("x");
    //sMap = Maps.uniqueIndex(strs,(p)->p.length());//java.lang.IllegalArgumentException: Multiple entries with same key: 1=x and 1=a. To index multiple values under a key, use Multimaps.index.
    System.out.println(sMap);
    //用来比较两个Map以获取所有不同点。该方法返回MapDifference对象，把不同点的维恩图分解不同键值对：
    Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
    Map<String, Integer> right = ImmutableMap.of("a", 1, "b", 2, "c", 3);
    MapDifference<String, Integer> diff = Maps.difference(left, right);
    System.out.println(diff);//equal
    Map<String, Integer> right2 = ImmutableMap.of("a", 1, "b", 3, "d", 4);
    MapDifference<String, Integer> diff2 = Maps.difference(left, right2);
    System.out.println(diff2);//not equal: only on left={c=3}: only on right={d=4}: value differences={b=(2, 3)}
    System.out.println(diff2.entriesInCommon());//{a=1, b=2}
    System.out.println(diff2.entriesDiffering());//{b=(2, 3)}
    System.out.println(diff2.entriesOnlyOnLeft());//{c=3}
    System.out.println(diff2.entriesOnlyOnRight());//{d=4}
}
```

> 处理 BiMap 的工具方法（BiMap也是一种Map实现）

| BiMap工具方法                | 相应的Map工具方法                       |
| :----------------------- | :------------------------------- |
| synchronizedBiMap(BiMap) | Collections.synchronizedMap(Map) |
| unmodifiableBiMap(BiMap) | Collections.unmodifiableMap(Map) |

```java
//和一个key可以用多个value一样，map还有一种方式可以使一个value对应一个key，这就是Bimap. BiMap中value的值是唯一的。
BiMap<String,Person> pMap = HashBiMap.create(10);
pMap.put("a",new Person("Pa"));
pMap.put("b",new Person("Pb"));
//pMap.put("c",new Person("Pa"));//java.lang.IllegalArgumentException: value already present: Person(name=Pa)
BiMap<String, Person> pMap2 = Maps.synchronizedBiMap(pMap);
System.out.println(pMap2);//{a=Person(name=Pa), b=Person(name=Pb)}
```

> 静态工厂方法

| 类型              | 工厂方法                                   |
| :-------------- | :------------------------------------- |
| HashMap         | basic, from Map, with expected size    |
| LinkedHashMap   | basic, from Map                        |
| TreeMap         | basic, from Comparator, from SortedMap |
| EnumMap         | from Class, from Map                   |
| ConcurrentMap   | basic                                  |
| IdentityHashMap | basic                                  |

## Multisets

> 标准的Collection操作会忽略Multiset重复元素的个数，而只关心元素是否存在于Multiset中，如containsAll方法。为此，Multisets提供了若干方法，以顾及Multiset元素的重复性：


`相似方法`

| 方法                                                          | 说明                                                              | 和Collection方法的区别                                |
| :---------------------------------------------------------- | :-------------------------------------------------------------- | :---------------------------------------------- |
| containsOccurrences(Multiset   sup, Multiset sub)           | 对任意o，如果sub.count(o)&lt;=super.count(o)，返回true                   | Collection.containsAll忽略个数，而只关心sub的元素是否都在super中 |
| removeOccurrences(Multiset   removeFrom, Multiset toRemove) | 对toRemove中的重复元素，仅在removeFrom中删除相同个数。                            | Collection.removeAll移除所有出现在toRemove的元素          |
| retainOccurrences(Multiset   removeFrom, Multiset toRetain) | 修改removeFrom，以保证任意o都符合removeFrom.count(o)&lt;=toRetain.count(o) | Collection.retainAll保留所有出现在toRetain的元素          |

`其他方法`

| 方法                                         | 说明                                 |
| :----------------------------------------- | :--------------------------------- |
| intersection(Multiset,   Multiset)         | 返回两个multiset的交集;                   |
| copyHighestCountFirst(Multiset)            | 返回Multiset的不可变拷贝，并将元素按重复出现的次数做降序排列 |
| unmodifiableMultiset(Multiset)             | 返回Multiset的只读视图                    |
| unmodifiableSortedMultiset(SortedMultiset) | 返回SortedMultiset的只视图               |

## Multimaps

## Tables

## REFERENCES

1.  [\[Google Guava\] 2.3-强大的集合工具类：java.util.Collections中未包含的集合工具](http://ifeve.com/google-guava-collectionutilities/)
2.  [Guava学习笔记：Immutable(不可变)集合](http://www.cnblogs.com/peida/p/Guava_ImmutableCollections.html)
3.  [guava翻译系列之Collections](https://yq.aliyun.com/articles/71082)
4.  [Java8：Lambda表达式增强版Comparator和排序](http://www.importnew.com/15259.html)
5.  [guava之Lists常用示例及newArrayListWithExpectedSize()和newArrayListWithCapacity()详细对比](https://blog.csdn.net/qq_27093465/article/details/53116390)


## 微信公众号

<center>
<img src="https://images.gitee.com/uploads/images/2018/0717/215030_8e782063_912956.png" width="50%" height="50%"/>
</center>
