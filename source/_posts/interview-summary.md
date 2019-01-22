---
title: Java后端面试题整理
date: 2018-12-18 21:47:55
tags: [interview,Java]
toc: true
---
<!-- ![惠灵顿](/asset/img/interview-summary/Mac.jpeg) -->


这个冬天有点冷，冬天了还是储藏点过冬的粮食，用倒推法来总结相关知识，这篇博客将会长期更新，收集网上的面试题，然后自己整理思路，以后自己肯定会用的上，也为以后博客积累素材。
<!-- more -->
## Java基础
### 语法糖
>1.String类有看过源码吗？和StringBuffer，StringBuilder有什么区别？

String类是一个final不可修改的类,说明String类不能被继承，StringBuffer和StringBuilder都是用来拼接字符串用的，只有调用toString()方法的时候才会真正的new String()对象，StringBuffer是线程安全的，StringBuilder是线程不安全的，因此StringBuilder的性能会更好一点，有并发场景的话还是使用StringBuffer来比较合适。

>2.Throwable、Error、Exception区别和联系。受查异常和非受查异常都用过哪些，谈一下对他们的理解

Throwable 是异常的顶层类，Error和Exception都继承了这个类，当然还有其他一些子类，在我们开发过程中常接触到也就是这两个类的子类，算是两种比较常用的异常处理。Error代表出现了非常严重的错误，JVM虚拟机无法处理，只能Crash，比如OutOfMemoryError。Exception是一般性的异常，这个一般性指的是不会造成虚拟机宕机，但是对开发人员来说，最复杂的也就是这块，对异常的处理是根据业务来的，是抛出去还是try起来，怎么对异常分类？这个都要根据具体的业务来决定，跟业务相关复杂性肯定不会小，因此如何处理好异常也是很有门道的。
Exception和Error本来是互不相关的，但是Exception中有一个叛徒RuntimeException，这个叛徒却是最受开发人员欢迎的，基本上自定义业务一样都会继承它，为什么说他是叛徒呢？它和Error一样，都是都是unchecked的，程序都会进行中止处理，除RuntimException及其子类外的Exception都是checked的，受查异常和非受查异常在在编译过程中就有体现出来，受查异常必须在代码层面做处理，不然编译不通过。常用的非受查异常有OutOfMemoryError,ClassCastException,NullPointerException等，常用的受查异常有：FileNotFoundException，NumberFormatException，NoSuchMethodException，IOException，ClassNotFoundException，InterruptedException等

> 3.父类和子类之间加载的时候，代码块，构造块，构造方法，普通方法调用的顺序

static静态成员变量或者静态代码块是JVM启动的时候加载，所以会优先执行这个，大概的顺序是代码块优先于构造方法，静态优先于非静态。执行的顺序如下：
1.父类静态成员变量和静态代码块
2.子类静态成员变量和静态代码块
3.父类普通成员变量和普通代码块
4.父类构造方法
5.子类普通成员变量和普通代码块
6.子类构造方法

> 4.Java泛型有什么好处？是怎么实现的?

泛型的的好处就是在编译的时候检查类型安全，把运行时异常提前到编译时异常，并且所有的强制转换都是自动和隐式转换，提高了代码的重用，不用到处都是显式的强制转换，让代码更加优雅。
Java泛型的实现原理是类型擦除，是在编译器这个层次来实现。什么是类型擦除呢？就是使用泛型的时候加上的类型参数，在编译的时候去掉，在生成的Java字节码文件中是不包含泛型中的类型信息的。比如，定义了一个`List<String>`类型，在编译之后都会变成`List`类型，JVM看到的只是`List`，而由泛型附加的类型信息是对JVM来说是透明的。

> 5.说说你对面向对象、封装、继承、多态的理解。

- 封装 隐藏具体的实现细节，并且明确标识允许外部使用的成员方法和成员变量，防止数据被破坏。
- 继承 子类继承父类，拥有父类除private修饰的所有成员变量和成员方法，并且可以在父类基础上进行扩展，实现了代码的重用。
- 多态 一个接口有多个子类或者多个实现类，在运行期间(非编译期间)才决定所引用的对象的实际类型，再根据其实际的类型调用其对应的方法，也就是常说的”动态绑定“。有三个条件：继承、重写、向上转型。因此面试的时候往往就是说说多态的理解，这样子面向对象编程的精髓基本都会涉及了。
(1) 继承： 子类继承父类或者实现父类
(2) 重写： 子类重写从父类继承过来的方法
(3) 向上转型： 父类引用指向子类

### 集合

>1.Java的集合类框架介绍一下

最顶层接口是分别是Collection和Map
【Collection】
Collection的实现类有List、Set和Queue。List的实现类有ArrayList和LinkedList等，ArrayList是一个可扩容的对象数组，LinkedList是一个双向链表。Set里的元素是不可重复，常见的有HashSet，TreeSet，LinkedHashSet等，HashSet的实现基于HashMap，实际上就是HashMap中的key。Queue的实现类有LinkedList，可以用作栈，队列和双向队列，另外还有ArrayBlockingQueue等。
【Map】
Map的实现类常见的有HashMap，TreeMap，LinkedHashMap和HashTable等，HashMap使用散列法实现，底层是数组+链表。TreeMap是根据键排好序的Map，使用红黑树实现。LinkedHashMap的实现综合了HashMap和双向链表，可保证以插入时的顺序进行迭代访问。HashTable和HashMap相比，HashTable是线程安全的，HashMap是线程不安全的，HashTable的键或值不允许为null，HashMap允许。

>2.集合容器类中使用了哪些设计模式？

（1）迭代器模式，Collection继承了Iterable接口，其中的Iterator()方法返回Iterator对象，通过这个对象可以迭代Collection中的元素。
（2）适配器模式 `java.util.Arrays#asList()`可以将数组类型转换为`List`数据类型。

>3.ArrayList源码分析

