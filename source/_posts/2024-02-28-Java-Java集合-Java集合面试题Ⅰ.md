---
title: Java 集合面试题总结 Ⅰ
date: 2024-02-28 21:56:43
tags: 
  - Java
categories: 
  - Interview
password: zzy   
message: 仅管理员可见
---

# 集合概述

## 使用集合目的

* Java 集合提供**更灵活、更有效**的方法来存储数据对象，各种集合类和接口可以存储不同类型和数量的对象，同时具有多样化的操作方式
* Java 集合的优势在于它们的**大小可变、支持泛型、具有内建算法**等

## Java 集合概览

> Java 集合两大接口：

| 接口       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| Collection | 主要用于存放单一元素，其下子接口： List、Set、Queue          |
| Map        | 主要用于存放键值对，其下子接口：HashMap、HashTable、SortedMap |

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E9%9B%86%E5%90%88-01.png)

 注：图中只列举主要的继承派生关系，并没有列举所有关系

## List, Set, Queue, Map 区别

| 集合  | 说明                                                         |
| ----- | ------------------------------------------------------------ |
| List  | 存储的元素是有序的、可重复的                                 |
| Set   | 存储的元素不可重复的                                         |
| Queue | 按特定的排队规则来确定先后顺序，存储的元素是有序的、可重复的 |
| Map   | 使用键值对存储，key 是无序的、不可重复的，value 是无序的、可重复的，每个键最多映射到一个值 |



## 集合框架底层数据结构

###  Collection 接口的集合

> List

| List 集合  | 底层实现                                                 |
| ---------- | -------------------------------------------------------- |
| ArrayList  | 基于 Object[] 数组实现                                   |
| Vector     | 基于 Object[] 数组实现                                   |
| LinkedList | 基于双向链表实现（JDK 1.6 前循环链表，JDK 1.7 取消循环） |

> Set

| Set 集合              | 底层实现                                  |
| --------------------- | ----------------------------------------- |
| HashSet（无序，唯一） | 基于 HashMap  实现                        |
| LinkedHashSet         | HashSet 子类，并且基于 LinkedHashMap 实现 |
| TreeSet（有序，唯一） | 基于红黑树实现                            |

> Queue

| Queue 集合    | 底层实现                                    |
| ------------- | ------------------------------------------- |
| PriorityQueue | 基于 Object[] 数组实现                      |
| DelayQueue    | 基于 PriorityQueue 实现                     |
| ArrayDeque    | 可扩容动态双向数组，基于 Object[]  数组实现 |

###  Map 接口集合

> HashMap

* JDK 1.8 前 HashMap 由 **数组+链表** 组成的，数组是 HashMap 主体，链表主要为解决哈希冲突而存在（**链地址法**）
* JDK 1.8 后在解决哈希冲突时有较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为**红黑树**，以减少搜索时间

> LinkedHashMap

* LinkedHashMap 继承自 HashMap，其底层仍然基于**数组和链表或红黑树**组成
* LinkedHashMap 在上面结构的基础上增加了一条**双向链表**，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现访问顺序相关逻辑

> Hashtable

基于 **数组+链表**，数组是 Hashtable 主体，链表则是主要为解决哈希冲突而存在

> TreeMap

基于**红黑树**实现



## 正确选用集合

> 需要根据键值获取到元素值，选用 `Map` 接口集合

| Map 集合          | 适用场景                                     |
| ----------------- | -------------------------------------------- |
| TreeMap           | 按顺序存储键值对，或者需要使用特定的排序方式 |
| HashMap           | 更关心性能和不需要排序                       |
| ConcurrentHashMap | 保证线程安全                                 |

> 只需要存放元素值，选用 `Collection` 接口集合

**需要保证元素唯一**，选择实现 `Set` 接口的集合：

| Set 集合 | 适用场景                     |
| -------- | ---------------------------- |
| HashSet  | 无序集合，且不关心元素的排序 |
| TreeSet  | 有序集合                     |

