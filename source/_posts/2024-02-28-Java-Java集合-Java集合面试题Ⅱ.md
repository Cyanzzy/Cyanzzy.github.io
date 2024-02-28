---
title: Java 集合面试题总结 Ⅱ
date: 2024-02-28 21:56:52
tags: 
  - Java
categories: 
  - Interview
password: zzy   
message: 仅管理员可见
---

#  Map

## HashMap 和 Hashtable 区别

> **线程是否安全：** 

 **HashMap：** 

-  `HashMap`是非线程安全的。多个线程可以同时访问和修改 `HashMap`，但如果多个线程同时修改`HashMap`，可能导致不一致的结果。  
-  在多线程环境下使用`HashMap`时，可以考虑使用`Collections.synchronizedMap`方法将其包装为线程安全的Map。 

 **Hashtable：** 

-  `Hashtable`是线程安全的。所有的方法都是同步的，保证在多线程环境下的一致性。 
-  尽管提供线程安全性，但在大多数情况下，`ConcurrentHashMap`更常用，因为它在并发性能上比`Hashtable`更好。 

> **对 Null 键和值的支持：**

- `HashMap`  允许使用`null`作为键和值 ，但 `null` 作为键只能有一个，`null` 作为值可以有多个
- `Hashtable`  不允许使用`null`作为键或值 ， 如果尝试插入`null`键或值，`Hashtable`会抛出`NullPointerException`。 

> **性能：**

- `HashMap` 通常比`Hashtable`具有更好的性能。在非多线程环境下，`HashMap`是首选的选择。 
- 在多线程环境下，`Hashtable`的性能受到同步的影响，可能较慢。 更推荐使用`ConcurrentHashMap`来实现线程安全的Map。 

> **底层数据结构：**   

**HashMap：**

-  `HashMap` 基于**数组 + 链表**实现（或红黑树，Java 8 开始引入） 
-  桶数组中的每个桶存储一个链表或红黑树，用于解决哈希冲突

-  JDK 1.8 以后在解决哈希冲突时有较大变化，当链表长度大于阈值（默认为 8）时，将链表转化为红黑树（将链表转换成红黑树前会判断，如果当前数组的长度小于 64 ，那么会选择先进行数组扩容，而不是转换为红黑树），以减少搜索时间

 **Hashtable：** 

- `Hashtable`基于**哈希表的数组**实现
- 每个元素（键值对）通过哈希函数计算索引，然后存储在对应的数组位置

> **初始容量大小和扩容机制：**  

 **HashMap：** 

- 创建时如果不指定容量初始值，HashMap 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。 
- 创建时如果给定容量初始值， `HashMap` 会将其扩充为 $2^n$

- `HashMap` 在达到一定的负载因子时（默认 0.75），会触发扩容操作。扩容会重新计算哈希值，重新分配桶，并将元素重新放置到新的桶中。 

 **Hashtable：** 

- 创建时如果不指定容量初始值，`Hashtable` 默认的初始大小为 11，之后每次扩充，容量变为原来的 $2n+1$
- 创建时如果给定容量初始值，那么 `Hashtable` 会直接使用你给定的大小

- `Hashtable`在达到一定的负载因子时（默认为0.75），会自动调整大小。它将数组大小扩展为原来的两倍加一，并重新调整哈希函数。 

>  **继承关系：**

- `HashMap`  继承自 `AbstractMap` 类，实现了`Map`接口。

  ```java
  public class HashMap<K,V> extends AbstractMap<K,V>
      implements Map<K,V>, Cloneable, Serializable {
  ```

- `Hashtable` 继承自`Dictionary`类，实现了`Map`接口。`Dictionary`类已经过时，推荐使用`Map`的实现类。 

  ```java
  public class Hashtable<K,V>
      extends Dictionary<K,V>
      implements Map<K,V>, Cloneable, java.io.Serializable {
  ```

> **迭代器（Iterator）：**

-  `HashMap` 的迭代器是快速失败（fail-fast）的。如果在迭代过程中对`HashMap`进行结构上的修改，会抛出`ConcurrentModificationException`。 

-  `Hashtable`的迭代器是安全的，不会抛出`ConcurrentModificationException`。但由于同步的代价，它在并发修改的情况下可能会导致低效。 

> **适用场景**

-  `HashMap`通常比`Hashtable`具有更好的性能，并且更灵活，可以用于非多线程环境。 
-  如果线程不安全下使用 `HashMap`，可以使用`Collections.synchronizedMap`方法将`HashMap`包装成线程安全的Map。 
-  在多线程环境下，推荐使用更现代的线程安全Map实现，如`ConcurrentHashMap`，而不是`Hashtable`。 

## HashMap 和 HashSet 区别

`HashSet` 是基于 `HashMap` 实现的  。（`HashSet` 的源码非常非常少，因为除了 `clone()`、`writeObject()`、`readObject()`是 `HashSet` 自己不得不实现之外 ，其他方法都是直接调用 `HashMap` 中的方法。 

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    static final long serialVersionUID = -5024744406713321676L;

    private transient HashMap<E,Object> map;
