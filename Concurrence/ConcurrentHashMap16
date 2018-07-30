# ConcurrentHashMap 概述
（基于[https://blog.csdn.net/justloveyou_/article/details/72783008](https://blog.csdn.net/justloveyou_/article/details/72783008)的总结）
- HashMap是非线程安全的，即在多线程环境下，HashMap**扩容重哈希**时出现**的死循环**问题、脏读问题等。JDK为HashMap提供了一个线程安全的高效版本 —— ConcurrentHashMap，解决上述多线程存在的问题。
- ConcurrentHashMap无论是读操作还是写操作都能保证很高的性能：在进行*读操作时(几乎)不需要加锁*，而在**写操作时通过锁分段技术**，**只对所操作的段加锁**而不影响客户端对其它段的访问。
- 在理想状态下，ConcurrentHashMap 可以支持** 16 个线程**执行**并发写操作**（如果并发级别设为16），及**任意数量线程的读操作。**
# HashMap 线程不安全的表现
### HashMap重哈希

```java
void transfer(Entry[] newTable) {

        // 将原数组 table 赋给数组 src
        Entry[] src = table;
        int newCapacity = newTable.length;

        // 将数组 src 中的每条链重新添加到 newTable 中
        for (int j = 0; j < src.length; j++) {
            Entry<K,V> e = src[j];
            if (e != null) {
                src[j] = null;   // src 回收

                // 将每条链的每个元素依次添加到 newTable 中相应的桶中
                do {
                    Entry<K,V> next = e.next;  // <--假设线程一执行到这里就被调度挂起了

                    // e.hash指的是 hash(key.hashCode())的返回值;
                    // 计算在newTable中的位置，注意原来在同一条子链上的元素可能被分配到不同的桶中
                    int i = indexFor(e.hash, newCapacity);   
                    e.next = newTable[i];
                    newTable[i] = e;
                    e = next;
                } while (e != null);
            }
        }
    }
```
假设有两个线程同时执行重哈希操作，线程一执行到Entry<K,V> next = e.next时被挂起，然后线程二开始执行完整的重哈希操作。完成后结果如下：

![image](http://static.zybuluo.com/Rico123/h53gjcs5jxoi10iv1mf3u9op/HashMap-rehash2.jpg)

当线程一在线程二执行完重哈希操作时，获得时间片，开始执行自己的重哈希。

根据重哈希代码逻辑，先从key3开始，重哈希到index为3的桶里，e.next = null；继续**遍历保存的next=key7**，如下图，key7重哈希也到了index为3的桶，**然而此时因为线程二的重哈希操作，key7的next是指向key3的**

![image](http://static.zybuluo.com/Rico123/13o929qaxxtmj6e1mvmx0agp/HashMap-rehash3.jpg)

因此在再遍历一次key3，这时的next=null，而重哈希后key3的next指向key7，此时**正好key7与key3形成一个环**，最终结束重哈希。

![image](http://static.zybuluo.com/Rico123/crzhiw1ipy684a87mxd08phh/HashMap-rehash5.jpg)

因此，**HashMap多线程环境下，进行扩容重哈希时导致Entry链形成环。**
# ConcurrentHashMap 整体结构
## ConcurrentHashMap 类结构
ConcurrentHashMap 继承了AbstractMap并实现了ConcurrentMap接口
```java
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V>
        implements ConcurrentMap<K, V>, Serializable {

    ...
}
```
## 成员变量
ConcurrentHashMap 增加了两个属性，segmentMask和segmentShift，用于定位段。
segmentMask可以理解掩码（类似计算机网络）
segmentShift理解为偏移量

```
    final int segmentMask;  // 用于定位段，大小等于segments数组的大小减 1，是不可变的

    /**
     * Shift value for indexing within segments.
     */
    final int segmentShift;    // 用于定位段，大小等于32(hash值的位数)减去对segments的大小取以2为底的对数值，是不可变的

    /**
     * The segments, each of which is a specialized hash table
     */
    final Segment<K,V>[] segments;   // ConcurrentHashMap的底层结构是一个Segment数组
```
## Segment 的定义
- Segment 类**继承于 ReentrantLock 类**，从而使得 Segment 对象能**充当锁**的角色。
- 每个 Segment 对象用来守护它的成员对象 table 中包含的若干个桶，table 是一个由 HashEntry 对象组成的链表数组。即每个segment相当于一个小型的HashMap

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {

        transient volatile int count;    // Segment中元素的数量，可见的

        transient int modCount;  //对count的大小造成影响的操作的次数（比如put或者remove操作）
      
        transient int threshold;      // 阈值，段中元素的数量超过这个值就会对Segment进行扩容


        transient volatile HashEntry<K,V>[] table;  // 链表数组

        final float loadFactor;  // 段的负载因子，其值等同于ConcurrentHashMap的负载因子

        ...
    }
```
段的结构图如下：

![image](http://static.zybuluo.com/Rico123/frevj4s6ka9n70v3i4qzeaxi/segment.jpg)
- 每个segment保存一个count，因此更新count个数时，不用更新整个concurrentHashMap。同时**count是volatile的**，这使得对count的任何更新**对其它线程都是立即可见**的。  
- ConcurrentHashMap允许**多个修改(写)操作并发进行**，其关键在于使用了**锁分段技术**，它使**用了不同的锁来控制对哈希表的不同部分进行的修改(写)**，而 ConcurrentHashMap 内部使用段(Segment)来表示这些不同的部分。
## 基本元素 HashEntry
- HashEntry包含属性与hashMap的Entry几乎一样。但不同的是，**在HashEntry类中，key，hash和next域都被声明为final的，value域被volatile所修饰，因此HashEntry对象几乎是不可变的，这是ConcurrentHashmap读操作并不需要加锁的一个重要原因。**(不可变性是线程安全的)
- **value域被volatile修饰，所以其可以确保被读线程读到最新的值**，这是ConcurrentHashmap读操作并不需要加锁的另一个重要原因。

```
static final class HashEntry<K,V> {
       final K key;                       // 声明 key 为 final 的
       final int hash;                   // 声明 hash 值为 final 的
       volatile V value;                // 声明 value 被volatile所修饰
       final HashEntry<K,V> next;      // 声明 next 为 final 的

        HashEntry(K key, int hash, HashEntry<K,V> next, V value) {
            this.key = key;
            this.hash = hash;
            this.next = next;
            this.value = value;
        }

        @SuppressWarnings("unchecked")
        static final <K,V> HashEntry<K,V>[] newArray(int i) {
        return new HashEntry[i];
        }
    }
```
# ConcurrentHashMap 的构造函数
该构造函数意在构造一个具有**指定容量、指定负载因子和指定段数目/并发级别**(若不是2的幂次方，则会**调整为2的幂次方**)的空ConcurrentHashMap

```java
 public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();

    if (concurrencyLevel > MAX_SEGMENTS)              
        concurrencyLevel = MAX_SEGMENTS;

       // Find power-of-two sizes best matching arguments
    int sshift = 0;            // 大小为 lg(ssize) 
    int ssize = 1;            // 段的数目，segments数组的大小(2的幂次方)
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    segmentShift = 32 - sshift;      // 用于定位段
    segmentMask = ssize - 1;      // 用于定位段
    this.segments = Segment.newArray(ssize);   // 创建segments数组

    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    int c = initialCapacity / ssize;    // 总的桶数/总的段数
    if (c * ssize < initialCapacity)
        ++c;
    int cap = 1;     // 每个段所拥有的桶的数目(2的幂次方)
    while (cap < c)
        cap <<= 1;

    for (int i = 0; i < this.segments.length; ++i)      // 初始化segments数组
        this.segments[i] = new Segment<K,V>(cap, loadFactor);
    }
```
- 默认的容量为16，默认负载因子为0.75，默认的并发级别为16。    
- 自定义的段数量为n，n为输入并发量的最小2的n次幂的n；其中每个段的数量为，大于等于（自定义容量除以端数量的结果）的最小整数最小2的m次幂。
- **因此每个段的容量也是2的n次幂，有利与之后的扩容操作**

# ConcurrentHashMap 的并发存取
在ConcurrentHashMap中，线程对映射表做读操作时，一般情况下不需要加锁就可以完成，**对容器做结构性修改的操作(比如，put操作、remove操作等)才需要加锁**。
## 并发写操作: put(key, vlaue)

```java
public V put(K key, V value) {
    if (value == null)
        throw new NullPointerException();
    int hash = hash(key.hashCode());
    return segmentFor(hash).put(key, hash, value, false);
}
```
- ConcurrentHashMap既不允许key值为null，也不允许value值为null。
- put操作被ConcurrentHashMap委托给特定的段来实现。
具体操作:
### 先定位到具体的段
- 取32位hash值的高n位于2^n - 1 做与运算，**结果即为高n位**，其中n为容量的最小2的n次幂的n。（hash >>> segmentShift 右移segmentShift位，segmentShift为 32 - n，segmentMask为2^n - 1 ）
- **根据key的hash值的高n位就可以确定元素到底在哪一个Segment中。**

```java
final Segment<K,V> segmentFor(int hash) {
    return segments[(hash >>> segmentShift) & segmentMask];
}
```
### 再插入指定段中桶
具体的桶定位：int index = hash & (tab.length - 1); 即hash值的低n位

```java
V put(K key, int hash, V value, boolean onlyIfAbsent) {
    lock();    // 上锁
    try {
        int c = count;
        if (c++ > threshold) // ensure capacity
            rehash();
        HashEntry<K,V>[] tab = table;    // table是Volatile的
        int index = hash & (tab.length - 1);    // 定位到段中特定的桶
        HashEntry<K,V> first = tab[index];   // first指向桶中链表的表头
        HashEntry<K,V> e = first;
            
        // 检查该桶中是否存在相同key的结点
        while (e != null && (e.hash != hash || !key.equals(e.key)))  
            e = e.next;

        V oldValue;
        if (e != null) {        // 该桶中存在相同key的结点
            oldValue = e.value;
            if (!onlyIfAbsent)
                e.value = value;        // 更新value值
        } else {         // 该桶中不存在相同key的结点
            oldValue = null;
            ++modCount;     // 结构性修改，modCount加1
            tab[index] = new HashEntry<K,V>(key, hash, first, value);  // 创建HashEntry并将其链到表头
            count = c;      //write-volatile，count值的更新一定要放在最后一步(volatile变量)
        }
        return oldValue;    // 返回旧值(该桶中不存在相同key的结点，则返回null)
    } finally {
        unlock();      // 在finally子句中解锁
    }
}
```
- put操作跟hashmap的操作大致相同。先判定大小是否超过阈值，如果是则重哈希。否则定位到具体的桶，然后遍历对应的hashEntry链表，如果存在key则修改；如果不存在则新增一个新的hashEntry节点插入在表头。对比hashMap这里加了锁操作。
- put操作中的加锁操作是针对某个具体的Segment，锁定的也是该Segment而不是整个ConcurrentHashMap。

## 重哈希操作 : rehash()
ConcurrentHashMap的重哈希实际上是对ConcurrentHashMap的**某个段的重哈希**，因此ConcurrentHashMap的**每个段所包含的桶位自然也就不尽相同**。

```java
void rehash() {
    HashEntry<K,V>[] oldTable = table;    // 扩容前的table
    int oldCapacity = oldTable.length;
    if (oldCapacity >= MAXIMUM_CAPACITY)   // 已经扩到最大容量，直接返回
        return;

    // 新创建一个table，其容量是原来的2倍
    HashEntry<K,V>[] newTable = HashEntry.newArray(oldCapacity<<1);   
    threshold = (int)(newTable.length * loadFactor);   // 新的阈值
    int sizeMask = newTable.length - 1;     // 用于定位桶
    for (int i = 0; i < oldCapacity ; i++) {
        // We need to guarantee that any existing reads of old Map can
        //  proceed. So we cannot yet null out each bin.
        HashEntry<K,V> e = oldTable[i];  // 依次指向旧table中的每个桶的链表表头

        if (e != null) {    // 旧table的该桶中链表不为空
            HashEntry<K,V> next = e.next;
            int idx = e.hash & sizeMask;   // 重哈希已定位到新桶
            if (next == null)    //  旧table的该桶中只有一个节点
                newTable[idx] = e;
            else {    
                // Reuse trailing consecutive sequence at same slot
                HashEntry<K,V> lastRun = e;
                int lastIdx = idx;
                for (HashEntry<K,V> last = next;last != null;last = last.next) {
                    int k = last.hash & sizeMask;
                    // 寻找k值相同的子链，该子链尾节点与父链的尾节点必须是同一个
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }

                // JDK直接将子链lastRun放到newTable[lastIdx]桶中
                newTable[lastIdx] = lastRun;

                // 对该子链之前的结点，JDK会挨个遍历并把它们复制到新桶中
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    int k = p.hash & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    newTable[k] = new HashEntry<K,V>(p.key, p.hash,
                                                     n, p.value);
                }
            }
        }
    }
    table = newTable;   // 扩容完成
}
```
1. 重哈希操作跟hashMap类似，新建一个新的HashEntry数组，大小为**原来的两倍**；然后遍历原来的hashEntry数组的每一个链表上的节点；
2. 如果链表只有一个节点，**重用插入**到新的数组新的位置。
3. 如果链表不止一个节点，则找到最后一个K值相同的子链的位置，把自该位置以后的子链**重用插入**到数组的新位置；最后把剩下的没重用插入的节点，**重新新建**一个新节点插入到计算出的index位置。

- 在初始化ConcurrentHashMap的时候，已经把每个段的大小设置成**2的n次幂**
- 计算节点的桶的位置时，是使用hash值的低n为，因此再扩容一倍的时候，计算桶的序号变为hash值的n+1位，即**重哈希的时候节点的桶的序号要么不变（n+1位为0），要么在原来位置基础上加上2^n（n+1位）。**
- 因为HashEntry的next属性是final，所以不能改变，需要创建一个新的HashEntry。可以用过重用最后k值相同的子链，来减少新建的开销，提升性能。


## 读取实现 ：get(Object key)
先段定位

```java
    public V get(Object key) {
        int hash = hash(key.hashCode());
        return segmentFor(hash).get(key, hash);
    }
```
get操作也和hashMap类似，定位到指定桶序号，遍历该链表，找到hash值相同且key值相同的节点，返回对应value。
```java
V get(Object key, int hash) {
    if (count != 0) {            // read-volatile，首先读 count 变量
        HashEntry<K,V> e = getFirst(hash);   // 获取桶中链表头结点
        while (e != null) {
            if (e.hash == hash && key.equals(e.key)) {    // 查找链中是否存在指定Key的键值对
                V v = e.value;
                if (v != null)  // 如果读到value域不为 null，直接返回
                    return v;   
                // 如果读到value域为null，说明发生了重排序，加锁后重新读取
                return readValueUnderLock(e); // recheck
            }
            e = e.next;
        }
    }
    return null;  // 如果不存在，直接返回null
}
```
- 需要注意的是，返回value之前需要判断value是否为空。一般情况，ConcurrentHashMap不允许value值为null，但是get操作发生的同时发生指令重排，获取到的HashEntry数组是rehash后但未初始化的HashEntry数组，因此会返回null值。因此获取到null值就需要**加锁重读**。

```java
V readValueUnderLock(HashEntry<K,V> e) {
    lock();
    try {
        return e.value;
    } finally {
        unlock();
    }
}
```

## remove 操作，remove(Object key)
首先需要定位到特定的段
```java
public V remove(Object key) {
    int hash = hash(key.hashCode());
    return segmentFor(hash).remove(key, hash, null);
}
```
将删除操作委派给该段
```java
V remove(Object key, int hash, Object value) {
    lock();     // 加锁
    try {
        int c = count - 1;      
        HashEntry<K,V>[] tab = table;
        int index = hash & (tab.length - 1);        // 定位桶
        HashEntry<K,V> first = tab[index];
        HashEntry<K,V> e = first;
        while (e != null && (e.hash != hash || !key.equals(e.key)))  // 查找待删除的键值对
            e = e.next;

        V oldValue = null;
        if (e != null) {    // 找到
            V v = e.value;
            if (value == null || value.equals(v)) {
                oldValue = v;
                // All entries following removed node can stay
                // in list, but all preceding ones need to be
                // cloned.
                ++modCount;
                // 所有处于待删除节点之后的节点原样保留在链表中
                HashEntry<K,V> newFirst = e.next;
                // 所有处于待删除节点之前的节点被克隆到新链表中
                for (HashEntry<K,V> p = first; p != e; p = p.next)
                    newFirst = new HashEntry<K,V>(p.key, p.hash,newFirst, p.value); 

                tab[index] = newFirst;   // 将删除指定节点并重组后的链重新放到桶中
                count = c;      // write-volatile，更新Volatile变量count
            }
        }
        return oldValue;
    } finally {
        unlock();          // finally子句解锁
    }
}
```
- 找到删除节点，把删除节点后的节点放入一个新的链表中，然后遍历删除节点前的节点，把他们逐个克隆到新的链表中，最终把心的链表替换原来的桶的链表。
- 删除操作执行过程，是通过**新建链表和克隆**的形式进行，所以删除过程中，不影响其他线程对原链表的读取。**即读线程不会受同时执行 remove 操作的并发写线程的干扰。**

![image](https://img-blog.csdn.net/20170527170625098?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvanVzdGxvdmV5b3Vf/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### 用Volatile变量协调读写线程间的内存可见性
- 在**happen-before的原则**里，有volatile的变量规则：**对一个变量的写操作先行发生于后面对这个变量的读操作**；即对concurrentHashMap中标有volatile的操作的变量进行同时进行读、写操作时（如count，和hashEntry的value），java内存模型会保证读线程能读取到写线程更新后的值。

![image](http://static.zybuluo.com/Rico123/bms6tb3f24x4jx3ot4eh3rqt/volatile.jpg)

- 正因为如此对如count等标记有volatile的变量，**写操作**（put、remove、clear）等都会在操作**结束前一刻去更新count值**，而**读操作**在读开始的**第一时间就读取count值**，从而保证完成写操作后，读线程才对count值进行读取。

## ConcurrentHashMap 跨段操作
跨段操作：size操作、containsValaue操作等。

```
public int size() {
    final Segment<K,V>[] segments = this.segments;
    long sum = 0;
    long check = 0;
        int[] mc = new int[segments.length];
    // Try a few times to get accurate count. On failure due to
    // continuous async changes in table, resort to locking.
    for (int k = 0; k < RETRIES_BEFORE_LOCK; ++k) {
        check = 0;
        sum = 0;
        int mcsum = 0;
        for (int i = 0; i < segments.length; ++i) {
            sum += segments[i].count;   
            mcsum += mc[i] = segments[i].modCount;  // 在统计size时记录modCount
        }
        if (mcsum != 0) {
            for (int i = 0; i < segments.length; ++i) {
                check += segments[i].count;
                if (mc[i] != segments[i].modCount) {  // 统计size后比较各段的modCount是否发生变化
                    check = -1; // force retry
                    break;
                }
            }
        }
        if (check == sum)// 如果统计size前后各段的modCount没变，且两次得到的总数一致，直接返回
            break;
    }
    if (check != sum) { // Resort to locking all segments  // 加锁统计
        sum = 0;
        for (int i = 0; i < segments.length; ++i)
            segments[i].lock();
        for (int i = 0; i < segments.length; ++i)
            sum += segments[i].count;
        for (int i = 0; i < segments.length; ++i)
            segments[i].unlock();
    }
    if (sum > Integer.MAX_VALUE)
        return Integer.MAX_VALUE;
    else
        return (int)sum;
}
```
1. 先在没有加锁的情况对所有段大小求和，最多执行RETRIES_BEFORE_LOCK次(默认是两次)
2. 在超过RETRIES_BEFORE_LOCK之后,还不成功就在持有所有段锁的情况下再对所有段大小求和(加锁)
3. JDK只需要在统计size前后**比较modCount是否发生变化**就可以得知容器的大小是否发生变化。