**不需要保证元素唯一**，选择实现 `List` 接口的集合：

| List 集合  | 适用场景                                             |
| ---------- | ---------------------------------------------------- |
| ArrayList  | 涉及随机访问和读取操作，并且不经常进行插入和删除     |
| LinkedList | 涉及频繁进行插入和删除操作，特别是在列表中间进行操作 |



# List

## ArrayList 和 数组 区别

> `ArrayList` 内部基于动态数组实现，比 `Array`（静态数组） 使用起来更加灵活： 

* `ArrayList` 会根据实际存储的元素**支持动态扩容**，而 `Array` 创建后无法改变长度

* `ArrayList` **允许使用泛型**来确保类型安全，而`Array` 不支持使用泛型

* `ArrayList` **只能存储对象**，对于基本类型数据，需要使用其对应的包装类。而 `Array` 可以直接存储基本类型数据或对象

* `ArrayList` 支持**插入、删除、遍历**等常见操作。而 `Array` 只是固定长度的数组，只能按索引访问元素，不具备动态添加、删除元素的能力

* `ArrayList`创建时无需指定大小，而 `Array` 创建时必须指定大小 

## ArrayList 和 Vector 区别

> 【了解】

**线程是否安全：**

- `ArrayList` 是 `List` 的主要实现类，底层使用 `Object[]`存储，适用于频繁的查找工作，**线程不安全**
- `Vector` 是 `List` 的古老实现类，底层使用`Object[]` 存储，**线程安全**

## Vector 和 Stack 区别

> 【了解】

- `Vector` 和 `Stack` 两者**都是线程安全**的集合，都使用 `synchronized` 关键字进行同步处理
- `Stack` 继承自 `Vector`，属于栈，而 `Vector` 是一个列表

## ArrayList 可以添加 null 值

ArrayList 中可以存储任何类型的对象，包括 `null` 值。但不建议向`ArrayList` 中添加 `null` 值， `null` 值无意义，并且会使代码难以维护

```java
ArrayList<String> listOfStrings = new ArrayList<>();
listOfStrings.add(null);
listOfStrings.add("java");
// [null, java]
System.out.println(listOfStrings);
```



## ArrayList 插入和删除元素的时间复杂度

> 插入：

- 头部插入：由于需要将所有元素都依次向后移动一个位置，因此时间复杂度是 $O(n)$
- 尾部插入：
  - 当 `ArrayList` 的容量未达到极限时，往列表末尾插入元素的时间复杂度是 $O(1)$，因为它只需要在数组末尾添加一个元素即可；
  - 当容量已达到极限并且需要扩容时，则需要执行一次 $O(n)$ 的操作将原数组复制到新的更大的数组中，然后再执行 $O(1)$ 的操作添加元素。
- 指定位置插入：需要将目标位置后的所有元素都向后移动一个位置，然后再把新元素放入指定位置。这个过程需要移动平均 $n/2$ 个元素，因此时间复杂度为 $O(n)$

> 删除：

- 头部删除：由于需要将所有元素依次向前移动一个位置，因此时间复杂度是 $O(n)$
- 尾部删除：当删除的元素位于列表末尾时，时间复杂度为 $O(1)$
- 指定位置删除：需要将目标元素之后的所有元素向前移动一个位置以填补被删除的空白位置，因此需要移动平均 $n/2$ 个元素，时间复杂度为 $O(n)$

```java
// ArrayList的底层数组大小为10，此时存储了7个元素
+---+---+---+---+---+---+---+---+---+---+
| 1 | 2 | 3 | 4 | 5 | 6 | 7 |   |   |   |
+---+---+---+---+---+---+---+---+---+---+
  0   1   2   3   4   5   6   7   8   9
// 在索引为1的位置插入一个元素8，该元素后面的所有元素都要向右移动一位
+---+---+---+---+---+---+---+---+---+---+
| 1 | 8 | 2 | 3 | 4 | 5 | 6 | 7 |   |   |
+---+---+---+---+---+---+---+---+---+---+
  0   1   2   3   4   5   6   7   8   9
// 删除索引为1的位置的元素，该元素后面的所有元素都要向左移动一位
+---+---+---+---+---+---+---+---+---+---+
| 1 | 2 | 3 | 4 | 5 | 6 | 7 |   |   |   |
+---+---+---+---+---+---+---+---+---+---+
  0   1   2   3   4   5   6   7   8   9
```

