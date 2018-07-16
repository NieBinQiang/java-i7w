# LinkedHashMap
众所周知 `HashMap` 是一个无序的 Map，因为每次根据 `key` 的 `hashcode` 映射到 `Entry` 数组上，所以遍历出来的顺序并不是写入的顺序。

因此 `JDK` 推出一个基于 `HashMap` 但具有顺序的 `LinkedHashMap` 来解决有排序需求的场景。

它的底层是继承于 `HashMap` 实现的，由一个双向链表所构成。

`LinkedHashMap` 的排序方式有两种：

- 根据写入顺序排序。
- 根据访问顺序排序。
其中根据访问顺序排序时，每次 `get` 都会将访问的值移动到链表末尾，这样重复操作就能得到一个按照访问顺序排序的链表。

## 数据结构
### LinkedHashMap 基本元素 Entry
- `LinkedHashMap`采用的`hash`算法和`HashMap`相同，但是它重新定义了`Entry`。
- `LinkedHashMap`中的`Entry`增加了两个指针 `before` 和 `after`，它们分别用于维护双向链接列表。

```java
private static class Entry<K,V> extends HashMap.Entry<K,V> {

    // These fields comprise the doubly linked list used for iteration.
    Entry<K,V> before, after;

    Entry(int hash, K key, V value, HashMap.Entry<K,V> next) {
        super(hash, key, value, next);
    }
    ...
}
```
### LinkedHashMap 成员变量
与`HashMap`相比，`LinkedHashMap`增加了两个属性用于保证迭代顺序，分别是 **双向链表头结点`header`** 和 **标志位`accessOrder`** (值为`true`时，表示按照访问顺序迭代；值为`false`时，表示按照插入顺序迭代)。

```java
/**
 * The head of the doubly linked list.
 */
private transient Entry<K,V> header;  // 双向链表的表头元素

/**
 * The iteration ordering method for this linked hash map: <tt>true</tt>
 * for access-order, <tt>false</tt> for insertion-order.
 *
 * @serial
 */
private final boolean accessOrder;  //true表示按照访问顺序迭代，false时表示按照插入顺序 
```
## LinkedHashMap 具体实现
  `LinkedHashMap`是用`HashMap`实现的，所以整体上大致和`HashMap`一致。下面主要简单概括不同的地方实现。
###   构造函数
- `LinkedHashMap` 一共提供了五个构造函数，前四个基本与HashMap一致，只是多了`accessOrder`默认设为`false`。
- 第五个构造函数意在构造一个**指定初始容量**和**指定负载因子**的**具有指定迭代顺序**的`LinkedHashMap`。如下：
```java
public LinkedHashMap(int initialCapacity,
             float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);   // 调用HashMap对应的构造函数
        this.accessOrder = accessOrder;    // 迭代顺序的默认值
    }
```
同时重写了`HashMap`中的`init()`方法（在`HashMap`中`init()`为空的方法），用于初始化`LinkedHashMap` 的 `header`。

```java
@Override
void init() {
    header = new Entry<>(-1, null, null, null);
    header.before = header.after = header;
}
```

### put()
`LinkedHashMap`没有对`put(key,vlaue)`方法进行任何直接的修改，完全继承了`HashMap`的 `put(Key,Value)` 方法。但它重写了`put()`方法中的`addEntry()`和`recordAccess(this)`

`hashMap`的`put()`

```java
public V put(K key, V value) {

        //当key为null时，调用putForNullKey方法，并将该键值对保存到table的第一个位置 
        if (key == null)
            return putForNullKey(value); 

        //根据key的hashCode计算hash值
        int hash = hash(key.hashCode());           

        //计算该键值对在数组中的存储位置（哪个桶）
        int i = indexFor(hash, table.length);              

        //在table的第i个桶上进行迭代，寻找 key 保存的位置
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {      
            Object k;
            //判断该条链上是否存在hash值相同且key值相等的映射，若存在，则直接覆盖 value，并返回旧value
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this); // LinkedHashMap重写了Entry中的recordAccess方法--- (1)    
                return oldValue;    // 返回旧值
            }
        }

        modCount++; //修改次数增加1，快速失败机制

        //原Map中无该映射，将该添加至该链的链头
        addEntry(hash, key, value, i);  // LinkedHashMap重写了HashMap中的createEntry方法 ---- (2)    
        return null;
    }
```