```



> **数据结构：**

 **HashMap：** 

- `HashMap`是键值对的集合，可以存储键值对。每个元素都是一个键值对，通过键来访问值 
- 底层基于**数组 + 链表**实现（或红黑树，从 Java 8 开始引入）

 **HashSet：** 

- `HashSet`是一组唯一元素的集合，不允许重复元素

- 底层基于 **HashMap** 实现，**但只存储键而没有对应的值**。实际上，`HashSet`就是`HashMap`的一个特例，只使用了键而没有值

  ```java
  private transient HashMap<E,Object> map;
  
  // Dummy value to associate with an Object in the backing Map
  private static final Object PRESENT = new Object();
  
  public boolean add(E e) {
      return map.put(e, PRESENT)==null;
  }
  ```

> **存储方式：**

- `HashMap` 存储键值对，每个元素由键和值组成。

- `HashSet` 存储独一无二的元素，每个元素只有一个值。 

> **方法调用：**

 **HashMap：** 

| 方法            | 说明           |
| --------------- | -------------- |
| put(key, value) | 插入键值对。   |
| get(key)        | 根据键获取值。 |

 **HashSet：** 

| 方法              | 说明                   |
| ----------------- | ---------------------- |
| add(element)      | 插入元素。             |
| contains(element) | 检查是否包含某个元素。 |

> **使用场景**：

| 数据结构 | 适用场景                                                     |
| -------- | ------------------------------------------------------------ |
| HashMap  | 需要存储键值对，通过键来查找值的场景                         |
| HashSet  | 需要存储独一无二元素的场景，不关心键值对关系，只关心元素的唯一性 |

|                HashMap                 |                           HashSet                            |
| :------------------------------------: | :----------------------------------------------------------: |
|            实现 `Map` 接口             |                       实现 `Set` 接口                        |
|               存储键值对               |                          仅存储对象                          |
|    调用 `put()`向 `map` 中添加元素     |             调用 `add()`方法向 `Set` 中添加元素              |
| `HashMap` 使用键（Key）计算 `hashcode` | `HashSet` 使用成员对象来计算 `hashcode` 值，对于两个对象来说 `hashcode` 可能相同，所以`equals()`方法用来判断对象的相等性 |

## HashMap 和 TreeMap 区别

> **底层实现**：

 **HashMap：** 

-  `HashMap`的底层基于**哈希表**实现，使用数组和链表（或红黑树）来存储键值对 
-  插入、删除和查找的平均时间复杂度为 $O(1)$，但在某些情况下可能会变为 $O(n)$，取决于哈希碰撞的程度 

 **TreeMap：** 

-  `TreeMap`的底层基于**红黑树**实现，红黑树是一种自平衡的二叉搜索树
-  插入、删除和查找的平均时间复杂度为 $O(log n)$，其中n是元素的数量。由于红黑树的自平衡性质，性能在各种情况下都相对稳定。 

> **元素排序**：

-  `HashMap`不保证元素的顺序。元素的存储顺序取决于哈希码的分布，因此是不可预测的。 
-  `TreeMap` 根据键的自然顺序或者通过提供的`Comparator`进行排序。因此，`TreeMap`中的元素是有序的

> **性能：**

- 在大多数情况下，`HashMap`的性能比`TreeMap`好。它具有 $O(1)$ 的平均时间复杂度，但在具体应用中要注意避免哈希碰撞。 
- `TreeMap`的性能受制于红黑树的自平衡性质，平均时间复杂度为 $O(log n)$ 。对于非常大的数据集，它可能在插入和删除时比`HashMap`慢。 

> **null 键值对支持：**

- `HashMap`  允许使用`null`作为键，允许使用`null`作为值
- `TreeMap` 不允许使用`null`作为键。在插入时，如果键为`null`，会抛出`NullPointerException`，但允许使用 `null`作为值

> **迭代顺序**：

-  `HashMap`的迭代顺序是不确定的，不受键的自然顺序或哈希码的影响 
-  `TreeMap`的迭代顺序是有序的，按照键的自然顺序或提供的`Comparator`的顺序进行 



> `TreeMap` 和`HashMap` 都继承自`AbstractMap` ，但是需要注意的是`TreeMap`它还实现了`NavigableMap`接口和`SortedMap` 接口。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E9%9B%86%E5%90%88-05.png)

- 实现 `NavigableMap` 接口让 `TreeMap` 有了对集合内元素的搜索的能力。 

- 实现`SortedMap`接口让 `TreeMap` 有了对集合中的元素根据键排序的能力。默认是按 key 的升序排序，不过我们也可以指定排序的比较器 

```java
public class Person {
    
    private Integer age;

    public Person(Integer age) {
        this.age = age;
    }

    public Integer getAge() {
        return age;
    }
    