`ArrayList`实现了`RandomAccess`接口，支持随机访问，底层是基于数组来实现，数组默认大小为10。
```java
public class ArrayList<E> extends AbstractList<E> implements List<E>,RandomAccess,Cloneable,java.io.Serializable{
    private static final int DEFAULT_CAPACITY = 10;
    }
```
（1）扩容   添加元素时使用`ensureCapacityInternal()`方法来保证容量足够，如果不够时，需要使用`grow()`方法来进行扩容,新容量的大小为`int newCapacity = oldCapacity + (oldCapacity >> 1);`,也就是1.5倍。扩容操作需要调用`Arrays.copyOf()`方法把原数组整个复制到新数组中，因此最好在创建ArrayList的时候就指定大概的容量大小，减少扩容操作的次数。

```java
public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

 private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }  
```

(2)删除元素 需要调用 System.arraycopy() 将 index+1 后面的元素都复制到 index 位置上，该操作的时间复杂度为 O(N)，可以看出 ArrayList 删除元素的代价是非常高的。
```java
public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```
(3)Fail-Fast机制  modCount 用来记录 ArrayList 结构发生变化的次数。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。在进行序列化或者迭代等操作时，需要比较操作前后 modCount 是否改变，如果改变了需要抛出 ConcurrentModificationException。
```java
private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```
(4)序列化机制 ArrayList 基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。保存元素的数组 elementData 使用 transient 修饰，该关键字声明数组默认不会被序列化。
```java
transient Object[] elementData; // non-private to simplify nested class access
```
ArrayList 实现了 writeObject() 和 readObject() 来控制只序列化数组中有元素填充那部分内容。
```java
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
    
private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```

>4.Vector源码分析

实现大体上与ArrayList类似，但是使用synchronized进行同步
```java
public synchronized boolean add(E var1) {
    ++this.modCount;
    this.ensureCapacityHelper(this.elementCount + 1);
    this.elementData[this.elementCount++] = var1;
    return true;
  }

public synchronized E get(int var1) {
    if (var1 >= this.elementCount) {
      throw new ArrayIndexOutOfBoundsException(var1);
    } else {
      return this.elementData(var1);
    }
  }
  ```
Vector是同步的，因此开销肯定比ArrayList大，访问速度更慢，因此不建议使用，应该使用ArrayList，如果需要到同步，可以使用`Collections.synchronizedList()`得到一个线程安全的ArrayList,或者使用concurrent包下的CopyOnWriteArrayList

```java
List<String> list = new ArrayList<>();
List<String> synList = Collections.synchronizedList(list);
List<String> copyOnWriteList = new CopyOnWriteArrayList<>();
```

>5.CopyOnWriteArrayList源码分析

写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。写操作需要加锁，防止并发写入时导致写入数据丢失。写操作结束之后需要把原始数组指向新的复制数组。

```java
public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }

final Object[] getArray() {
        return array;
    }
    
final void setArray(Object[] a) {
        array = a;
    }
    
public E get(int index) {
        return get(getArray(), index);
    }
```
`CopyOnWriteArrayList`在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。但是`CopyOnWriteArrayList`有其缺陷：
(1)内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；
(2)数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。
所以`CopyOnWriteArrayList`不适合内存敏感以及对实时性要求很高的场景。

>6.LinkedList源码分析

基于双向链表实现，使用Node存储链表节点信息。
```java
private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```
并且每个链表存储了first和last指针：
```java
/**
 * Pointer to first node.
 * Invariant: (first == null && last == null) ||
 *            (first.prev == null && first.item != null)
 */
transient Node<E> first;
    
/**
 * Pointer to last node.
 * Invariant: (first == null && last == null) ||
 *            (last.next == null && last.item != null)
 */
transient Node<E> last;
```
跟ArrayList相比：
(1)ArrayList基于动态数组实现，LinkedList基于双向链表实现
(2)ArrayList支持随机访问，LinkedList不支持
(3)LinkedList在任意位置添加删除元素更快

>7.HashMap源码分析(JDK1.7)

(1)存储结构
包含一个Entry类型的数组table,Entry存储着键值对，包含了四个字段，从netx字段可以看出Entry是一个链表。即数组中的每个位置被当成一个桶，一个桶存放一个链表。HashMap使用拉链法来解决冲突，同一个链表中存放哈希值相同的Entry。
```java
transient Entry[] table;

static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    int hash;

    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }

    public final K getKey() {
        return key;
    }

    public final V getValue() {
        return value;
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry e = (Map.Entry)o;
        Object k1 = getKey();
        Object k2 = e.getKey();
        if (k1 == k2 || (k1 != null && k1.equals(k2))) {
            Object v1 = getValue();
            Object v2 = e.getValue();
            if (v1 == v2 || (v1 != null && v1.equals(v2)))
                return true;
        }
        return false;
    }

    public final int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    public final String toString() {
        return getKey() + "=" + getValue();
    }
}

```

(2)拉链法原理
```java
Map<String,String> map = new HashMap<>();
map.put("k1","v1");
map.put("k2","v2");
map.put("k3","v3");
```
新建一个HashMap,默认大小为16，插入<k1,v1>键值对，假设k1的hashcode为115，则115%16=3，则在table[3]插入<k1,v1>这个Entry，假设k2和k3的hashcode都是118，118%16=6，那么<k2,v2>和<k3,v3>都是插入table[6]这个桶内，<k3,v3>在<k2,v2>前面，即<k3,v3>的next元素是<k2,v2>,每次插入元素都是直接插入链表的表头。

(3)get操作
查找需要分为两步：
①计算键值对所在的桶
②在链表上顺序查找，时间复杂度显然和链表的长度成正比。

