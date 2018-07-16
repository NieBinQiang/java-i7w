# Set架构
![image](https://images0.cnblogs.com/blog/497634/201309/09223827-04741ce6b3f84b3ab76cee8dd316b403.jpg)
-  Set 是一个**不允许有重复元素**的集合,继承于Collection。
-  HastSet 和 TreeSet 是Set的两个实现类。
- - HashSet依赖于HashMap，它实际上是通过HashMap实现的。HashSet中的元素是**无序的**。
- - TreeSet依赖于TreeMap，它实际上是通过TreeMap实现的。TreeSet中的元素是**有序的**。
# HashSet
- HashSet 是一个没有重复元素的集合。
- 它是由HashMap实现的，不保证元素的顺序（无序的），而且HashSet允许使用 null 元素。
- HashSet是非同步的。
## hashSet构造函数

```java
// HashSet是通过map(HashMap对象)保存内容的

private transient HashMap<E,Object> map;

public HashSet() {
        map = new HashMap<>();
    }
    
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}
```
  
构造函数很简单，利用了 HashMap 初始化了 map 。
## add

```java
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```

add() 方法。，可以看出它是将存放的对象当做了 HashMap 的健，value 都是相同的 PRESENT。由于HashMap的key是不能重复的，所以每当有重复的值写入到 HashSet时，value会被覆盖，但key不会受到影响，这样就保证了 HashSet 中只能存放不重复的元素。
## 总结
HashSet 的原理比较简单，几乎全部借助于 HashMap 来实现的。所以 HashMap 会出现的问题 HashSet 依然不能避免。

# TreeSet
- TreeSet 是一个有序的集合
- 实现了NavigableSet<E>, Cloneable, java.io.Serializable接口。所以它支持一系列的导航方法、能被克隆、支持序列化
- TreeSet中的元素支持2种排序方式：自然排序 或者 根据创建TreeSet 时提供的 Comparator 进行排序（比较器排序）。
## TreeSet的构造函数

```java
// NavigableMap对象
private transient NavigableMap<E,Object> m;
// 不带参数的构造函数。创建一个空的TreeMap
public TreeSet() {
    this(new TreeMap<E,Object>());
}
// 将TreeMap赋值给 "NavigableMap对象m"
TreeSet(NavigableMap<E,Object> m) {
    this.m = m;
}
// 带比较器的构造函数。
public TreeSet(Comparator<? super E> comparator) {
    this(new TreeMap<E,Object>(comparator));
}
```
TreeSet构造函数利用了TreeMap 初始化了 map 。
## add

```java
// 添加e到TreeSet中
public boolean add(E e) {
    return m.put(e, PRESENT)==null;
}
```
和hashSet类似，TreeSet的add() 方法它也是将存放的对象当做了 TreeMap的add 的健，value 都是相同的 PRESENT。由于HashMap的key是不能重复的，所以每当有重复的值写入到 TreeSet时，value会被覆盖，但key不会受到影响，这样就保证了 TreeSet 中只能存放不重复的元素。同时，因为TreeMap的值是有序的，所以TreeSet中元素也是有序的。
## 其他方法
lower() 小于 floor() 小于等于 ceiling () 大于等于 higher() 大于
