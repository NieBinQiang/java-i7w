# HashMap和Hashtable异同
## 相同点
- `HashMap`和`Hashtable`都是存储“键值对(key-value)”的散列表，而且核心实现方法都一样（**通`table`数组存储，数组的每一个元素都是一个`Entry`；而一个`Entry`就是一个单向链表，`Entry`链表中的每一个节点就保存了`key-value`键值对数据。）**
- `add()`: **根据`key`值求出`hash`值**，再**算出数组`index`**。然后，根据`index`**找到并遍历对应`Entry`**，将`key`和链表中的每一个节点`的key`进行对比。若`key`**已经存在**`Entry`链表中，则用**新值替换旧值**；若key**不存在**Entry链表中，**则新建一个`key-value`节点**，并将该节点插入`Entry`链表的**表头**位置。
- `remove()`: **根据`key`计算出`hash`，再计算出数组`index`。然后，根据index找出并遍历Entry**。若节点`key-value`存在与链表`Entry`中，则删除链表中的节点即可。
## 不同点
### 继承和实现方式不同、遍历种类不同
- `HashMap` 继承于`AbstractMap`，支持`Iterator`(迭代器)遍历。
- `Hashtable` 继承于`Dictionary`，同时又实现了Map接口，所以它既支持`Enumeration`遍历，也支持`Iterator`遍历。
### 线程安全不同
- `Hashtable`的几乎所有函数都是同步的，即它是线程安全的，支持多线程。
- `HashMap`的函数则是非同步的，它不是线程安全的。
### 对`null`值的处理不同
- **`HashMap`的`key`、`value`都可以为`null`**。当`HashMap`的`key`为`null`时，`HashMap`会将其**固定的插入`table[0]`位置**；而且**table[0]处只会容纳一个`key`为`null`的值**，当有多个`key`为`null`的值插入的时候，`table[0]`会**保留最后插入的`value`**。

```java
// 将“key-value”添加到HashMap中
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

// putForNullKey()的作用是将“key为null”键值对添加到table[0]位置
private V putForNullKey(V value) {
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            // recordAccess()函数什么也没有做
            e.recordAccess(this);
            return oldValue;
        }
    }
    // 添加第1个“key为null”的元素都table中的时候，会执行到这里。
    // 它的作用是将“设置table[0]的key为null，值为value”。
    modCount++;
    addEntry(0, null, value, 0);
    return null;
}
```

- **`Hashtable`的`key`、`value`都不可以为`null`**，会抛出异常`NullPointerException`。

```java
// 将“key-value”添加到Hashtable中
public synchronized V put(K key, V value) {
    // Hashtable中不能插入value为null的元素！！！
    if (value == null) {
        throw new NullPointerException();
    }

    // 若“Hashtable中已存在键为key的键值对”，
    // 则用“新的value”替换“旧的value”
    Entry tab[] = table;
    // Hashtable中不能插入key为null的元素！！！
    // 否则，下面的语句会抛出异常！
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    for (Entry<K,V> e = tab[index] ; e != null ; e = e.next) {
        if ((e.hash == hash) && e.key.equals(key)) {
            V old = e.value;
            e.value = value;
            return old;
        }
    }

    // 若“Hashtable中不存在键为key的键值对”，
    // (01) 将“修改统计数”+1
    modCount++;
    // (02) 若“Hashtable实际容量” > “阈值”(阈值=总的容量 * 加载因子)
    //  则调整Hashtable的大小
    if (count >= threshold) {
        // Rehash the table if the threshold is exceeded
        rehash();

        tab = table;
        index = (hash & 0x7FFFFFFF) % tab.length;
    }

    // (03) 将“Hashtable中index”位置的Entry(链表)保存到e中 Entry<K,V> e = tab[index];
    // (04) 创建“新的Entry节点”，并将“新的Entry”插入“Hashtable的index位置”，并设置e为“新的Entry”的下一个元素(即“新Entry”为链表表头)。        
    tab[index] = new Entry<K,V>(hash, key, value, e);
    // (05) 将“Hashtable的实际容量”+1
    count++;
    return null;
}
```
### 通过`Iterator`迭代器遍历时，遍历的顺序不同
- `HashMap`是“从前向后”的遍历数组；再对数组具体某一项对应的链表，从表头开始进行遍历。
- `Hashtable`是“从后往前”的遍历数组；再对数组具体某一项对应的链表，从表头开始进行遍历。
### 容量的初始值 和 增加方式都不一样
- `HashMap`默认的容量大小是16；增加容量时，每次将容量变为“原始容量x2”。
- `Hashtable`默认的容量大小是11；增加容量时，每次将容量变为“原始容量x2 + 1”。
### 添加`key-value`时的`hash`值算法不同
`HashMap`添加元素时，是使用自定义的哈希算法。

```java
static int hash(int h) {
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

`Hashtable`没有自定义哈希算法，而直接采用的`key`的`hashCode()`。

```java
int hash = key.hashCode();
```
### 部分API不同

- `Hashtable`支持`contains(Object value)`方法，而且重写了`toString()`方法；
- 而`HashMap`不支持`contains(Object value)`方法，没有重写`toString()`方法。