如何确定键值对的桶下标呢？
```java
int hash = hash(key);
int i = indexFor(hash, table.length);

final int hash(Object k) {
    int h = hashSeed;
    if (0 != h && k instanceof String) {
        return sun.misc.Hashing.stringHash32((String) k);
    }

    h ^= k.hashCode();

    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}

public final int hashCode() {
    return Objects.hashCode(key) ^ Objects.hashCode(value);
}

static int indexFor(int h, int length) {
    return h & (length-1);
}
```
其实就是两步，先计算hash值，然后对table长度取模。计算hash值这个方法很常规，关键是取模indexFor()这个方法很有意思，正常的写法应该是`h%length`,但是jdk源码里边写的是`h&(length-1)`,第一次看到这个的时候我就愣了，jdk源码应该不会有这么大的bug！这两相等？研究了一下之后发现都是写代码的，人与人之间的差别不是一丁半点！！！

假设x=1<<4,即x为2的4次方，y的值为10110010
正常人取模：y%x
y:10110010
x:00010000
y%x:00000010
大神取模：y&(x-1)
y:10110010
x-1:00001111
y&(x-1):00000010

位运算的代价比求模运算小的多，因此带来更高的性能。但是这个有一个前提就是x必须是2的n次方，这样子x-1就会是一堆的1，和y做与运算之后就把y的高位干掉，只剩下低位，达到取模效果，这个不得不佩服！

(4)put操作
```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    // 键为 null 单独处理
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    // 确定桶下标
    int i = indexFor(hash, table.length);
    // 先找出是否已经存在键为 key 的键值对，如果存在的话就更新这个键值对的值为 value
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    // 插入新键值对
    addEntry(hash, key, value, i);
    return null;
}

private V putForNullKey(V value) {
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    addEntry(0, null, value, 0);
    return null;
}

void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }

    createEntry(hash, key, value, bucketIndex);
}

void createEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    // 头插法，链表头部指向新的键值对
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
```
HashMapy允许插入键为null的键值对，但是无法调用null的hashCode()方法，也就无法确定该键值对的桶下标，只能通过强制指定一个桶下标来存放。HashMap使用第0个桶存放键为null的键值对。
对于键不是null的键值对，使用链表的头插法，也就是新的键值对插在链表的头部，而不是链表的尾部。

(5) 扩容原理
假设HashMap的table长度为M，需要存储的键值对数量为N，如果哈希函数满足均匀性的要求，那么每条链表的长度为N/M,因此平均查找次数的复杂度为O(N/M)。
为了让查找的成本降低，应该尽可能使得N/M尽可能小，因此需要M足够大，HashMap采用动态扩容来根据当前的N值来调整M值，使得空间效率和时间效率都得到保证。

```java
/** 初始容量 */
static final int DEFAULT_INITIAL_CAPACITY = 16;

/** 最大容量 */
static final int MAXIMUM_CAPACITY = 1 << 30;

/** 默认负载因子 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/** Entry数组 */
transient Entry[] table;

/** 键值对数量 */
transient int size;

/** 阈值 */
int threshold;

/** 负载因子 */
final float loadFactor;

/** 修改次数 */
transient int modCount;
```
当需要size大于threshold时，需要进行扩容操作，capacity为原来的两倍，扩容操作需要把oldTable中的所有键值对重新插入newTable中，这个步骤是非常耗时的。因此，如果知道table大小的话最好通过构造函数传入，避免后续扩容造成性能问题。
```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    if (size++ >= threshold)
        resize(2 * table.length);
}

void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }
    Entry[] newTable = new Entry[newCapacity];
    transfer(newTable);
    table = newTable;
    threshold = (int)(newCapacity * loadFactor);
}

void transfer(Entry[] newTable) {
    Entry[] src = table;
    int newCapacity = newTable.length;
    for (int j = 0; j < src.length; j++) {
        Entry<K,V> e = src[j];
        if (e != null) {
            src[j] = null;
            do {
                Entry<K,V> next = e.next;
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            } while (e != null);
        }
    }
}
```
那么问题来了，扩容之后把原来table的entry复制到新的table中，并且table长度变了，那原来的hash值又要重新计算一遍？是的！HashMap使用了一个特殊的机制，可以降低重新计算桶下标的操作，假设原来的table长度为16(00010000)，扩容之后的table长度为32(00100000)，对于一个key，如果它的hash值第五位上为0，那么取模结果和以前一样，如果为1，那么得到的结果为原来的加上16。

(6)计算table容量
```java
/**
  * Returns a power of two size for the given target capacity.
  */
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

> 8.ConcurrentHashMap源码分析(JDK1.7)

`ConcurrentHashMap`和`HashMap`实现上类似，最主要区别是`ConcurrentHashMap`采用了分段锁(Segment),每个分段锁维护着几个同(HashEntry),多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高，并发度就是Segment的个数。
```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {

    private static final long serialVersionUID = 2249069246763182397L;

    static final int MAX_SCAN_RETRIES =
        Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;

    transient volatile HashEntry<K,V>[] table;

    transient int count;

    transient int modCount;

    transient int threshold;

    final float loadFactor;
}

static final class HashEntry<K,V> {
    final int hash;
    final K key;
    volatile V value;
    volatile HashEntry<K,V> next;
}

/** 默认的并发级别为 16，也就是说默认创建 16 个 Segment */
static final int DEFAULT_CONCURRENCY_LEVEL = 16;
```
每个Segment维护一个count变量来统计该Segment中的键值对个数，在执行size()方法时，需要遍历所有的Segment然后把count累计起来。ConcurrentHashMap在执行size()时，先尝试不加锁获取，如果连续两次不加锁获取的结果一致，那么可以认为这个结果是正确的。尝试次数超过3次，就需要对每个Segment进行加锁。
```java
/**
 * Number of unsynchronized retries in size and containsValue
 * methods before resorting to locking. This is used to avoid
 * unbounded retries if tables undergo continuous modification
 * which would make it impossible to obtain an accurate result.
 */
static final int RETRIES_BEFORE_LOCK = 2;

