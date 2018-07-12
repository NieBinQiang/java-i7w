## HashMap总览 
（该篇是基于JDK1.6）

```
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {}
```
- `HashMap` 是一个散列表，它存储的内容是**键值对(key-value)**映射。
- `HashMap` 继承于AbstractMap，实现了`Map`、`Cloneable`、`java.io.Serializable`接口。
- HashMap 是**非线程安全**的。它的**key、value都可以为null**。此外，HashMap中的映射**不是有序的**。

`HashMap` 的实例有两个参数影响其性能：**初始容量** 和 **加载因子**。
- `容量` 是哈希表中存储键值对的数量，初始容量只是哈希表在创建时的容量。默认大小是 16。
- `加载因子` 是哈希表在其容量自动增加之前可以达到多满的一种**尺度**。
- 当哈希表中的条目数**超出了加载因子与当前容量的乘积**时，则要对该哈希表进行 `rehash`操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数。
- 默认加载因子是**0.75**,这是在时间和空间成本上寻求一种**折衷**。加载因子过高虽然减少了空间开销，但同时也**增加了查询成本**（在大多数 HashMap 类的操作中，包括 get 和 put 操作，都反映了这一点）。在设置初始容量时应该考虑到映射中所需的条目数及其加载因子，以便最大限度地减少 rehash 操作次数。

