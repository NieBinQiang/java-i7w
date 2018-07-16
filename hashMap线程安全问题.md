## hashMap 线程安全问题
### 1、put方法值被覆盖

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
    
    // 新增Entry。将“key-value”插入指定位置，bucketIndex是位置索引。
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
如果有两个线程A和B，都进行插入数据，刚好这两条不同的数据经过哈希计算后得到的哈希码是一样的，且该位置还没有其他的数据。当A线程执行到addEntry（），还没执行完，系统切换至B线程，也执行addEntry（），则最后切换回A线程时，B线程的值会被A线程覆盖。

### 2、resize扩容产生环形链表
在并发场景发生扩容，调用 resize() 方法里的 rehash() 时，容易出现环形链表。
```java
// 重新调整HashMap的大小，newCapacity是调整后的单位
    void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        // 新建一个HashMap，将“旧HashMap”的全部元素添加到“新HashMap”中，
        // 然后，将“新HashMap”赋值给“旧HashMap”。
        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable);
        table = newTable;
        threshold = (int)(newCapacity * loadFactor);
    }

    // 将HashMap中的全部元素都添加到newTable中
    void transfer(Entry[] newTable) {
        Entry[] src = table;
        int newCapacity = newTable.length;
        for (int j = 0; j < src.length; j++) {
            Entry<K,V> e = src[j];
            if (e != null) {
                src[j] = null;
                do {
                    Entry<K,V> next = e.next;  // ----假设A在这里挂起
                    int i = indexFor(e.hash, newCapacity);
                    e.next = newTable[i];
                    newTable[i] = e;
                    e = next;
                } while (e != null);
            }
        }
    }
```
![image](http://incdn1.b0.upaiyun.com/2016/10/665f644e43731ff9db3d341da5c827e1.jpg)
假设AB线程都在对hashMap进行resize。A线程执行到resize的transfer中的“Entry<K,V> next = e.next”（此时保存的next为key(7)），然后被挂起，等B完成resize后A继续执行以下操作：
- 将key(3)插入index = 3 的头，然后执行e = next，这时next 为key(7)
- e!==null，继续将next=key(7)插入index = 3 的头，但此时Entry<K,V> next = e.next，是B线程resize后的key(7)的next，**即key(7)的next指向key(3)**,e = next = key(3)
- e!==null,最后执行到key(3),继续将next = key(3)插入index = 3 的头，**则key(3)的next指向key(7)**,然后next=null，结束插入。但是最终key(3)和key(7)形成环形链表。
-![image](http://incdn1.b0.upaiyun.com/2016/10/011ecee7d295c066ae68d4396215c3d0.jpg)

## 其他
在 JDK1.8 中对 HashMap 进行了优化： 当 hash 碰撞之后写入链表的长度超过了阈值(默认为8)，链表将会转换为红黑树。
多线程场景下推荐使用 **ConcurrentHashMap**。