public int size() {
    // Try a few times to get accurate count. On failure due to
    // continuous async changes in table, resort to locking.
    final Segment<K,V>[] segments = this.segments;
    int size;
    boolean overflow; // true if size overflows 32 bits
    long sum;         // sum of modCounts
    long last = 0L;   // previous sum
    int retries = -1; // first iteration isn't retry
    try {
        for (;;) {
            // 超过尝试次数，则对每个 Segment 加锁
            if (retries++ == RETRIES_BEFORE_LOCK) {
                for (int j = 0; j < segments.length; ++j)
                    ensureSegment(j).lock(); // force creation
            }
            sum = 0L;
            size = 0;
            overflow = false;
            for (int j = 0; j < segments.length; ++j) {
                Segment<K,V> seg = segmentAt(segments, j);
                if (seg != null) {
                    sum += seg.modCount;
                    int c = seg.count;
                    if (c < 0 || (size += c) < 0)
                        overflow = true;
                }
            }
            // 连续两次得到的结果一致，则认为这个结果是正确的
            if (sum == last)
                break;
            last = sum;
        }
    } finally {
        if (retries > RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j)
                segmentAt(segments, j).unlock();
        }
    }
    return overflow ? Integer.MAX_VALUE : size;
}
```
JDK 1.7 使用分段锁机制来实现并发更新操作，核心类为 Segment，它继承自重入锁 ReentrantLock，并发度与 Segment 数量相等。JDK 1.8 使用了CAS操作来支持更高的并发度，在CAS操作失败时使用内置锁 synchronized。

>9.LinkedHashMap源码分析(JDK1.7)

`LinkedHashMap`继承自HashMap，因此有HashMap的所有特性
```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>{

    }
```
内部维护了一个双向列表，用来维护插入顺序或者LRU顺序。
```java
/**
  * The head (eldest) of the doubly linked list.
  */
transient LinkedHashMap.Entry<K,V> head;

/**
  * The tail (youngest) of the doubly linked list.
  */
transient LinkedHashMap.Entry<K,V> tail;

/** accessOrder 决定了顺序，默认为 false，此时维护的是插入顺序 */
final boolean accessOrder;
```
LinkedHashMap最重要的是以下用于对维护顺序的函数afterNodeAccess()和afterNodeInsertion()，会在put、get等方法中调用。

当一个节点被使用时，调用get方法，如果assessOrder为true，则会将该节点移到链表尾部。也就是说指定为LRU顺序之后，在每次访问一个节点时，会将这个节点移到链表尾部，保证链表尾部是最近访问的节点，那么链表首部就是最近最久未使用的点。
```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

执行put操作之后，当 removeEldestEntry() 方法返回 true 时会移除最晚的节点，也就是链表首部节点 first。evict 只有在构建 Map 的时候才为 false，在这里为 true
```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```
removeEldestEntry() 默认为 false，如果需要让它为 true，需要继承 LinkedHashMap 并且覆盖这个方法的实现，这在实现 LRU 的缓存中特别有用，通过移除最近最久未使用的节点，从而保证缓存空间足够，并且缓存的数据都是热点数据。
```java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```





### 多线程

>1.Java中有几种方式可以创建线程？

- 继承`Thread`并重写`run`方法
- 实现`Runnable`接口并重写`run()`方法，并将作为参数传入`Thread`
- 实现`Callable`接口并重写`call()`方法，`call()`方法有返回值
- 由线程池创建并管理线程

>2.Java线程池怎么实现的？主要核心类讲一下

`Executors`是线程池的工厂类，通过调用它的静态方法来创建线程池，比如`newCachedThreadPool()`方法，这些静态方法统一返回一个`ThreadPoolExecutor`。
```java
public ThreadPoolExecutor(int corePoolSize,
                        int maximumPoolSize,
                        long keepAliveTime,
                        TimeUnit unit,
                        BlockingQueue<Runnable> workQueue,
                        ThreadFactory threadFactory,
                        RejectedExecutionHandler handler) {
    
}
```
参数解释：
- corePoolSize: 指定线程池中线程的数量
- maximumPoolSize: 线程池中最大的线程池数量
- keepAliveTime: 当线程池中的线程数量超过corePoolSize的时候，多余的线程最大的存活时间
- unit: keepAliveTime的单位
- workQueue: 任务队列，被提交但还未被执行的任务
- threadFactory: 线程工厂，用于创建线程
- handler: 拒绝策略，当任务太多来不及处理的时候，采用什么方式拒绝任务

核心参数: workQueue和handler
【workQueue】
有`ArrayBlockingQueue`有界队列，`LinkedBlockingQueue`无界队列，`SynchronousQueue`直接提交队列
`ArrayBlockingQueue`：当线程池中实际线程数小于corePoolSize时，直接创建线程执行任务，当大于corePoolSize时，提交到workQueue中，因为这个队列是有界的，当队列满时，在不大于maximumPoolSize的情况下，创建线程执行任务，如果大于maximumPoolSize，执行拒绝策略handler
`LinkedBlockingQueue`：当线程池中实际线程数小于corePoolSize时，直接创建线程执行任务，当大于corePoolSize小于maximumPoolSize时，提交到workQueue中，因为队列是无界的，所以之后提交的任务都会进入队列中
`SynchronousQueue`：该队列没有容量，对提交的任务不做保存，直接新增线程来执行任务。

【handler】
- 直接抛异常
- 在调用者的线程中执行当前任务
- 丢弃最老的一个请求，将队列头的任务poll出去
- 直接丢弃无法处理的任务，不做任何处理


### IO
### JVM
>1.ClassLoader的种类，父子关系(不一定是继承)，双亲委派机制，Java为什么要用双亲委派机制?

【种类】
ClassLoader包括：引导类加载器(BoostrapClassLoader)、扩展类加载器(ExtensionsClassLoader)、应用类程序加载器(ApplicationClassLoader)、自定义类加载器