## HashMap数据结构
![image](https://camo.githubusercontent.com/9cc00b7c617f772aa70816727f5b6801e1145fef/68747470733a2f2f7773322e73696e61696d672e636e2f6c617267652f303036744e633739677931666e3834623066746a346a333065623035363073762e6a7067)

`HashMap` 是通过"**拉链法**"实现的哈希表。它包括几个重要的成员变量：table, size, threshold, loadFactor, modCount。
- `table` 是一个Entry[]类型的**单向链表**。哈希表的"key-value键值对"都是存储在Entry数组中的。 
- `size` 是HashMap的实际大小，它是HashMap**当前的键值对的数量**。 
- `threshold` 是HashMap的**阈值**，用于判断是否需要调整HashMap的容量。threshold的值="容量*加载因子"，当HashMap中存储数据的数量达到threshold时，就需要将HashMap的容量加倍--`rehash`。
- `loadFactor`就是加载因子。 
- `modCount` 是用来实现fail-fast机制的。

PS: “key为null”的元素存储在table[0]位置

### 数据节点Entry的数据结构

```
static class Entry<K,V> implements Map.Entry<K,V> {}
```
Entry<K,V> 实现 Map.Entry<K,V>接口，包含key， value ，next ，hash几个重要参数。
- final K key 
- V value
- Entry<K,V> next 指向下一个节点
- final int hash  key对应的hash值

构造函数
```java
// 构造函数。
// 输入参数包括"哈希值(h)", "键(k)", "值(v)", "下一节点(n)"
Entry(int h, K k, V v, Entry<K,V> n) {
    value = v;
    next = n;
    key = k;
    hash = h;
}
```


判断两个Entry是否相等

```java
// 判断两个Entry是否相等
// 若两个Entry的“key”和“value”都相等，则返回true。
// 否则，返回false
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
```
## HashMap源码解析
### HashMap的构造函数

```
// 默认构造函数。
public HashMap() {
    // 设置“加载因子” 默认值0.75
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    // 设置“HashMap阈值”，用于扩容判断
    threshold = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);
    // 创建单向链表数组，用于保存数据
    table = new Entry[DEFAULT_INITIAL_CAPACITY];
    init();
}

// 指定“容量大小”和“加载因子”的构造函数
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    // HashMap的最大容量只能是MAXIMUM_CAPACITY 2的30次方
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);

    // 初始化容量，大于或等于给定容量的最小2次幂
    int capacity = 1;
    while (capacity < initialCapacity)
        capacity <<= 1;

    // 设置“加载因子”
    this.loadFactor = loadFactor;
    // 设置“HashMap阈值”，当HashMap中存储数据的数量达到threshold时，就需要将HashMap的容量加倍。
    threshold = (int)(capacity * loadFactor);
    // 创建Entry数组，用来保存数据
    table = new Entry[capacity];
    init();
}

// 指定“容量大小”的构造函数
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

// 包含“子Map”的构造函数
public HashMap(Map<? extends K, ? extends V> m) {
    this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                  DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
    // 将m中的全部元素逐个添加到HashMap中
    putAllForCreate(m);
}
```
### HashMap的主对外接口
#### clear() 
clear() 清空HashMap，将Entry数组所有元素设为null。

```
public void clear() {
    modCount++;
    Entry[] tab = table;
    for (int i = 0; i < tab.length; i++)
        tab[i] = null;
    size = 0;
}
```
#### containsKey()
containsKey() 的作用是判断HashMap是否包含key。

```JAVA
public boolean containsKey(Object key) {
    return getEntry(key) != null;
}
```
getEntry()的源码
这里需要强调的是：**HashMap将“key为null”的元素都放在table的位置0处，即table[0]**
```JAVA
final Entry<K,V> getEntry(Object key) {
    // 获取哈希值
    // HashMap将“key为null”的元素存储在table[0]位置，“key不为null”的则调用hash()计算哈希值
    int hash = (key == null) ? 0 : hash(key.hashCode());
    // 在“该hash值对应的链表”上查找“键值等于key”的元素
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```
#### containsValue()    
containsValue() 的作用是判断HashMap**是否包含“值为value”的元素。**

```java
public boolean containsValue(Object value) {
    // 若“value为null”，则调用containsNullValue()查找
    if (value == null)
        return containsNullValue();

    // 若“value不为null”，则查找HashMap中是否有值为value的节点。
    Entry[] tab = table;
    for (int i = 0; i < tab.length ; i++)
        for (Entry e = tab[i] ; e != null ; e = e.next)
            if (value.equals(e.value))
                return true;
    return false;
}
```
containsNullValue()的作用判断HashMap中是否包含“值为null”的元素。

```java
private boolean containsNullValue() {
    Entry[] tab = table;
    for (int i = 0; i < tab.length ; i++)
        for (Entry e = tab[i] ; e != null ; e = e.next)
            if (e.value == null)
                return true;
    return false;
}
```
#### get()
get() 的作用是获取key对应的value 

步骤：先求出key对应的Hash值，再对hash值取模（模为table大小），找到对应链表的下标，再遍历链表去查询`key.equals(k)`的节点。
```java
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    // 获取key的hash值
    int hash = hash(key.hashCode());
    // 在“该hash值对应的链表”上查找“键值等于key”的元素
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
            return e.value;
    }
    return null;
}
```

#### put()
- 首先会将传入的 Key 做** hash 运算计算**出 hashcode,然后**根据table长度取模**计算出在数组中的 index 下标。
- 由于在计算中位运算比取模运算效率高的多，所以 HashMap **规定数组的长度为 2^n** 。这样用 2^n - 1 做位运算与取模效果一致，并且效率还要高出许多。
- 由于数组的长度有限，所以难免会出现不同的 Key 通过运算**得到的 index 相同**，这种情况可以**利用链表**来解决，HashMap 会在 table[index]处形成链表，采用**头插法**将数据插入到链表中。
```java
public V put(K key, V value) {
    // 若“key为null”，则将该键值对添加到table[0]中。
    if (key == null)
        return putForNullKey(value);
    // 若“key不为null”，则计算该key的哈希值，然后将其添加到该哈希值对应的链表中。
    int hash = hash(key.hashCode());
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        // 若“该key”对应的键值对已经存在，则用新的value取代旧的value。然后退出！
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    // 若“该key”对应的键值对不存在，则将“key-value”添加到table中
    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```
- 若要添加到HashMap中的键值对对应的key**已经存在**HashMap中，则找到该键值对；然后**新的value取代旧的**value。
- 若要添加到HashMap中的键值对对应的key**不在HashMap中**，则将其添加到该哈希值对应的链表中，并**调用addEntry()**。

addEntry()

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    // 保存“bucketIndex”位置的值到“e”中
    Entry<K,V> e = table[bucketIndex];
    // 设置“bucketIndex”位置的元素为“新Entry”，
    // 设置“e”为“新Entry的下一个节点”
    table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
    // 若HashMap的实际大小 不小于 “阈值”，则调整HashMap的大小
    if (size++ >= threshold)
        resize(2 * table.length);
}
```
createEntry()

```java
void createEntry(int hash, K key, V value, int bucketIndex) {
    // 保存“bucketIndex”位置的值到“e”中
    Entry<K,V> e = table[bucketIndex];
    // 设置“bucketIndex”位置的元素为“新Entry”，
    // 设置“e”为“新Entry的下一个节点”
    table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
    size++;
}
```
区别：addEntry()会判断table大小是否超过阈值。

#### putAll()

```java
public void putAll(Map<? extends K, ? extends V> m) {
    // 有效性判断
    int numKeysToBeAdded = m.size();
    if (numKeysToBeAdded == 0)
        return;

    // 计算容量是否足够，
    // 若“当前实际容量 < 需要的容量”，则扩容。
    if (numKeysToBeAdded > threshold) {
        int targetCapacity = (int)(numKeysToBeAdded / loadFactor + 1);
        if (targetCapacity > MAXIMUM_CAPACITY)
            targetCapacity = MAXIMUM_CAPACITY;
        int newCapacity = table.length;
        while (newCapacity < targetCapacity)
            newCapacity <<= 1;
        if (newCapacity > table.length)
            resize(newCapacity);
    }

    // 通过迭代器，将“m”中的元素逐个添加到HashMap中。
    for (Iterator<? extends Map.Entry<? extends K, ? extends V>> i = m.entrySet().iterator(); i.hasNext(); ) {
        Map.Entry<? extends K, ? extends V> e = i.next();
        put(e.getKey(), e.getValue());
    }
}
```
#### remove()

```java
public V remove(Object key) {
    Entry<K,V> e = removeEntryForKey(key);
    return (e == null ? null : e.value);
}