## LinkedList 插入和删除元素的时间复杂度

* 头部插入/删除：只需要修改头结点的指针即可完成插入/删除操作，因此时间复杂度为 $O(1)$

* 尾部插入/删除：只需要修改尾结点的指针即可完成插入/删除操作，因此时间复杂度为 $O(1)$

* 指定位置插入/删除：需要先移动到指定位置，再修改指定节点的指针完成插入/删除，因此需要移动平均 $n/2$ 个元素，时间复杂度为 $O(n)$

> 假如我们要删除节点 9 的话，需要先遍历链表找到该节点。然后，再执行相应节点指针指向的更改

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E9%9B%86%E5%90%88-02.png)

## LinkedList 为什么不能实现 RandomAccess 接口

- `RandomAccess` 是一个标记接口，用来表明实现该接口的类支持随机访问（即可以通过索引快速访问元素）。
- 由于  `LinkedList ` 底层数据结构是链表，内存地址不连续，只能通过指针来定位，不支持随机快速访问，所以无法实现 `RandomAccess`  接口

## ArrayList 与 LinkedList 区别

> **都不保证线程安全：** 

`ArrayList` 和 `LinkedList` 都是不同步的，也就是都**不保证线程安全**；

> **底层数据结构：** 

-  `ArrayList` 底层基于 `Object` 数组，内部维护可变数组，当数组空间不足时会进行自动扩容
-  `LinkedList` 底层使用的是 **双向链表** 数据结构（JDK 1.6 前为循环链表，JDK1.7 取消循环）

> **插入和删除性能** 

*  `ArrayList` 采用数组存储
   *  插入和删除元素的时间复杂度受元素位置的影响：
   *  在末尾插入和删除操作性能较好，它只需要在数组中添加或删除元素，时间复杂度为 $O(1)$
   *  在中间插入或删除元素可能涉及到数组元素的移动，时间复杂度为  $O(n)$
*  `LinkedList` 采用链表存储：
   * 在头尾插入或者删除元素不受元素位置的影响，时间复杂度为 $O(1)$
   * 在指定位置 `i` 插入和删除元素，需要先移动到指定位置再插入和删除，时间复杂度为 $O(n)$  

>  **是否支持快速随机访问**：

*  `LinkedList` 不支持高效的随机元素访问，在随机访问方面性能较差，需要遍历链表，时间复杂度为 $O(n)$

*  `ArrayList` 由于是基于数组实现的，可以通过索引进行随机访问，时间复杂度为 $O(1)$

*  快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。 

> **内存空间占用：** 

*  `ArrayList`：通常在空间效率上更好，它不需要存储额外的链表节点指针，只需存储元素本身
*  `LinkedList`：需要额外的内存来存储每个节点的前后指针，可能导致占用更多的内存  

> **迭代性能**

-  `ArrayList`：在迭代元素时性能较好，因为可以通过数组的连续存储结构更快地访问元素。 
-  `LinkedList`：在迭代时需要遍历链表，性能相对较差。 

> **适用场景**

-  `ArrayList` 适用于读取操作频繁、随机访问多、插入和删除操作相对较少的场景。 
-  `LinkedList`适用于插入和删除操作频繁、读取操作相对较少的场景，尤其在列表中间进行操作

> 拓展：`RandomAccess`  接口

```java
public interface RandomAccess {
}
```

 查看源码我们发现实际上 `RandomAccess` 接口中什么都没有定义，仅用于标识实现这个接口的类具有随机访问功能 

 在 `binarySearch（)` 方法中，它要判断传入的 list 是否 `RandomAccess` 的实例，  如果是，调用`indexedBinarySearch()`方法，如果不是，那么调用`iteratorBinarySearch()`方法 