- 引导类加载器：加载JVM自身需要的类，由C++实现，是虚拟机的一部分，负责将lib下和核心类库和-Xbootclasspath参数指定路径下的jar包加载到内存中，而且虚拟机加载的jar包的时候是根据jar包文件名来识别的，比如rt.jar,如果文件名不被虚拟机识别，即使把jar包放在lib下边也不会被加载，并且BootstrapClassLoader只加载包名为java、javax、sun等开头的类，这些都是为了虚拟机的安全考虑。
- 扩展类加载器：负责加载JAVA_HOME/lib/ext目录下或者由虚拟机参数指定的`-Djava.ext.dir`指定路径中的类库。
- 应用类程序加载器：ApplicationClassLoader是指sun公司实现的`sun.misc.Launcher$AppClassLoader`,负责加载系统类路径`java -classpath`或者`-Djava.class.path`下的类库，一般情况下，这个是默认的类加载器，通过`ClassLoader#getSystemClassLoader()`方法可以获取。
- 自定义类加载器：通过继承`java.lang.ClassLoader`并重写`findClass()`方法来实现。

【父子关系】
 - 启动类加载器，由C++实现，没有父类
 - 扩展类加载器，由Java实现，父类为Null，其实是启动类加载器，但是启动类加载器不是Java实现实现的，所以获取到的是Null
 - 系统类加载器，由Java实现，父类加载器为扩展类加载器
 - 自定义类加载器，由Java实现，父类加载器为系统类加载器

 【双亲委派机制】
 如果一个类收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行，如果父类加载器还存在父类加载器，则进一步向上委托，请求最终到达顶层的启动类加载器，如果父类加载器可以完成加载任务，则成功返回，倘若父类加载器无法完成此加载任务，子类加载器才会去尝试加载，这就是双亲委派机制。但是这个翻译不应该叫双亲，应该叫好爸爸机制，因为子类都是不管怎样都是先丢给父类来加载，只有父类加载不了子类才会去加载。
 
 【双亲委派机制好处】
 为什么要用双亲委派机制呢？肯定是因为这种机制有某个好处使得官方采用。
 （1）避免类的重复加载。当父类加载器加载了该类时，就没有必要子ClassLoader再加载一次。
 （2）安全因素考虑，Java核心API中定义的类不会被改变，防止核心API库被随意修改。

> 2.如何打破双亲委派机制?什么时候需要打破双亲委派机制？有哪些框架打破了双亲委派机制？

打破双亲委派机制不仅要继承`java.lang.ClassLoader`重写`findClass()`方法，还要重写`loadClass()`方法，默认的`loadClass()`方法是实现了双亲委派机制的逻辑，即会先让父类加载器加载，当无法加载时才由自己加载。因此为了打破双亲委派机制就必须重写`loadClass()`,可以在这个方法里边使用任何一个类加载器来加载，不一定优先由启动类加载器来加载，这样子就打破了双亲委派机制。

什么时候需要打破双亲委派机制呢？双亲委派机制很好的解决了各个类加载器的基础类统一问题，越基础的类越由上层的类加载器加载，基础类之所以叫做基础类，是因为他们总是作为被调方被别人调用，但是，如果基础类想调用户的代码，这个时候怎么办呢？这并非不可能的事，这个时候就需要打破双亲委派机制，一个典型的例子就是JNDI服务，它的代码由引导类启动器去加载，但是JNDI的目的就是对资源进行集中管理和查找，需要调用各个厂商实现部署在应用程序的classpath下的JNDI接口提供者(SPI)的代码，但是引导类加载器不认识这些代码，该如何解决呢？引入线程上下文加载器，这个类加载器可以通过`java.lang.Thread`类的`setContextClassLoader()`方法进行设置，如果创建线程时还未设置，它将会从父线程中继承一个,如果在应用程序的全局范围内都没有设置过，那么这个类加载器默认就是应用程序类加载器。有了线程上下文类加载器，JNDI服务使用这个线程上下文类加载器去加载所需要的SPI代码，也就是父类加载器请求子类加载器去完成类加载动作，这种行为实际上就是打通了双亲委派模型的层次结构来逆向使用类加载器，已经违背了双亲委派模型。还有一个就是OSGi,为了可以动态加载模块，每个模块都有自己类加载器，这些类加载器组成了网状结构，不再是双亲委派模型中的树状结构。

> 3.JVM内存结构

JVM内存结构主要有三大块：堆内存、方法区和栈。
堆内存是JVM中最大的一块由年轻代和老年代组成，而年轻代内存又被分成三部分，Eden空间、From Survivor空间、To Survivor空间,默认情况下年轻代按照8:1:1的比例来分配
方法区存储类信息、常量、静态变量等数据，是线程共享的区域，为与Java堆区分，方法区还有一个别名Non-Heap(非堆)
栈又分为java虚拟机栈和本地方法栈,主要用于方法的执行

> 4.对象是否可GC

这个问题其实就是JVM如何判断对象是否需要被回收，JVM使用可达性算法来计算一个对象是否可达。
算法思路：通过一些被列为”GC Roots“的对象作为起始点，从这些点开始向下搜索，搜索走过的路径称为引用链，当一个对象到GC Roots没有任何引用链时，则说明对象需要被回收。
可以作为GC Roots对象包括以下几种：
- 虚拟机栈中引用的对象
- 方法区中静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中JNI引用的对象

> 5.Minor GC、Major GC和 Full GC

堆内存由年轻代和老年代组成。
从年轻代空间（包括 Eden 和 Survivor 区域）回收内存被称为 Minor GC。
Major GC 是清理老年代。
Full GC 是清理整个堆空间—包括年轻代和老年代。


## 数据库
### MySQL
> 1.MySQL数据库的底层原理