// 删除“键为key”的元素
final Entry<K,V> removeEntryForKey(Object key) {
    // 获取哈希值。若key为null，则哈希值为0；否则调用hash()进行计算
    int hash = (key == null) ? 0 : hash(key.hashCode());
    int i = indexFor(hash, table.length);
    Entry<K,V> prev = table[i];
    Entry<K,V> e = prev;

    // 删除链表中“键为key”的元素
    // 本质是“删除单向链表中的节点”
    // e == null 即为链尾
    while (e != null) {
        Entry<K,V> next = e.next;
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k)))) {
            modCount++;
            size--;
            if (prev == e) // 说明这是头节点
                table[i] = next;
            else
                prev.next = next; // 把删除节点的prev和next连起来
            e.recordRemoval(this);
            return e;
        }
        prev = e;
        e = next;
    }

    return e;
}
```
### HashMap实现的Serializable接口
- 串行写入函数是writeObject()，它的作用是将HashMap的“**总的容量，实际容量，所有的Entry**”都写入到输出流中。
而串行读取函数是readObject()，它的作用是将HashMap的“**总的容量，实际容量，所有的Entry**”依次读出
## HashMap遍历方式
### 遍历HashMap的键值对
1. 根据entrySet()获取HashMap的“键值对”的Set集合。
2. 通过Iterator迭代器遍历“第一步”得到的集合。

```java
Integer integ = null;
Iterator iter = map.entrySet().iterator();
while(iter.hasNext()) {
    Map.Entry entry = (Map.Entry)iter.next();
    // 获取key
    key = (String)entry.getKey();
        // 获取value
    integ = (Integer)entry.getValue();
}
```
### 遍历HashMap的键/遍历HashMap的值
遍历HashMap的键/遍历HashMap的值与遍历HashMap的键值对类似，先通过keySet()/value()获取“键”的Set集合或者“值”的集合，再进行遍历。代码略。