```java
public static <T>
int binarySearch(List<? extends Comparable<? super T>> list, T key) {
    if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
        return Collections.indexedBinarySearch(list, key);
    else
        return Collections.iteratorBinarySearch(list, key);
}
```

-  `ArrayList` 底层是数组，而 `LinkedList` 底层是链表 。
-  数组天然支持随机访问，时间复杂度为 O(1)，支持快速随机访问。

-  链表需要遍历到特定位置才能访问特定位置的元素，时间复杂度为 O(n)，不支持快速随机访问 。

`ArrayList` 实现了 `RandomAccess` 接口，就表明了他具有快速随机访问功能。  `RandomAccess` 接口只是标识，并不是说 `ArrayList` 实现 `RandomAccess` 接口才具有快速随机访问功能的！ 

#  Set

## Comparable 和 Comparator 区别

 `Comparable`  接口和 `Comparator` 接口都是 Java 中用于排序的接口，**它们在实现类对象之间比较大小、排序等方面发挥重要作用**： 

>  **位置：** 

Comparable 接口：

- Comparable 接口在目标类的定义中实现，表示对象自己的自然排序方式

  ```java
  public class Person implements Comparable<Person> {
  ```

- 通过实现 Comparable 接口，对象告诉其他对象如何比较它们自己 

Comparator 接口：

- Comparator 接口是一个单独的比较器，不依赖于被比较的对象

- 它通常作为独立的类或者作为匿名内部类来创建

  ```java
  public class PersonByNameComparator implements Comparator<Person> {
  ```

- 通过创建 Comparator 的实例，可以在不修改类本身的情况下定义多种不同的排序方式

>  **方法：** 

Comparable 接口：

- Comparable 接口中只有方法：`compareTo(Object obj)`

- 该方法用于定义**对象与其他对象的比较规则**

  ```java
  @Override
  public int compareTo(Person otherPerson) {
  ```

- 返回值为负数表示当前对象小于传入对象，零表示相等，正数表示当前对象大于传入对象。 

 Comparator 接口：

- Comparator 接口中有两个方法：`compare(Object o1, Object o2)` 和 `equals(Object obj)`

  ```java
  @Override
  public int compare(Person person1, Person person2) {
  ```

- 主要关注`compare`方法，该方法用于**定义两个对象之间的比较规则**

> **排序方式：**

- 通过实现`Comparable`接口，对象定义自己的自然排序方式。例如，对于数字，自然排序就是升序（**自然排序**）
- 通过实现`Comparator`接口，可以在不修改目标类的情况下定制多种不同的排序方式（**定制排序**）

> **适用场景**

-  如果一个类天生具备一种自然的比较方式，可以实现`Comparable`接口，使其支持自然排序 
-  如果需要多种排序方式，或者要在不改变类本身的情况下定义排序，可以使用`Comparator`接口

> 代码演示

**`Comparable`接口：**

 假设有一个`Person`类，它具有姓名和年龄属性，我们希望按照年龄进行自然排序。 

```java
public class Person implements Comparable<Person> {
    private String name;
    private int age;

    // 构造函数、getter和setter方法省略

    @Override
    public int compareTo(Person otherPerson) {
        // 按照年龄进行自然排序
        return Integer.compare(this.age, otherPerson.age);
    }

    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + '}';
    }
}
```

 然后，可以使用`Collections.sort()`进行排序： 

```java
public class ComparableExample {
    public static void main(String[] args) {
        List<Person> people = new ArrayList<>();
        people.add(new Person("Alice", 30));
        people.add(new Person("Bob", 25));
        people.add(new Person("Charlie", 35));

        Collections.sort(people);

        for (Person person : people) {
            System.out.println(person);
        }
    }
}
```

 **`Comparator`接口：  **

 假设我们有同样的`Person`类，但是这次我们想按照姓名进行排序。 