    public static void main(String[] args) {
        TreeMap<Person, String> treeMap = new TreeMap<>(new Comparator<Person>() {
            @Override
            public int compare(Person person1, Person person2) {
                int num = person1.getAge() - person2.getAge();
                return Integer.compare(num, 0);
            }
        });
        treeMap.put(new Person(3), "person1");
        treeMap.put(new Person(18), "person2");
        treeMap.put(new Person(35), "person3");
        treeMap.put(new Person(16), "person4");
        treeMap.entrySet().stream().forEach(personStringEntry -> {
            System.out.println(personStringEntry.getValue());
        });
    }
}
// outputs
person1
person4
person2
person3
```

可以看出 `TreeMap` 中的元素已经是按照 `Person` 的 age 字段的升序来排列了。

上面，我们是通过传入匿名内部类的方式实现的，你可以将代码替换成 Lambda 表达式实现的方式：

````java
TreeMap<Person, String> treeMap = new TreeMap<>((person1, person2) -> {
  int num = person1.getAge() - person2.getAge();
  return Integer.compare(num, 0);
});
````

 **综上，相比于`HashMap`来说 `TreeMap` 主要多了对集合中的元素根据键排序的能力以及对集合内元素的搜索的能力。** 



## HashSet 检查重复的原理

 HashSet 基于哈希表实现，用于存储独一无二的元素，不允许重复。它的底层原理涉及哈希函数、哈希码、桶（buckets）和链表（或红黑树）等概念。 

 **哈希函数：** 

-  `HashSet`使用哈希函数将元素的键（即对象的哈希码）映射到哈希表中的桶。 
-  哈希函数的目标是将元素均匀地分布在哈希表的各个桶中，以最大程度减少哈希冲突。 

 **哈希码和桶：** 

- 对于每个元素，通过调用其 `hashCode()` 方法获取哈希码。 
- 哈希码被映射到哈希表的桶，即数组的一个位置。 

- 桶中存储了链表（或红黑树），用于处理哈希冲突，即多个元素哈希码相同的情况。 

 **检查重复：** 

1. 当把对象加入`HashSet`时，`HashSet` 会先计算对象的`hashcode`， 然后根据哈希码找到对应的桶
2. 在桶中搜索，检查是否存在相同的元素。 如果桶中不存在相同的元素，将新元素添加到桶中。 此时与其他加入的对象的 `hashcode` 值作比较，如果没有相同的 `hashcode`，`HashSet` 会假设对象没有重复出现。 
3. 但是如果发现有相同 `hashcode` 值的对象，这时会调用`equals()`方法来检查 `hashcode` 相等的对象是否真的相等
4. 如果元素相等，则不添加重复元素；如果不相等，则处理哈希冲突，例如通过在链表中添加新元素或将红黑树平衡。 

  **红黑树的引入：** 

-  从Java 8开始，`HashSet`在桶的链表长度超过一定阈值时，会将链表转换为红黑树。 
-  这提高了查找效率，特别是在具有大量元素的情况下，因为红黑树的查找时间复杂度更好（O(log n)）。 

> 在 JDK1.8 中，`HashSet`的`add()`方法只是简单的调用了`HashMap`的`put()`方法，并且判断了一下返回值以确保是否有重复元素。直接看一下`HashSet`中的源码： 

```java
// Returns: true if this set did not already contain the specified element
// 返回值：当 set 中没有包含 add 的元素时返回真
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
}
```

 而在`HashMap`的`putVal()`方法中也能看到如下说明： 

```java
// Returns : previous value, or null if none
// 返回值：如果插入位置没有元素返回null，否则返回上一个元素
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
...
}
```

**也就是说，在 JDK1.8 中，实际上无论`HashSet`中是否已经存在某元素，`HashSet`都会直接插入，只是会在`add()`方法的返回值处告诉我们插入前是否存在相同元素** 

## HashMap 的底层原理

> 【扩展】哈希冲突 

哈希冲突是指两个或更多不同的输入数据经过哈希函数处理后得到相同的哈希码。在哈希表中，哈希冲突可能会发生，因为哈希函数的输出范围是有限的，而输入的数据可能是无限的。

解决哈希冲突的常见方法：

**链地址法（Separate Chaining）：** 该方法中，每个哈希桶都是一个链表。当多个键映射到同一个哈希码时，它们会被存储在同一个桶中的链表中。这样，相同哈希码的键值对可以共享同一个哈希桶，通过链表来存储。

**开放寻址法（Open Addressing）：**该方法中，当发生冲突时，程序会尝试寻找另一个空的哈希桶，而不是使用链表。 

- **线性探测（Linear Probing）：** 当发生哈希冲突时，线性探测会依次检查下一个哈希桶，直到找到一个空的桶或者遍历整个哈希表。这样，冲突的元素会被放置在找到的第一个空桶中。

- **二次探测（Quadratic Probing）：** 它使用二次方程来决定下一个探测的位置。这样可以更均匀地分布元素，避免线性探测的一些问题。

- **双重散列（Double Hashing）：** 它使用第二个哈希函数来决定下一个探测的位置。这可以提供更好的均匀性，减少冲突的概率。



### JDK 1.8  前的原理

- JDK 1.8  前 `HashMap` 底层是 **数组和链表** 结合在一起使用（**链表散列**）
- key 的 hashcode 经扰动函数处理后得到 **hash** 值，所谓扰动函数指的就是 HashMap 的 `hash` 方法，使用扰动函数之后可以减少碰撞。 
- 然后通过运算 `(n - 1) & hash` **判断当前元素存放的位置**（这里的 n 指的是数组的长度）
- 如果当前位置存在元素，则判断该元素与待存入元素的 `hash` 值和 `key` 是否相同，如果相同则直接覆盖，不相同就通过拉链法解决哈希冲突

> **“拉链法（链地址法）”**  ：
>
> 将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E9%9B%86%E5%90%88-06.png)

>  **JDK 1.8 HashMap 的 hash 方法源码:** 

 JDK 1.8 的 hash 方法 相比于 JDK 1.7 hash 方法更加简化，但是原理不变。 

```java
static final int hash(Object key) {
  int h;
  // key.hashCode()：返回散列值也就是hashcode
  // ^：按位异或
  // >>>:无符号右移，忽略符号位，空位都以0补齐
  return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

> 对比一下 JDK1.7 的 HashMap 的 hash 方法源码. 

```java
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).

    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

 相比于 JDK1.8 的 hash 方法 ，JDK 1.7 的 hash 方法的性能会稍差一点点，因为毕竟扰动了 **4** 次。 



###  JDK1.8 及后的原理

 JDK 1.8 后在解决哈希冲突时有较大的变化，当链表长度大于阈值（默认为 **8**）时，将链表转化为红黑树，以减少搜索时间。

注意： 将链表转换成红黑树前会判断，如果当前数组长度小于 **64**，那么先进行数组扩容，而不是转换为红黑树

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E9%9B%86%E5%90%88-07.png)



TreeMap、TreeSet 以及 JDK1.8 之后的 HashMap 底层都用到红黑树。红黑树就是为了解决二叉查找树的缺陷，因为二叉查找树在某些情况下会退化成一个线性结构



> 结合源码分析一下 `HashMap` 链表到红黑树的转换。 

 **1、 `putVal` 方法中执行链表转红黑树的判断逻辑。** 

 链表的长度大于 8 的时候，就执行 `treeifyBin` （转换红黑树）的逻辑。 

```java
// 遍历链表
for (int binCount = 0; ; ++binCount) {
    // 遍历到链表最后一个节点
    if ((e = p.next) == null) {
        p.next = newNode(hash, key, value, null);
        // 如果链表元素个数大于等于TREEIFY_THRESHOLD（8）
        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
            // 红黑树转换（并不会直接转换成红黑树）
            treeifyBin(tab, hash);
        break;
    }
    if (e.hash == hash &&
        ((k = e.key) == key || (key != null && key.equals(k))))
        break;
    p = e;
}
```

 **2、`treeifyBin` 方法中判断是否真的转换为红黑树。** 

 将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树。 

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    // 判断当前数组的长度是否小于 64
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        // 如果当前数组的长度小于 64，那么会选择先进行数组扩容
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        // 否则才将列表转换为红黑树

        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```



## HashMap 容量是 $2^n$ 的原因

> 存在问题：哈希值范围太大，直接构建数组，内存不够（40 亿长度）

为能让 HashMap 存取高效，尽量均匀分配数据。Hash 值的范围值 -2147483648 到 2147483647，前后加起来大概 40 亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的 

但问题是一个 40 亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。 

> 【运算优化】  **hash%length==hash&(length-1)**

*  用之前先做**对数组的长度取模运算，得到余数**才能用来存放数据，即先计算对应的数组下标 “ `(n - 1) & hash`”。（n 代表数组长度） 
*  取模（%）操作中，如果除数是 $2^n$ 则等价于和**其除数减一的&**操作，即  **hash%length==hash&(length-1)的前提是 length 是 2 的 n 次方；**   
*  采用二进制位操作 &，相对于%能够提高运算效率

> 总结

1. 由于 Hash 值的范围值 $[-2^{31}, 2^{31}-1]$，前后 40 亿长度，不能直接用于存储数据
2. 所以存放数据前，先对数组长度 length 取模运算，得到余数才能存储，即 hash % length
3. 而当 除数 length 是2 的幂次方时，刚好等价于 hash&(length-1)，为提高运算效率，将使用 & 运算，同时将容量定义为 $2^n$

## HashMap 多线程操作导致死循环问题

>  JDK1.7 及之前版本的死循环问题

-  JDK1.7 及之前版本的 `HashMap` 在多线程环境下扩容操作可能存在死循环问题
-  当一个桶位中有多个元素需要进行扩容时，多个线程同时对链表进行操作，**头插法**可能会导致链表中的节点指向错误的位置，从而**形成一个环形链表**，进而使得查询元素的操作陷入死循环无法结束。 

> JDK1.8 版本的解决方案

为了解决这个问题，JDK1.8 版本的 HashMap 采用**尾插法**而不是头插法来避免链表倒置，使插入的节点永远都是放在链表的末尾，避免链表产生环形结构。

但是还是不建议在多线程下使用 `HashMap` ，因为多线程下使用 `HashMap` 还是**会存在数据覆盖的问题**。**并发环境下，推荐使用** `ConcurrentHashMap` 。 

 如果想要详细了解 `HashMap` 扩容导致死循环问题，可以看看耗子叔的这篇文章：[Java HashMap 的死循环](https://coolshell.cn/articles/9606.html) 

##  HashMap 线程不安全的原因

 JDK1.7 及之前版本，在多线程环境下，`HashMap` 扩容时会造成**死循环和数据丢失**的问题

> 原因

1. HashMap 属于非同步操作，多线程下会出现并发修改异常
2. HashMap 扩容操作非原子的，多线程下扩容时会造成**死循环和数据丢失**的问题

**非同步操作：**

- `HashMap` 的操作不是线程安全的，例如 `put`、`get`、`remove` 等方法没有进行同步处理。
- 当多个线程同时修改 `HashMap` 的状态时，可能导致竞态条件，从而破坏 `HashMap` 的一致性。

**并发修改异常：**

- 多个线程同时对 `HashMap` 进行修改操作，可能导致 `HashMap` 的内部结构发生变化，但没有足够的同步机制来确保线程安全。
- 在多线程环境下，可能会抛出 `ConcurrentModificationException` 异常

**扩容操作不是原子的：**

- `HashMap` 在容量达到一定阈值时会触发扩容操作，这个操作不是原子的。
- 如果有两个线程同时判断需要扩容，它们可能同时触发扩容操作，导致其中一个线程的操作被覆盖，从而导致数据结构异常。

**链表的环形问题：**

- 在扩容操作期间，由于链表的头尾指针变化，可能导致链表形成环形结构。
- 这样的环形结构可能会导致链表遍历进入死循环，从而破坏了 `HashMap` 的数据结构。



> 数据丢失这个在 JDK1.7 和 JDK 1.8 中都存在，这里以 JDK 1.8 为例进行介绍。 

 JDK 1.8 后，在 `HashMap` 中，多个键值对可能会被分配到同一个桶（bucket），并以链表或红黑树的形式存储。多个线程  对 `HashMap` 的 `put` 操作会导致线程不安全，具体来说会有数据覆盖的风险。 

> **栗子1**

- 两个线程 1,2 同时进行 put 操作，并且发生了哈希冲突（hash 函数计算出的插入下标是相同的）。

- 不同的线程可能在不同的时间片获得 CPU 执行的机会，当前线程 1 执行完哈希冲突判断后，由于时间片耗尽挂起。线程 2 先完成了插入操作。
- 随后，线程 1 获得时间片，由于之前已经进行过 hash 碰撞的判断，所有此时会直接进行插入，这就导致线程 2 插入的数据被线程 1 覆盖了。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    // ...
    // 判断是否出现 hash 碰撞
    // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 桶中已经存在元素（处理hash冲突）
    else {
    // ...
}
```

> **栗子2**

 还有一种情况是这两个线程同时 `put` 操作导致 `size` 的值不正确，进而导致数据覆盖的问题： 

1. 线程 1 执行 `if(++size > threshold)` 判断时，假设获得 `size` 的值为 10，由于时间片耗尽挂起。
2. 线程 2 也执行 `if(++size > threshold)` 判断，获得 `size` 的值也为 10，并将元素插入到该桶位中，并将 `size` 的值更新为 11。
3. 随后，线程 1 获得时间片，它也将元素放入桶位中，并将 size 的值更新为 11。
4. 线程 1、2 都执行了一次 `put` 操作，但是 `size` 的值只增加了 1，也就导致实际上只有一个元素被添加到了 `HashMap` 中。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    // ...
    // 实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
}
```

## HashMap 常见遍历方式

> HashMap **遍历从大的方向来说，可分为以下 4 类**： 

1. 迭代器（Iterator）方式遍历；
2. For Each 方式遍历；
3. Lambda 表达式遍历（JDK 1.8+）;
4. Streams API 遍历（JDK 1.8+）。

>  具体的遍历方式又可以分为以下 7 种： 

1. 使用迭代器（Iterator）EntrySet 的方式进行遍历；
2. 使用迭代器（Iterator）KeySet 的方式进行遍历；
3. 使用 For Each EntrySet 的方式进行遍历；
4. 使用 For Each KeySet 的方式进行遍历；
5. 使用 Lambda 表达式的方式进行遍历；
6. 使用 Streams API 单线程的方式进行遍历；
7. 使用 Streams API 多线程的方式进行遍历。

> 使用迭代器（Iterator）EntrySet 的方式进行遍历；

```java
public class HashMapTest {
    public static void main(String[] args) {
        // 创建并赋值 HashMap
        Map<Integer, String> map = new HashMap();
        map.put(1, "Java");
        map.put(2, "JDK");
        map.put(3, "Spring Framework");
        map.put(4, "MyBatis framework");
        map.put(5, "Java中文社群");
        
        // 遍历
        Iterator<Map.Entry<Integer, String>> iterator = map.entrySet().iterator();
        while (iterator.hasNext()) {
            Map.Entry<Integer, String> entry = iterator.next();
            System.out.println(entry.getKey());
            System.out.println(entry.getValue());
        }
    }
}
```

> 使用迭代器（Iterator）KeySet 的方式进行遍历；

```java
public class HashMapTest {
    public static void main(String[] args) {
        // 创建并赋值 HashMap
        Map<Integer, String> map = new HashMap();
        map.put(1, "Java");
        map.put(2, "JDK");
        map.put(3, "Spring Framework");
        map.put(4, "MyBatis framework");
        map.put(5, "Java中文社群");
        
        // 遍历
        Iterator<Integer> iterator = map.keySet().iterator();
        while (iterator.hasNext()) {
            Integer key = iterator.next();
            System.out.println(key);
            System.out.println(map.get(key));
        }
    }
}
```

> 使用 For Each EntrySet 的方式进行遍历；

```java
public class HashMapTest {
    public static void main(String[] args) {
        // 创建并赋值 HashMap
        Map<Integer, String> map = new HashMap();
        map.put(1, "Java");
        map.put(2, "JDK");
        map.put(3, "Spring Framework");
        map.put(4, "MyBatis framework");
        map.put(5, "Java中文社群");
        
        // 遍历
        for (Map.Entry<Integer, String> entry : map.entrySet()) {
            System.out.println(entry.getKey());
            System.out.println(entry.getValue());
        }
    }
}

```

> 使用 For Each KeySet 的方式进行遍历；

```java
public class HashMapTest {
    public static void main(String[] args) {
        // 创建并赋值 HashMap
        Map<Integer, String> map = new HashMap();
        map.put(1, "Java");
        map.put(2, "JDK");
        map.put(3, "Spring Framework");
        map.put(4, "MyBatis framework");
        map.put(5, "Java中文社群");
        
        // 遍历
        for (Integer key : map.keySet()) {
            System.out.println(key);
            System.out.println(map.get(key));
        }
    }
}

```

> 使用 Lambda 表达式的方式进行遍历；

```java
public class HashMapTest {
    public static void main(String[] args) {
        // 创建并赋值 HashMap
        Map<Integer, String> map = new HashMap();
        map.put(1, "Java");
        map.put(2, "JDK");
        map.put(3, "Spring Framework");
        map.put(4, "MyBatis framework");
        map.put(5, "Java中文社群");
        
        // 遍历
        map.forEach((key, value) -> {
            System.out.println(key);
            System.out.println(value);
        });
    }

```

> 使用 Streams API 单线程的方式进行遍历；

```java
public class HashMapTest {
    public static void main(String[] args) {
        // 创建并赋值 HashMap
        Map<Integer, String> map = new HashMap();
        map.put(1, "Java");
        map.put(2, "JDK");
        map.put(3, "Spring Framework");
        map.put(4, "MyBatis framework");
        map.put(5, "Java中文社群");
        
        // 遍历
        map.entrySet().stream().forEach((entry) -> {
            System.out.println(entry.getKey());
            System.out.println(entry.getValue());
        });
    }
}

```

>  使用 Streams API 多线程的方式进行遍历；

```java
public class HashMapTest {
    public static void main(String[] args) {
        // 创建并赋值 HashMap
        Map<Integer, String> map = new HashMap();
        map.put(1, "Java");
        map.put(2, "JDK");
        map.put(3, "Spring Framework");
        map.put(4, "MyBatis framework");
        map.put(5, "Java中文社群");
        
        // 遍历
        map.entrySet().parallelStream().forEach((entry) -> {
            System.out.println(entry.getKey());
            System.out.println(entry.getValue());
        });
    }
}

```

 [HashMap 的 7 种遍历方式与性能分析！](https://mp.weixin.qq.com/s/zQBN3UvJDhRTKP6SzcZFKw) 

 这篇文章对于 parallelStream 遍历方式的性能分析有误，先说结论：**存在阻塞时 parallelStream 性能最高, 非阻塞时 parallelStream 性能最低** 。 

 当遍历不存在阻塞时, parallelStream 的性能是最低的： 

```text
Benchmark               Mode  Cnt     Score      Error  Units
Test.entrySet           avgt    5   288.651 ±   10.536  ns/op
Test.keySet             avgt    5   584.594 ±   21.431  ns/op
Test.lambda             avgt    5   221.791 ±   10.198  ns/op
Test.parallelStream     avgt    5  6919.163 ± 1116.139  ns/op

```

 加入阻塞代码`Thread.sleep(10)`后, parallelStream 的性能才是最高的: 

```text
Benchmark               Mode  Cnt           Score          Error  Units
Test.entrySet           avgt    5  1554828440.000 ± 23657748.653  ns/op
Test.keySet             avgt    5  1550612500.000 ±  6474562.858  ns/op
Test.lambda             avgt    5  1551065180.000 ± 19164407.426  ns/op
Test.parallelStream     avgt    5   186345456.667 ±  3210435.590  ns/op

```

## ConcurrentHashMap 和 Hashtable 区别

> **底层数据结构** 

**ConcurrentHashMap**

JDK1.7 的 `ConcurrentHashMap` 底层采用 **分段的数组+链表** 实现

- `ConcurrentHashMap` 是由 `Segment` 数组结构和 `HashEntry` 数组结构组成。

- `Segment` 数组中的每个元素包含一个 `HashEntry` 数组，每个 `HashEntry` 数组属于链表结构

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E9%9B%86%E5%90%88-08.png)

JDK1.8 `ConcurrentHashMap`  采用的数据结构跟 `HashMap 1.8` 的结构一样，**数组+链表/红黑二叉树** 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E9%9B%86%E5%90%88-09.png)

- JDK1.8 的 `ConcurrentHashMap` 不再是 **Segment 数组 + HashEntry 数组 + 链表**，而是 **Node 数组 + 链表 / 红黑树**

- 不过，Node 只能用于链表的情况，红黑树的情况需要使用   **`TreeNode`**。当冲突链表达到一定长度时，链表会转换成红黑树。 

- `TreeNode`是存储红黑树节点，被`TreeBin`包装。`TreeBin`通过`root`属性维护红黑树的根结点，  因为红黑树在旋转的时候，根结点可能会被它原来的子节点替换掉，在这个时间点，如果有其他线程要写这棵红黑树就会发生线程不安全问题，所以在 `ConcurrentHashMap` 中  `TreeBin`通过`waiter`属性维护当前使用这棵红黑树的线程，来防止其他线程的进入。 

  ```java
  static final class TreeBin<K,V> extends Node<K,V> {
          TreeNode<K,V> root;
          volatile TreeNode<K,V> first;
          volatile Thread waiter;
          volatile int lockState;
          // values for lockState
          static final int WRITER = 1; // set while holding write lock
          static final int WAITER = 2; // set when waiting for write lock
          static final int READER = 4; // increment value for setting read lock
  ...
  }
  
  ```

**Hashtable**

Hashtable 底层就是 HashMap

*  `Hashtable` 和 JDK1.8 之前的 `HashMap` 的底层数据结构类似都是采用 **数组+链表** 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的； 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E9%9B%86%E5%90%88-06.png)