MySQL数据库是C/S架构模式，我们关心的是Server端，也就是mysqld这个进程。MySQL服务端是二层架构：
1.SQL层(SQL Layer),在MySQL数据库系统处理底层数据之前的所有工作都是在这一层完成，包括权限判断，SQL解析，执行计划优化，查询缓存的处理等等。具体模块及职责如下:
- 初始化模块: MySQL Server启动的时候做初始化操作
- 核心API: 提供一些非常高效的底层操作功能的优化实现
- 网络交互模块: 抽象底层网络交互所使用的api，实现底层网络数据的接收与发送
- Client&Server交互协议模块: MySQL 客户端和服务端交互的协议实现，基于TCP/IP,Socket等
- 用户模块: 用户管理模块，包括用户的登录连接权限以及用户的授权管理
- 访问限制模块: 根据用户模块的授权信息来控制用户对数据的访问
- 连接管理模块: 负责监听对MySQL Server的各种请求，接收连接请求，转发所有的连接请求到线程管理模块，每一个连接上MySQL Server的客户端请求都会被分配或者创建一个连接线程为其单独服务
- Query解析和转发模块: 连接线程接收到客户端的一个Query之后，将Query分类后转发给各个对应的处理模块
- Query Cache模块: 将客户端提交给MySQL的Select类Query请求的返回结果集缓存到内存中，与该Query的一个Hash值做一个对应。当取数据的基表发生变化之后，缓存失效
- Query 优化器模块: 优化客户端请求的Query，根据一些算法来得出最优的策略，告诉程序如何去取这个Query语句的结果
- 表变更管理模块: 负责完成DDL、DML的Query，比如update，delete，insert，create table，alter table等语句的处理
- 表维护模块: 表的状态检查，错误修复
- 系统状态管理模块: 在客户端请求系统状态的时候，将各种系统状态返回给用户，比如show status,show variables等
- 表管理器: 维护.frm文件，以及一个Cache,缓存各个表的结构信息。
- 日志记录模块: 负责的整个系统的日志记录
- 复制模块: 分为Master模块和Slave模块，Master模块主要负责在Replication环境下读取Master节点的binary日志，以及和Slave端的IO线程交互。Slave模块有两个线程(IO线程和SQL线程)，IO线程从Master请求和接受binary日志，并写入本地relay log。SQL线程从relay log中读取日志事件，解析成可以执行的SQL语句，然后顺序执行，这样子从节点就和主节点的数据基本上保持一致。
- 存储引擎接口模块: 将各种数据处理高度抽象化，各种存储引擎产品实现接口即可，实现了MySQL特有的可插拔存储引擎。

2.存储引擎层(Storage Engine Layer),底层数据的存取都是在这一层做的，常用的存储引擎有InnoDB引擎和MyISAM引擎。


> 2.InnoDB引擎和MyISAM引擎的区别和应用场景

从这两种存储引擎的优缺点来谈区别以及应用场景
【InnoDB】
优点：支持事务，支持外键
缺点：占用磁盘多，读效率慢于MyISAM
【MyISAM】
优点：查询较快（读取数据不加锁），支持多种存储方式（静态表，压缩表等）
缺点：写入较慢（锁表），没有事务。

结论：支持事务选InnoDB,对读有要求的选MyISAM。

> 3.数据库索引的原理，分类

> 4.B+树有了解吗？为什么MySQL选用B+树做主要存储结构，为什么常用索引推荐使用B+树？为什么B+树可以减少磁盘IO?

> 4.如何避免索引失效？

> 5.有处理过MySQL优化吗？如何优化？

## 框架
### SpringFramework
> 1.简单谈谈Spring的IOC和AOP,都在哪些场景用到过？

IOC是控制反转，就是对象的创建工作交给Spring容器来做，程序员不需要new出来，需要什么就跟Spring容器要，前提是容器里边有这个对象，底层实现是反射。AOP是面向切面编程，在记录日志，权限控制，事务方面都有应用，底层实现是动态代理。

> 2.Spring中Bean生命周期过程，以及两种作用域Singleton和Prototype有什么区别，应用场景有哪些？

生命周期:
1.Bean的建立：有BeanFactory读取Bean定义文件，并生成各个实例

2.Setter注入：执行Bean的属性依赖注入

3.BeanNameAware的setBeanName()：如果Bean实现了org.springframework.beans.factory.BeanNameAware接口，则执行其setBeanName()方法

4.BeanFactoryAware的setBeanFactory()：如果Bean实现了org.springframework.beans.factory.BeanFactoryAware接口，则执行其setBeanFactory()方法

5.BeanPostProcessors的processBeforeInitialization()：容器中如果有实现org.springframework.beans.factory.BeanPostProcessors接口的实例，则任何Bean在初始化之前都会执行这个实例的processBeforeInitializaton()方法

6.InitializingBean的afterPropertiesSet()：如果Bean类实现了org.springframework.beans.factory.InitailizingBean接口，则执行去afterPropertiesSet()方法

7.Bean定义文件中定义init-method：如果在xml文件中定义一个Bean的时候指定了init-methond，则执行这个方法，并且这个初始化方法是不能带有参数。

8.BeanPostProcessors的processAfterInitializaton()：容器中如果有实现org.springframework.beans.factory.BeanPostProcessors接口的实例，则去执行processAfterInitialization()方法

9.DisposableBean的destroy()方法：在容器关闭时，如果Bean实现了org.springframework.beans.factory.DisposableBean接口，则执行它的destroy()方法

10.Bean定义文件中定义destroy-method：在容器关闭时，执行destroy-method()方法，并且这个方法是不带参数的

作用域：
【singleton】：单例，每次访问都是返回同一个实例，Spring默认，无状态的Bean都应该使用Singleton，比如需要回收的重要资源(数据库连接池，线程池)
【prototype】：多例，每次访问都会进行new操作，Spring不会对Bean的整个生命周期进行负责，具有prototype的作用域的Bean创建后交由调用者负责销毁对象回收资源，有状态的Bean应该配置成prototype



> 3.AOP动态代理模式的两种类型,区别是什么？