```java
import java.util.Comparator;

public class PersonByNameComparator implements Comparator<Person> {
    @Override
    public int compare(Person person1, Person person2) {
        // 按照姓名进行排序
        return person1.getName().compareTo(person2.getName());
    }
}
```

 然后，我们可以使用`Collections.sort()`并传入自定义的`Comparator` 

```java
public class ComparatorExample {
    public static void main(String[] args) {
        List<Person> people = new ArrayList<>();
        people.add(new Person("Alice", 30));
        people.add(new Person("Bob", 25));
        people.add(new Person("Charlie", 35));

        Collections.sort(people, new PersonByNameComparator());

        for (Person person : people) {
            System.out.println(person);
        }
    }
}
```



## 无序性和不可重复性

- **无序性不等于随机性** ，无序性是指存储的数据在底层数组中并非按照数组索引的顺序添加 ，而是**根据数据的哈希值决定**的。

- 不可重复性是指添加的元素按照 `equals()` 判断时 ，返回 false，**需要同时重写** `equals()` 方法和 `hashCode()` 方法 

## HashSet、LinkedHashSet 和 TreeSet 区别

`HashSet`、`LinkedHashSet` 和 `TreeSet` 都是 `Set` 接口的实现类，**都能保证元素唯一，并且线程都不安全**

> **HashSet**

-  **存储顺序：** **不保证**元素存储顺序，具体存储顺序由元素的哈希码决定（**不保证顺序**）
-  **底层实现：** 基于**哈希表**实现，使用 HashMap 存储元素
-  **性能：** 插入、删除和查找操作的性能通常为常数时间复杂度，具有较好的性能 
-  **存储 null**：允许存储 `null` 值

> **TreeSet**

-  **存储顺序：** 按照元素的自然顺序或者通过提供的 Comparator 进行排序（**保证顺序**）
-  **底层实现：** 基于**红黑树**实现，元素有序，排序的方式有自然排序和定制排序 

-  **性能：** 插入、删除和查找操作性能通常为 $O(log n)$。相较于 HashSet 和 LinkedHashSet，TreeSet 在排序方面具有更好的性能

-  **存储 null**：不允许存储`null`值的。`TreeSet`是基于红黑树实现的，而红黑树的节点不能为`null`

> **LinkedHashSet**

-  **存储顺序：**按照插入顺序存储元素，元素的插入和取出顺序满足 FIFO （**保证顺序**）
-  **底层实现：** 基于哈希表和链表实现，使用 **LinkedHashMap** 来存储元素
-  **性能**：插入、删除和查找操作性能通常为常数时间复杂度，但略低于 HashSet，因为还需要维护插入顺序。 
-  **存储 null**：允许存储 `null` 值

> **适用场景**

| Set 集合      | 适用场景                                                   |
| ------------- | ---------------------------------------------------------- |
| HashSet       | 高性能的插入、删除和查找操作，并不关心元素的顺序           |
| LinkedHashSet | 保留元素的插入顺序，并且仍然需要较好的插入、删除和查找性能 |
| TreeSet       | 元素有序，并且能够按照自然顺序或者提供 Comparator 进行排序 |

# Queue

## Queue 与 Deque 区别

> Queue

*  Queue 是单端队列，只能从一端插入元素，另一端删除元素，一般遵循 **先进先出（FIFO）** 规则
*  Queue 扩展了 `Collection` 接口， **因为容量问题而导致操作失败后处理方式的不同** 可以分为两类方法
   * 一种在操作失败后会抛出异常
   * 另一种则会返回特殊值 

| `Queue` 接口 | 抛出异常  | 返回特殊值 |
| ------------ | --------- | ---------- |
| 插入队尾     | add(E e)  | offer(E e) |
| 删除队首     | remove()  | poll()     |
| 查询队首元素 | element() | peek()     |

> Deque

*  Deque 是双端队列，在队列的两端均可以插入或删除元素

*  Deque 扩展了 `Queue`接口，增加在队首和队尾进行插入和删除的方法，同样根据失败后处理方式的不同分为两类： 