> **同步机制** 

**ConcurrentHashMap**

*   JDK1.7 时，`ConcurrentHashMap `使用分段锁（Segment Level Locking）的机制。它将整个数据集分成多个小的段（segments），每个段独立加锁。当一个线程对某个段进行修改时，只有该段被锁定，其他的段不受影响，可以并发进行读取操作。  这种分段锁的机制在读多写少的情况下性能较好  
*   到  JDK1.8 时，`ConcurrentHashMap` 已经摒弃 `Segment` 的概念 ，而是直接用 `Node` 数组+链表+红黑树的数据结构来实现，并发控制使用 `synchronized` 和 CAS 来操作。 

**Hashtable**

*  `Hashtable`（同一把锁）：使用 `synchronized` 来保证线程安全， 效率非常低下。
*  在对 `Hashtable` 进行修改操作时，会锁住整个数据结构，其他线程必须等待。这意味着在并发写入的情况下性能可能会受到较大的影响。 

> **扩容时读写操作**

 **ConcurrentHashMap** 

-  `ConcurrentHashMap` 在进行扩容时，只需要锁住当前正在进行扩容的段，而其他的段仍然可以被并发读取和修改。 
-  扩容过程对其他线程的影响较小，相对较好地保持了并发性。 

 **Hashtable** 