AOP动态代理有两种：JDK代理和CGLIB代理，JDK代理只能对实现了接口的类进行代理，而不能针对未实现接口的类。CGLIB代理是针对类(未使用final修饰)实现代理,主要是对指定的类生成一个子类，覆盖其中的方法，底层是使用ASM字节码生成框架，使用字节码技术生成代理类。
Spring是怎么选择使用哪种代理的呢？如果一个Bean实现了接口，那么默认使用JDK代理，当Bean没有实现接口时，Spring默认使用CGLIB代理。

> 4.Spring事务的特性，隔离级别，传播行为

特性：
1.原子性(Atomicity)： 事务的不可分割性
2.一致性(Consistency)：事务执行前后数据的完整性保持一致
3.隔离性(Isolation)：事务执行过程中，不应该受到其他事务的干扰
4.持久性(Durability)：事务一旦结束，数据就持久到数据库

如果事务不考虑隔离性，就会引发安全性问题。比如：
1.脏读：一个事务读到了另一个事务未提交的数据
2.不可重复读：一个事务读到了另一个事务已经提交的update数据导致多次查询结果不一致
3.虚幻读：一个事务读到了另一个事务已经提交的insert的数据导致多次查询结果不一致

为了解决这个问题，引出事务隔离级别：
1.默认(default)：默认的隔离级别，使用数据库的默认隔离级别
2.未提交读(read uncommited)：脏读，不可重复读，虚幻读都有可能发生
3.已提交读(read commited)：避免脏读，但是不可重复读和虚幻读还是有可能发生
4.可重复读(repeatable read)：避免脏读和不可重复读，但是虚幻读还是有可能发生
5.串行化(Serializable)：避免上述所有读问题

MySQL默认的隔离级别是可重复读，Oracle默认的隔离级别是已提交读

事务的传播行为：
1.保证同一个事务中
PROPAGATION_REQUIRED：支持当前事务，如果不存在就新建一个
PROPAGATION_SUPPORTS：支持当前事务，如果不存在，就不使用事务
PROPAGATION_MANDATORY：支持当前事务，如果不存在就抛异常
2.保证没有在同一个事务中
PROPAGATION_REQUIRED_NEW：如果有事务存在，挂起当前事务，创建一个新的事务
PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果有事务存在，挂起当前事务
PROPAGATION_NEVER：已非事务方式运行，如果有事务存在，则抛出异常
PROPAGATION_NESTED：如果当前事务存在，则已嵌套事务执行

### SpringMVC
> 1.SpringMVC的作用，与Struct的区别是什么？

SpringMVC是一个基于请求驱动的Web框架，使用了前端控制器模式来设计，再根据请求映射规则分发给相应的页面控制器进行处理。简单点就是处理http请求和响应。
区别：
- SpringMVC是基于Servlet来实现，Struct是基于Filter来实现的。
- SpringMVC是基于方法级别的拦截，一个方法对应一个request上下文，Controller Bean默认是sigleton的，只会创建一个Controller，但是没有共享的属性，所以是线程安全。Struct是基于类级别的拦截，每次请求都会创建一个Action，Action Bean在Spring容器中是prototype的，通过setter方法将request注入到属性当中。

> 2.SpringMVC的工作原理，都涉及到哪些类？

1.用户发送请求至前端控制器DispatherServlet

2.DispatherServlet接收到请求之后调用HandlerMapping处理映射器

3.处理映射器找到具体的处理类(xml配置，注解)，生成处理器对象及处理拦截器一并返回给
DispatherServlet

4.DispatherServlet调用HandlerAdapter处理适配器

5.HandlerAdapter经过适配调用具体的处理器(Controller)

6.Controller执行完成返回ModelAndView对象

7.HandlerAdapter将ModelAndView返回给DispatherServlet

8.DispatherServlet将ModelAndView对象返回给ViewResolver视图解析器

9.ViewResolver解析后返回具体的View

10.DispatherServlet根据View进行渲染视图

11.DispatherServlet响应用户

## 计算机网络

> 1.谈谈TCP/IP模型

这个问题可以从OSI七层模型谈起引出TCP/IP五层模型，OSI是学术定义，但是不实用，工业界使用的是TCP/IP。
OSI七层模型:
- 应用层 针对特定的协议，为应用程序做服务,比如SMP,POP3,SSH,FTP等协议。
- 表示层 负责数据格式的转换，把不同表现形式的信息转换成适合网络传输的格式。
- 会话层 负责建立和断开通信连接，什么时候建立连接，什么时候断开连接以及保持多久的连接。
- 传输层 在两个通信节点之间负责数据的传输，起着可靠传输的作用。(运行在这一层的设备有四层交换机，四层路由器)
- 网络层 路由选择，在多个网络之间转发数据包，负责将数据包传送到目的地址。(运行在这一层的设备有路由器，三层交换机)
- 数据链路层 负责物理层面互联设备之间的通信传输，比如一个以太网相连的两个节点之间的通信，是数据帧与1、0比特流之间的转换。(运行在这一层的设备是网桥、以太网交换机、网卡)
- 物理层 主要是1、0比特流与电子信号的高低电平之间的转换。(运行在这一层的设备是中继器、双绞线)

TCP/IP五层模型:
七层模型中的应用层、表示层、会话层在这个模型中都属于应用层，其他四层不变。

> 2.TCP的三次握手和四次挥手，为什么要这样子设计？

【三次握手】
1.客户端发起连接请求，发送SYN=1和客户端序号c给服务器端，同时进入SYN_SENT状态。
2.服务端收到SYN后需要作出确认，于是发送ACK=1,同时自己也发送SYN=1,服务端序号s，同时发送确认号c+1(表示下一个接受序号),进入SYN_RCVD状态。
3.客户端收到服务端的SYN和ACK,作出确认，发送ACK=1，以及序号c+1的数据，同时发送确认号s+1(表示客户端下一个要接收的序号)。此时客户端和服务端进入ESTABLISHED状态，确认过眼神，状态已经建立。