重写的`addEntry()`，同时也对里面的`createEntry()`进行重写。
```java
void addEntry(int hash, K key, V value, int bucketIndex) {   

    //创建新的Entry，并插入到LinkedHashMap中  
    createEntry(hash, key, value, bucketIndex);  // 重写了HashMap中的createEntry方法

    //双向链表的第一个有效节点（header后的那个节点）为最近最少使用的节点，这是用来支持LRU算法的。
    
    Entry<K,V> eldest = header.after;  
    //如果有必要，则删除掉该近期最少使用的节点，  
    //removeEldestEntry默认返回false，具体实现靠子类重写该方法
    if (removeEldestEntry(eldest)) {  
        removeEntryForKey(eldest.key);  
    } else {  
        //扩容到原来的2倍  
        if (size >= threshold)  
            resize(2 * table.length);  
    }  
} 

void createEntry(int hash, K key, V value, int bucketIndex) { 
    // 向哈希表中插入Entry，这点与HashMap中相同 
    //创建新的Entry并将其链入到数组对应桶的链表的头结点处， 
    HashMap.Entry<K,V> old = table[bucketIndex];  
    Entry<K,V> e = new Entry<K,V>(hash, key, value, old);  
    table[bucketIndex] = e;     

    //在每次向哈希表插入Entry的同时，都会将其插入到双向链表的尾部，  
    //这样就按照Entry插入LinkedHashMap的先后顺序来迭代元素(LinkedHashMap根据双向链表重写了迭代器)
    //同时，新put进来的Entry是最近访问的Entry，把其放在链表末尾 ，也符合LRU算法的实现  
    e.addBefore(header);  
    size++;  
}  
```


重写的`recordAccess()`

```
void recordAccess(HashMap<K,V> m) {  
    LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;  
    //如果链表中元素按照访问顺序排序，则将当前访问的Entry移到双向循环链表的尾部，  
    //如果是按照插入的先后顺序排序，则不做任何事情。  
    if (lm.accessOrder) {  
        lm.modCount++;  
        //移除当前访问的Entry  
        remove();  
        //将当前访问的Entry插入到链表的尾部  
        addBefore(lm.header);  
    }  
} 

//写入到双向链表中
private void addBefore(Entry<K,V> existingEntry) {
    after  = existingEntry;
    before = existingEntry.before;
    before.after = this;
    after.before = this;
}  
```
### get()
`LinkedHashMap`也重写了`get()`

```java
public V get(Object key) {
        Entry<K,V> e = (Entry<K,V>)getEntry(key);
        if (e == null)
            return null;
            
        //多了一个判断是否是按照访问顺序排序，是则将当前的 Entry 移动到链表头部。   
        e.recordAccess(this);
        return e.value;
    }
    
    void recordAccess(HashMap<K,V> m) {
        LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
        if (lm.accessOrder) {
            lm.modCount++;
            
            //删除
            remove();
            //添加到头部
            addBefore(lm.header);
        }
    }
```
### resize()
`LinkedHashMap`对重哈希过程`transfer()`进行了重写.

```java
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;

    // 若 oldCapacity 已达到最大值，直接将 threshold 设为 Integer.MAX_VALUE
    if (oldCapacity == MAXIMUM_CAPACITY) {  
        threshold = Integer.MAX_VALUE;
        return;             // 直接返回
    }

    // 否则，创建一个更大的数组
    Entry[] newTable = new Entry[newCapacity];

    //将每条Entry重新哈希到新的数组中
    transfer(newTable);
    //LinkedHashMap对它所调用的transfer方法进行了重写

    table = newTable;
    threshold = (int)(newCapacity * loadFactor);  // 重新设定 threshold
}

void transfer(HashMap.Entry[] newTable) {
    int newCapacity = newTable.length;
    // 与HashMap相比，借助于双向链表的特点进行重哈希使得代码更加简洁
    for (Entry<K,V> e = header.after; e != header; e = e.after) {
        int index = indexFor(e.hash, newCapacity);   // 计算每个Entry所在的桶
        // 将其链入桶中的链表
        e.next = newTable[index];
        newTable[index] = e;   
    }
}
```
## 总结
- 总的来说 `LinkedHashMap` 其实就是对 `HashMap` 进行了拓展，使用了双向链表来保证了顺序性。
- 
- 因为是继承与 `HashMap `的，所以一些 `HashMap` 存在的问题 `LinkedHashMap` 也会存在，比如不支持并发等。