- `Hashtable` 在扩容时需要锁住整个表，这可能导致在扩容期间的短时间内其他线程无法执行读取或写入操作，性能相对较差。 

> **性能**：

- 在并发读取的场景中，`ConcurrentHashMap` 的性能通常优于 `Hashtable`。由于分段锁机制，读操作可以并发进行。 
- 在高并发写入的场景中，`Hashtable` 的性能可能会受到锁的影响，相对较低。 

> 【扩展】**null 键和值支持**

- `ConcurrentHashMap` 不允许 `null` 键和值。 
- `Hashtable` 不允许 `null` 键和值。如果尝试插入 `null` 键或值，会抛出 `NullPointerException`。 

## ConcurrentHashMap 线程安全的底层原理

### JDK 1.8 前的原理

使用分段锁（Segment Level Locking）的机制。它将整个数据集分成多个小的段（ `Segment`），每个段独立加锁。当一个线程对某个段进行修改时，只有该段被锁定，其他的段不受影响，可以并发进行读取操作。 

-  `ConcurrentHashMap` 是由 `Segment` 数组结构和 `HashEntry` 数组结构组成 
-  `Segment` 继承了 `ReentrantLock`，所以 `Segment` 是一种可重入锁。
-  `HashEntry` 用于存储键值对数据。 