| `Deque` 接口 | 抛出异常      | 返回特殊值      |
| ------------ | ------------- | --------------- |
| 插入队首     | addFirst(E e) | offerFirst(E e) |
| 插入队尾     | addLast(E e)  | offerLast(E e)  |
| 删除队首     | removeFirst() | pollFirst()     |
| 删除队尾     | removeLast()  | pollLast()      |
| 查询队首元素 | getFirst()    | peekFirst()     |
| 查询队尾元素 | getLast()     | peekLast()      |

> 适用场景

| 队列  | 适用场景                                                     |
| ----- | ------------------------------------------------------------ |
| Queue | 只关心先进先出的队列操作，不需要在队列两端进行操作           |
| Deque | 需要在队列两端进行添加、删除操作，或者需要将其用作栈（先进后出） |



## ArrayDeque 与 LinkedList 区别

 `ArrayDeque` 和 `LinkedList` 都实现  `Deque` 接口，两者都具有队列的功能，但两者有什么区别呢？ 

> **ArrayDeque**

- **底层实现：**  `ArrayDeque` 基于动态数组实现，**线程不安全**
- **随机访问：**  `ArrayDeque` 支持通过索引访问元素，随机访问性能较好，时间复杂度为 $O(1)$
- **空间利用**： 它不需要为每个元素额外存储指针， `ArrayDeque` 空间利用更有效
- **存储 null：**`ArrayDeque` **不支持**存储 `null`数据

> **LinkedList**

-  **底层实现：**  `LinkedList` 基于双向链表实现，**线程不安全**
-  **插入删除性能**：与`ArrayDeque`相比，它对插入和删除操作的支持更好。 
-  **迭代性能**： `LinkedList`在迭代时性能较差，时间复杂度为 $O(n)$
-  **存储 null：** `LinkedList` **支持**存储 `null`数据

> 适用场景

| 队列       | 适用场景                                                     |
| ---------- | ------------------------------------------------------------ |
| ArrayDeque | 当你需要高性能的栈和队列操作，尤其是在随机访问方面的性能。<br>当你关心空间效率，因为它相对更省空间。 |
| LinkedList | 当你需要在中间进行频繁的插入和删除操作，而不关心随机访问的性能。<br>当你需要使用`List`和`Deque`的混合特性，例如在队列两端添加或删除元素，以及在中间插入或删除元素。 |



##  PriorityQueue

 `PriorityQueue`  其与 `Queue` 的区别在于元素出队顺序是与优先级相关的，即总是优先级最高的元素先出队。

- **优先级队列：**  `PriorityQueue` 其中的元素按照其**自然顺序**或者通过提供的`Comparator`进行排序。排序顺序决定了队列中元素的出队顺序。 

- **底层实现：** `PriorityQueue` 利用**二叉堆**的数据结构实现，底层使用**可变长的数组**来存储数据。默认情况下，元素按照自然顺序进行排序，或者可以通过提供的`Comparator`进行自定义排序。 

- **元素插入和删除：**  插入元素和删除元素（出队）的时间复杂度为 $O(log n)$
- **线程不安全**：`PriorityQueue` 是**非线程安全**的
- **存储 null：** **不支持存储** `null` 和 `non-comparable` 的对象。

- `PriorityQueue` **默认是小顶堆**，但可以接收一个 `Comparator` 作为构造参数，从而来自定义元素优先级的先后。

> 【掌握】

`PriorityQueue` 在面试中可能更多的会出现在手撕算法的时候，典型例题包括堆排序、求第 K 大的数、带权图的遍历等，所以需要会熟练使用才行。 

## BlockingQueue

> BlockingQueue

```java
public interface BlockingQueue<E> extends Queue<E> {
  // ...
}
```

- BlockingQueue 是 Java 集合框架中的接口，表示一种**线程安全**的队列，特别适用于多线程之间的数据交换

