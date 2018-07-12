## List使用场景
- 对于需要快速插入，删除元素，应该使用 `LinkedList`。
- - `Vector`、`ArrayList`、`Stack` 在插入时需要判断是否扩容。
- - `ArrayList` 除了需要扩容还需要移动数组，再插入或删除
- - `LinkedList` 在查找指定位置元素时使用**二分查找**。
- 对于需要快速随机访问元素，应该使用 `ArrayList`。
- `Vector`、`Stack` 线程安全，但效率低。`ArrayList`  线程不安全，适合用于单线程，多线程的用并发包 `CopyOnWriteArrayList`代替。
## Vector 与 ArrayList 区别
- `Vector`实现线程安全，大多数操作List的方法都加了 `synchronized`, 而ArrayList是非线程安全的。
- `ArrayList` 支持序列化，而 `Vector` 不支持
- `Vector` 除了包括和 `ArrayList` 类似的3个构造函数之外，另外的一个构造函数可以指定容量增加系数
- **容量增加方式**不同, `ArrayList`容量变为原来的1.5 倍（newCapacity = oldCapacity + (oldCapacity >> 1)），`Vector`则跟容量增加系数有关或者变为原来的2倍
- Vector支持通过 `Enumeration` 去遍历，而List不支持