```java
static class Segment<K,V> extends ReentrantLock implements Serializable {
}

```

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E9%9B%86%E5%90%88-08.png)



 一个 `ConcurrentHashMap` 里包含一个 `Segment` 数组，`Segment` 的个数一旦**初始化就不能改变**  。 `Segment` 数组的大小默认是 16，也就是说默认可以同时支持 16 个线程并发写。 

-  `Segment` 的结构和 `HashMap` 类似，是一种数组和链表结构
-  每个 `Segment` 包含一个 `HashEntry` 数组，每个 `HashEntry` 是一个链表结构的元素
-  每个 `Segment` 守护着一个 `HashEntry` 数组里的元素，当对 `HashEntry` 数组的数据进行修改时，必须首先获得对应的 `Segment` 的锁  。也就是说，对同一 `Segment` 的并发写入会被阻塞，不同 `Segment` 的写入是可以并发执行的。 

### JDK1.8 后的原理

 `ConcurrentHashMap` 取消了 `Segment` 分段锁，采用 `Node + CAS + synchronized` 来保证并发安全。  

- 数据结构跟 `HashMap` 1.8 的结构类似，数组+链表/红黑二叉树 
- Java 8 在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为 $O(n)$）转换为红黑树（寻址时间复杂度为 $O(logn)$
- Java 8 中，锁粒度更细，`synchronized` 只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发，就不会影响其他 Node 的读写，效率大幅提升 