- Java 8 BlockingQueue 提供对阻塞操作的支持，可以在队列为空或队列已满时阻塞线程，从而协调多线程之间的操作
- BlockingQueue 常用于生产者-消费者模型中，生产者线程会向队列中添加数据，而消费者线程会从队列中取出数据进行处理。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E9%9B%86%E5%90%88-03.png)



```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class BlockingQueueExample {
    public static void main(String[] args) {
        // 创建一个容量为3的阻塞队列
        BlockingQueue<Integer> blockingQueue = new ArrayBlockingQueue<>(3);

        // 生产者线程
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    blockingQueue.put(i);
                    System.out.println("Produced: " + i);
                    Thread.sleep(1000);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        // 消费者线程
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    Integer value = blockingQueue.take();
                    System.out.println("Consumed: " + value);
                    Thread.sleep(1500);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        // 启动线程
        producer.start();
        consumer.start();
    }
}
```

BlockingQueue 被用来在生产者线程和消费者线程之间传递数据。`put` 和 `take` 方法的调用实现阻塞的生产者-消费者模型。

* 在队列为空时，消费者线程会被阻塞，直到有数据可用
* 在队列满时，生产者线程会被阻塞，直到有空间可用
* 这种机制可以很好地协调多线程之间的操作。 

> 方法说明

 **阻塞操作：** 

| 方法     | 说明                                                     |
| -------- | -------------------------------------------------------- |
| put(E e) | 将元素添加到队列，如果队列已满，则阻塞直到队列有可用空间 |
| take()   | 从队列中取出元素，如果队列为空，则阻塞直到队列非空。     |

 **超时操作：** 

| 方法                                    | 说明                                                     |
| --------------------------------------- | -------------------------------------------------------- |
| offer(E e, long timeout, TimeUnit unit) | 在指定时间内将元素添加到队列，如果队列仍满，则返回 false |
| poll(long timeout, TimeUnit unit)       | 在指定时间内从队列中取出元素，如果队列为空，则返回 null  |

 **非阻塞操作：** 

| 方法       | 说明                                               |
| ---------- | -------------------------------------------------- |
| offer(E e) | 尝试将元素添加到队列，如果队列已满则立即返回 false |
| poll()     | 尝试从队列中取出元素，如果队列为空则立即返回 null  |

 **其他方法：** 

| 方法                | 说明                   |
| ------------------- | ---------------------- |
| remainingCapacity() | 返回队列中剩余的容量   |
| size()              | 返回队列的当前大小     |
| remove(Object o)    | 从队列中删除指定的元素 |

> BlockingQueue 实现类

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E9%9B%86%E5%90%88-04.png)

`ArrayBlockingQueue`

- **底层实现：**使用**数组**实现的**有界**阻塞队列。创建时需要指定队列容量，并支持公平和非公平两种方式的锁访问机制
- **阻塞操作：** 当队列满时，`put`操作会阻塞直到队列有空间。当队列为空时，`take`操作会阻塞直到队列非空。 

```java
BlockingQueue<Integer> arrayBlockingQueue = new ArrayBlockingQueue<>(10);
```

`LinkedBlockingQueue`

- **底层实现：**使用**单向链表**实现的**可选有界**阻塞队列。可以选择在创建时指定容量（有界）或者不指定容量（无界），如果不指定则默认为`Integer.MAX_VALUE`。  和`ArrayBlockingQueue`类似， 它也支持公平和非公平的锁访问机制。 
- **阻塞操作：** 与`ArrayBlockingQueue`类似，`put`和`take`操作会根据队列状态进行阻塞。 

```java
BlockingQueue<Integer> linkedBlockingQueue = new LinkedBlockingQueue<>();
```

`PriorityBlockingQueue`

-  **底层实现：** 基于优先级堆的**无界**阻塞队列，元素按照优先级进行排序。  元素必须实现`Comparable`接口或者在构造函数中传入`Comparator`对象，并且**不能插入** null 元素。
-  **阻塞操作：** `put`和`take`操作会根据元素的优先级进行阻塞。 

