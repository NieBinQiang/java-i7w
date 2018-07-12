## Map概况
![image](https://images0.cnblogs.com/blog/497634/201309/08221402-aa63b46891d0466a87e54411cd920237.jpg)
- `Map` 是**键值对(key-value)映射接口**，Map中存储的内容是**键值对**(key-value)
- `AbstractMap` 是继承于Map的抽象类，它**实现了Map中的大部分API**。其它Map的实现类可以通过继承AbstractMap来**减少重复编码**。
- `SortedMap` 中的内容是**排序的键值对**，排序的方法是通过**比较器(Comparator)**
- 相比于`SortedMap` ，`NavigableMap` 有一系列的**导航方法**；如"获取大于/等于某对象的键值对"、“获取小于/等于某对象的键值对”等等。 
- `TreeMap` 实现了NavigableMap接口, 它保存的内容是“有序的键值对”
- 对比`TreeMap`，`HashMap` 没有实现NavigableMap接口, 所以保存的键值对**不保证次序**。
- `Hashtable` 它继承于Dictionary(Dictionary也是键值对的接口)，而且也实现Map接口

## Map

```
public interface Map<K,V> { }
```
- **Map映射中不能包含重复的键；每个键最多只能映射到一个值**
- Map 接口提供三种 `Collection` 视图，**键集keySet()**、**值集values()**或**键-值映射关系集entrySet()**
- `Map` 映射顺序。 `TreeMap` 有序， `HashMap`不保证次序。
- `Map` 提供了“键-值对”、“根据键获取值”、“删除键”、“获取容量大小”等方法。
## Map.Entry
```
interface Entry<K,V> { }
```
`Map.Entry` 定义了对`键值对`的操作
- equals(Object object)
- getKey() 获取Key
- getValue() 获取值
- hashCode() 获取hash值
- setValue(V object) 设置值 

## AbstractMap

```
public abstract class AbstractMap<K,V> implements Map<K,V> {}
```
`AbstractMap` 类提供 Map 接口的骨干实现。
## SortedMap
```
public interface SortedMap<K,V> extends Map<K,V> { }
```
 `SortedMap` 是一个**有序**的SortedMap键值映射。
 `SortedMap` 的排序方式有两种：**自然排序** 或者 **用户指定比较器**。 **插入有序 SortedMap 的所有元素都必须实现 Comparable 接口（或者被指定的比较器所接受）**。
 
 所有 `SortedMap` 实现类都应该提供 **4 个“标准”构造方法**：
 - **void（无参数）构造方法**，它创建一个空的有序映射，按照键的**自然顺序**进行排序。
 - **带有一个 `Comparator` 类型参数的构造方法**，它创建一个空的有序映射，根据**指定的比较器**进行排序。
 - **带有一个 Map 类型参数的构造方法**，它创建一个新的有序映射，其**键-值映射关系与参数相同**，按照键的**自然顺序**进行排序。
 - **带有一个 `SortedMap` 类型参数的构造方法**，它创建一个新的有序映射，其键-值映射关系和**排序方法与输入的有序映射相同**。无法保证强制实施此建议，因为接口不能包含构造方法。
## NavigableMap
`NavigableMap` 是一个**可导航的键-值对集合**,具有**了为给定搜索目标报告最接近匹配项的导航方法**。它分别提供了获取“键”、“键-值对”、“键集”、“键-值对集”的相关方法。

`NavigableMap` 提供的功能可以分为4类：
1. 提供操作键-值对的方法。
- lowerEntry() 返回**小于**给定键的 `Map.Entry`
- floorEntry() 返回**小于等于**给定键 `Map.Entry`
- ceilingEntry() 返回**大于等于**给定键 `Map.Entry`
- higherEntry() 返回**大于**给定键的键 `Map.Entry`
- firstEntry() 返回最小
- pollFirstEntry() 删除并返回最小，如果不存在就返回 `null`
- lastEntry() 返回最大
- pollLastEntry() 删除并返回最大，如果不存在就返回 `null`

2. 提供操作键的方法。这个和第1类比较类似,lowerKey()、floorKey()、ceilingKey() 和 higherKey() 
3. 获取键集。`navigableKeySet`、`descendingKeySet`分别获取正序/反序的键集。
4. 获取键-值对的子集。
