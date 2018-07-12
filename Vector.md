# Vector
Vector 是**矢量队列**，继承于AbstractList，实现了List, RandomAccess, Cloneable这些接口。
- Vector 继承了AbstractList，实现了List，所有是一个队列，支持相关的添加、删除、修改、遍历等功能。
- Vector 实现了RandmoAccess接口，即提供了随机访问功能。
- Vector 实现了Cloneable接口，它能被克隆。
- 和ArrayList不同，**Vector是同步容器**。
## 数据结构
![image](https://images0.cnblogs.com/blog/497634/201401/272347229531613.jpg)

&emsp;它包含了3个成员变量：elementData , elementCount， capacityIncrement。
- elementData： 动态数组，保存了添加到Vector中的元素
- elementCount： 动态数组的实际大小
- capacityIncrement： 动态数组的增长系数。每次当Vector中动态数组容量增加时增加的大小
## 源码解析
Voctor 底层数据结构和 ArrayList 类似,也是一个动态数组存放数据。不过是大多数操作数据方法的时候使用 synchronize 进行同步写数据，但是开销较大，所以 Vector 是一个同步容器并不是一个并发容器。如：
- void copyInto(Object[] anArray)
- void trimToSize()
- void ensureCapacity(int minCapacity)
- ...等等

确认“Vector容量”函数
```java
private void ensureCapacityHelper(int minCapacity) {
    int oldCapacity = elementData.length;
    // 当Vector的容量不足，则增加容量大小。
    // 若容量增量系数>0(即capacityIncrement>0)，则将容量增大当capacityIncrement
    // 否则，将容量增大一倍。
    if (minCapacity > oldCapacity) {
        Object[] oldData = elementData;
        int newCapacity = (capacityIncrement > 0) ?
            (oldCapacity + capacityIncrement) : (oldCapacity * 2);
        if (newCapacity < minCapacity) {
           newCapacity = minCapacity;
        }
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
}
```
指定位置插入数据:
```java
public synchronized void insertElementAt(E obj, int index) {
        modCount++;
        if (index > elementCount) {
            throw new ArrayIndexOutOfBoundsException(index
                                                     + " > " + elementCount);
        }
        ensureCapacityHelper(elementCount + 1);
        System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
        elementData[index] = obj;
        elementCount++;
    }
```

## 遍历方式
- 迭代器遍历 （效率最低）
- 随机访问  （效率最高）
- for循环
- Enumeration遍历

```
Integer value = null;
Enumeration enu = vec.elements();
while (enu.hasMoreElements()) {
    value = (Integer)enu.nextElement();
}
```
