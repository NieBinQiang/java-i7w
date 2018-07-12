#  Stack
`Stack` 是栈。(`LIFO`, Last In First Out 后进先出)。Stack是继承于Vector(矢量队列)，这也就意味着**Stack也是通过数组实现的**，而非链表。
## 结构
![image](https://images0.cnblogs.com/blog/497634/201309/08213747-6f2f69ba19e9485f9f6ae8c17f0f253b.jpg)
#### 常用API 
- empty() 栈是否为空
- peek() 获取栈顶数据，返回数组末尾的元素
- pop() 删除并返回栈顶数据，取出数组末尾的元素，然后将该元素从数组中删除
- push(E object) 往栈顶添加数据，通过将元素追加的数组的末尾中
- search(Object o) 搜索