【四次挥手】
1.客户端发起断开请求，发送FIN=1和序号c给服务端，客户端进入FIN-WAIT-1状态。
2.服务端收到FIN后作出确认，发送ACK=1和服务端序号s,确认号c+1(表示下一个接收序号),服务端此时还可以向客户端发送数据,服务端进入CLOSE-WAIT状态，客户端进入FIN-WAIT-2状态。
3.服务端没有数据发送时，它向客户端发送FIN=1,ACK=1,请求断开连接，同时发送服务端序号s，确认号c+1,服务端进入LAST-ACK状态。
4.客户端收到之后进行确认，发送ACK=1,以及序号c+1和确认号s+1。客户端进入TIME-WAIT状态，客户端需要等待2MSL,确保服务端收到了ACK，若这期间没有服务端的消息，便可认为服务端收到了确认，此时可以断开连接。服务端和客户端进入CLOSED状态。

![TCP三次握手四次挥手](/asset/img/interview-summary/tcp_shake_hand.jpg)

为什么这样子设计呢？握手两次行不行？

两次握手的话，只要服务端发出确认就建立连接了。有一种情况是客户端发出了两次连接请求，但由于某种原因，使得第一次请求被滞留了。第二次请求先到达后建立连接成功，此后第一次请求终于到达，这是一个失效的请求了，服务端以为这是一个新的请求于是同意建立连接，但是此时客户端不搭理服务端，服务端一直处于等待状态，这样就浪费了资源。假设采用三次握手，由于服务端还需要等待客户端的确认，若客户端没有确认，服务端就可以认为客户端没有想要建立连接的意思，于是这次连接不会生效。

四次握手，为什么客户端发送确认后还需要等待2MSL?

因为第四次握手客户端发送ACK确认后，有可能丢包了，导致服务端没有收到，服务端就会再次发送FIN = 1，如果客户端不等待立即CLOSED，客户端就不能对服务端的FIN = 1进行确认。等待的目的就是为了能在服务端再次发送FIN = 1时候能进行确认。如果在2MSL内客户端都没有收到服务端的任何消息，便认为服务端收到了确认。此时可以结束TCP连接。

> 3.有了传输层为什么还要网络层？

网络层是针对主机与主机之间的服务。而传输层针对的是不同主机进程之间的通信。网络层负责将数据包从源IP地址转发到目标IP地址，而传输层负责将数据包再递交给主机中对应端口的进程。

> 4.TCP序号的作用，怎么样保证可靠传输？

序号和确认号是实现可靠传输的关键。
序号-当前数据包的首个字节的顺序号，确认号-表示下一个想要接收的字节序号，并且已经正确收到确认号之前的所有字节。
通信双方通过序号和确认号来判断数据是否丢失，是否按顺序到达，是否冗余等等，如果丢失了就重传，如果冗余了就丢弃，换句话说，序号，确认号和重传机制保证了数据不丢失、不重复。

> 5.TCP和UDP的区别

- TCP面向连接，传输数据之前需要建立会话，UDP是无连接的
- TCP是可靠传输，UDP只负责发送数据，不保证接收方是否接收，不保证可靠
- TCP面向字节流，UDP面向报文
- TCP只支持点到点通信，UDP支持点到点、点到面、面到面的通信


> 6.浏览器的一个HTTP请求到后端的一个大致过程是怎样的？

1.利用DNS进行域名解析(先找本地hosts再找运营商的DNS服务器)
2.发起TCP三次握手
3.建立TCP连接之后发起HTTP请求
4.服务器响应HTTP请求，浏览器得到HTML代码
5.浏览器解析HTML代码，并请求HTML代码中的资源(csc,js,image)
6.浏览器对页面进行渲染

> 7.HTTP是基于TCP还是UDP?

HTTP 1.0/1.1 是基于TCP协议，客户端向服务端发送一个HTTP请求时，需要与服务端建立TCP连接，三次握手成功后才会进行数据交互，目前基本上都是HTTP 1.1的。2.0是基于UDP的。

> 8.HTTP请求和响应的报文结构

HTTP请求报文结构：
- 请求头：包括请求方法、URL、HTTP协议版本号
- 请求头： 多组键值对
- 请求空行：告诉服务器请求头的键值对已经发送完毕
- 请求主体

HTTP响应报文结构：
- 响应行：包括HTTP协议版本号、状态码、状态码描述
- 响应头：多组键值对
- 响应空行：告诉客户端响应头键值对结束
- 响应主体

> 9.HTTP常见的状态码

- 1xx：信息性状态码，表示接收的请求正在处理
- 2xx：成功状态码，表示请求处理完毕
- 3xx：重定向状态码，表示需要进行附加操作以完成请求
- 4xx：客户端错误状态码，表示服务器无法处理请求
- 5xx：服务端错误状态码，表示服务器处理请求出错

常见的状态码有：
- 200 OK,请求被正常处理
- 201 created，对象创建成功
- 301 Move Permanently，永久性重定向
- 302 Found，临时重定向
- 400 Bad Request，请求报文中格式不对
- 403 Forbidden，对请求的资源没有权限访问
- 404 Not Found，在服务器上找不到请求资源
- 405 Method Not Allowed，请求方法不允许
- 500 Internal Server Error，服务器内部错误

> 10.GET和POST区别

- GET用于获取数据，POST用于提交数据
- GET的参数长度有限制，POST没有限制
- GET把参数放在url中，POST通过封装参数到请求体中发送
- GET请求只能进行url编码，POST支持多种编码方式
- GET比POST参数更不安全，因为参数暴露在url上，所以不能用来传递敏感信息
- GET具有幂等性，多次请求得到的结果一样，POST不具备幂等性，多次请求会有重复提交问题
- GET请求会被浏览器保存为历史记录，POST请求不能
- GET请求产生一个TCP数据包，POST产生两个TCP数据包
  对于GET请求方式，浏览器会把header和data一起发送出去，服务器接收请求返回数据
  对于POST请求方式，浏览器先发送header，服务器响应100继续，浏览器在发送data，服务器接收请求返回数据

## 算法