![](https://cyan-images.oss-cn-shanghai.aliyuncs.com/images/Java-Java%E9%9B%86%E5%90%88-09.png)



## JDK 1.7 和 JDK 1.8 的 ConcurrentHashMap 实现的不同

>  **线程安全实现方式** 

*  JDK 1.7 采用 `Segment` 分段锁来保证安全， `Segment` 是继承自 `ReentrantLock` 
*  JDK 1.8 放弃  `Segment` 分段锁的设计，采用 `Node + CAS + synchronized` 保证线程安全，锁粒度更细，`synchronized` 只锁定当前链表或红黑二叉树的首节点。 

> **Hash 碰撞解决方法** 

*  JDK 1.7 采用拉链法

*  JDK 1.8 采用拉链法结合红黑树（链表长度超过一定阈值时，将链表转换为红黑树） 

> **并发度**

*  JDK 1.7 最大并发度是 Segment 的个数，默认是 16 
*  JDK 1.8 最大并发度是 Node 数组的大小，并发度更大 

> 具体优化

### JDK 1.7 中的 `ConcurrentHashMap` 

 **分段锁：** 

-  JDK 1.7 中的 `ConcurrentHashMap` 使用分段锁的机制。底层数据结构被分为一定数量的段（segments），每个段独立加锁。 
-  这种机制在一定程度上提高了并发性，但在高并发写入的情况下，仍可能存在性能瓶颈 

 **链表：** 

-  每个段内部使用链表来解决哈希冲突。链表的结构导致在高并发写入时可能存在竞争，从而影响性能。 

### JDK 1.8 中的 `ConcurrentHashMap`

 **CAS 和 Synchronized 机制：** 

-  JDK 1.8 中引入 CAS（Compare And Swap）操作和更细粒度的锁机制，取代分段锁。 
-  通过使用 CAS 操作，降低了锁的竞争，提高了并发性能。 
-  在一些关键操作上使用了 synchronized 机制，以确保线程安全性。 

 **红黑树：** 

-  JDK 1.8 中的 `ConcurrentHashMap` 引入红黑树的概念，用于解决链表过长的问题。当链表长度达到一定阈值时，将链表转化为红黑树，提高查找性能。 
-  这个改进使得 `ConcurrentHashMap` 在处理大量数据时更具优势。 

 **优化的结构：** 

- JDK 1.8 中对 `Node` 结构进行了优化，减少了内存占用。
- 优化了哈希冲突的处理机制，通过使用 CAS 操作而不是锁来提高并发性。 

 **并发度的调整：** 

-  JDK 1.8 中允许通过构造函数参数指定并发度，即期望的段数。
-  默认情况下，`ConcurrentHashMap` 会自动调整并发度以适应并发写入的需求。  

## ConcurrentHashMap 不允许键和值为 null

**规避歧义：**

- 不允许键和值为 `null` 可以减少对于特殊情况的歧义。`null` 值通常被认为是一个特殊的标志，可能导致一些错误的假设和判断。

- null 是一个特殊的值，表示没有对象或没有引用。如果你用 null 作为键，那么你就无法区分这个键是否存在于 `ConcurrentHashMap` 中，还是根本没有这个键。
- 同样，如果你用 null 作为值，那么你就无法区分这个值是否是真正存储在 `ConcurrentHashMap` 中的，还是因为找不到对应的键而返回的。 

> 拿 get 方法取值来说，返回的结果为 null 存在两种情况：

- 值没有在集合中 ；
- 值本身就是 null。

> 多线程环境下，存在一个线程操作该 `ConcurrentHashMap` 时，其他的线程将该  `ConcurrentHashMap` 修改的情况，所以无法通过 `containsKey(key)` 来判断否存在这个键值对，也就没办法解决二义性问题 

**`HashMap`  对比**

-  `HashMap` 可以存储 `null` 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个。
-  如果传入 `null` 作为参数，就会返回 hash 值为 0 的位置的值。
-  单线程环境下，不存在一个线程操作该 HashMap 时，  其他的线程将该 `HashMap` 修改的情况，所以可以通过  `contains(key)`来做判断是否存在这个键值对，从而做相应的处理，也就不存在二义性问题。 

 **也就是说，多线程下无法正确判定键值对是否存在（存在其他线程修改的情况），单线程是可以的（不存在其他线程修改的情况）** 

> 如果你确实需要在 ConcurrentHashMap 中使用 null 的话，可以使用一个特殊的静态空对象来代替 null。 

```java
public static final Object NULL = new Object();

```

## ConcurrentHashMap 是否能保证复合操作的原子性

-  `ConcurrentHashMap` 是线程安全的，它可以保证多个线程同时对它进行读写操作时，不会出现数据不一致的情况，也不会导致  JDK1.7 及之前版本的 `HashMap` 多线程操作导致死循环问题。 

-  **但是，这并不意味着它可以保证所有的复合操作都是原子性的** 

> 复合操作是指由多个基本操作(如`put`、`get`、`remove`、`containsKey`等)组成的操作
>
> 例如先判断某个键是否存在`containsKey(key)`，  然后根据结果进行插入或更新`put(key, value)`。这种操作在执行过程中可能会被其他线程打断，导致结果不符合预期。 

 有两个线程 A 和 B 同时对 `ConcurrentHashMap` 进行复合操作，如下： 

```java
// 线程 A
if (!map.containsKey(key)) {
map.put(key, value);
}
// 线程 B
if (!map.containsKey(key)) {
map.put(key, anotherValue);
}

```

如果线程 A 和 B 的执行顺序是这样：

1. 线程 A 判断 map 中不存在 key
2. 线程 B 判断 map 中不存在 key
3. 线程 B 将 (key, anotherValue) 插入 map
4. 线程 A 将 (key, value) 插入 map

那么最终的结果是 (key, value)，而不是预期的 (key, anotherValue)。这就是复合操作的非原子性导致的问题。

>  **那如何保证 `ConcurrentHashMap` 复合操作的原子性呢？** 

 `ConcurrentHashMap` 提供了一些原子性的复合操作，如 `putIfAbsent`、`compute`、`computeIfAbsent` 、`computeIfPresent`、`merge`等。这些方法都可以接受一个函数作为参数，根据给定的 key 和 value 来计算一个新的 value，并且将其更新到 map 中。 

```java
// 线程 A
map.putIfAbsent(key, value);
// 线程 B
map.putIfAbsent(key, anotherValue);

```

 或者： 

```java
// 线程 A
map.computeIfAbsent(key, k -> value);
// 线程 B
map.computeIfAbsent(key, k -> anotherValue);

```

#  Collections 工具类

## 排序操作

```java
void reverse(List list) // 反转
void shuffle(List list) // 随机排序
void sort(List list) // 按自然排序的升序排序
void sort(List list, Comparator c) // 定制排序，由Comparator控制排序逻辑
void swap(List list, int i , int j) // 交换两个索引位置的元素
void rotate(List list, int distance) // 旋转。当distance为正数时，将list后distance个元素整体移到前面。当distance为负数时，将 list的前distance个元素整体移到后面

```

## 查找 & 替换操作

```java
int binarySearch(List list, Object key) // 对List进行二分查找，返回索引，注意List必须是有序的
int max(Collection coll)//根据元素的自然顺序，返回最大的元素。 类比int min(Collection coll)
int max(Collection coll, Comparator c)//根据定制排序，返回最大元素，排序规则由Comparatator类控制。类比int min(Collection coll, Comparator c)
void fill(List list, Object obj)// 用指定的元素代替指定list中的所有元素
int frequency(Collection c, Object o) // 统计元素出现次数
int indexOfSubList(List list, List target) // 统计target在list中第一次出现的索引，找不到则返回-1，类比int lastIndexOfSubList(List source, list target)
boolean replaceAll(List list, Object oldVal, Object newVal) // 用新元素替换旧元素
```

## 同步控制

 `Collections` 提供了多个`synchronizedXxx()`方法·，该方法可以将指定集合包装成线程同步的集合，从而解决多线程并发访问集合时的线程安全问题。 

 我们知道 `HashSet`，`TreeSet`，`ArrayList`,`LinkedList`,`HashMap`,`TreeMap`   都是线程不安全的。`Collections` 提供了多个静态方法可以把他们包装成线程同步的集合。 

 **最好不要用下面这些方法，效率非常低，需要线程安全的集合类型时请考虑使用 JUC 包下的并发集合** 

```java
synchronizedCollection(Collection<T>  c) // 返回指定 collection 支持的同步（线程安全的）collection。
synchronizedList(List<T> list) // 返回指定列表支持的同步（线程安全的）List。
synchronizedMap(Map<K,V> m) // 返回由指定映射支持的同步（线程安全的）Map。
synchronizedSet(Set<T> s) // 返回指定 set 支持的同步（线程安全的）set。
```