```java
BlockingQueue<Integer> priorityBlockingQueue = new PriorityBlockingQueue<>();
```

`DelayQueue`

- **底层实现：** 基于优先级堆的**无界**阻塞队列，用于延迟元素的消费，其中的元素只有到其指定的延迟时间，才能够从队列中出队。
- **阻塞操作：** `take`操作会根据元素的延迟时间进行阻塞，直到延迟时间到达。

```java
BlockingQueue<DelayedElement> delayQueue = new DelayQueue<>();
```

`SynchronousQueue`

- **底层实现：** 同步队列，一个**不存储元素**的阻塞队列，每个插入操作都必须等待对应的删除操作，反之删除操作也必须等待插入操作，  `SynchronousQueue`通常用于线程之间的直接传递数据。 
- **阻塞操作：** 插入和删除操作是成对的，没有缓冲区，元素不存储。  

```java
BlockingQueue<Integer> synchronousQueue = new SynchronousQueue<>();
```

> 适用场景

| 阻塞队列              | 适用场景                              |
| --------------------- | ------------------------------------- |
| ArrayBlockingQueue    | 用于有限的、固定大小的队列            |
| LinkedBlockingQueue   | 用于无限或者动态大小的队列            |
| PriorityBlockingQueue | 用于按优先级排序的场景                |
| DelayQueue            | 用于延迟执行任务                      |
| SynchronousQueue      | 用于实现生产者-消费者模型的线程间通信 |



## ArrayBlockingQueue 和 LinkedBlockingQueue

ArrayBlockingQueue 和 LinkedBlockingQueue 都是**线程安全**的：

> **底层实现**： 

- `ArrayBlockingQueue`是基于**数组**的**有界**阻塞队列。在创建时需要指定队列的容量 
- `LinkedBlockingQueue`是基于**链表**的阻塞队列。可以选择在创建时指定容量（有界）或者不指定容量（无界） 

> **容量限制：** 

-  `ArrayBlockingQueue` 由于是有界队列，因此它的容量是固定的，无法动态扩展，必须在创建时指定容量大小 
-  `LinkedBlockingQueue`  可以选择创建一个无界队列，也可以在创建时指定容量，成为有界队列。 创建时可以不指定容量大小，默认是`Integer.MAX_VALUE`

 **阻塞操作：** 

- `ArrayBlockingQueue`， 当队列满时，`put`操作会阻塞直到队列有空间。 当队列为空时，`take`操作会阻塞直到队列非空 
- `LinkedBlockingQueue`的`put`和`take`操作也会根据队列状态进行阻塞 

> **性能：** 

-  由于使用数组实现，`ArrayBlockingQueue`在高负载情况下可能具有更好的性能。 

-  由于使用链表实现，`LinkedBlockingQueue`在某些情况下可能对于插入和删除操作的性能更好。 

> **内存占用：**

- `ArrayBlockingQueue` 需要**提前分配**数组内存，而 `LinkedBlockingQueue` 则是**动态分配**链表节点内存。 
- `ArrayBlockingQueue` 在创建时就会占用一定的内存空间，且往往申请的内存比实际所用的内存更大，而`LinkedBlockingQueue` 则是根据元素的增加而逐渐占用内存空间。 

> **锁是否分离：**

-  `ArrayBlockingQueue`中的**锁没有分离**的，即**生产和消费用的是同一个锁**；
-  `LinkedBlockingQueue`中的**锁是分离**的，即生产用的是`putLock`，消费是`takeLock`，可以防止生产者和消费者线程之间的锁争夺。 

**适用场景：**

| 阻塞队列            | 适用场景                                                     |
| ------------------- | ------------------------------------------------------------ |
| ArrayBlockingQueue  | 当需要一个固定大小的有界队列<br>在高负载情况下，由于底层是数组实现，可能具有更好的性能 |
| LinkedBlockingQueue | 当需要一个动态大小的队列，或者需要使用无界队列<br>插入和删除操作的性能对于应用的性能更为重要 |

