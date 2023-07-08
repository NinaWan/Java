# HashMap JDK7

## 实现原理

数组+链表

每个数组元素为Entry<k,v>的链表

Entry<k,v>是HashMap内部定义的静态内部类：

* key
* value
* next，指向链表上下一个Entry对象
* hash，key对象经过hash函数得到的hashcode

第一次put的时候，才为数组分配内存空间

初始容量(数组的size)默认值为16，loadFactor默认值为0.75

size，Entry对象的个数

threshold，扩容的阈值，capacity*loadFactor

### 扩容机制

数组大小扩充为原来的2倍，index=hash&(n-1)，需要对旧数据重新计算index

数据转移逻辑：

* for entry in table // 遍历数组
    * while entry!=null // 遍历链表
      index = reindex(entry)
      next = entry.next
      entry.next = newTable[index]
      newTable[index] = entry
      entry = next
      table = newTable

可以看出，遍历链表的时候是从前往后遍历，同时进行表头插入，所以扩容后链表中节点的顺序与扩容前的顺序是不同的

### get(Object key)

key => hash => index => 遍历链表 => key.hash==entry.hash && (key== entry.key || key.equals(entry.key))

### resize(int newCapacity)

1. 如果table已达到最大容量，threshold赋值为整数最大值，直接返回；
2. 如果table可以扩容：
    * create a new table with new capacity；
    * call transfer(newTable, rehash) method to transfer all data from old table to new table:
        * for i in [0...table.length): // 遍历数组
            * for each entry in table[i]: // 遍历链表
                * 如果rehash is true，调用hash(entry.key)重新计算该entry的哈希值；
                * 重新计算该entry在table数组中对应的位置；
                * 将该entry插入到newTable中对应位置的链表的表头；
3. update threshold to min(newCapacity*loadFactor, MAXIMUM_CAPACITY+1)。

## Q&A

1. 为什么数组的长度一定是2的次幂？

    * index = hash & (n-1)，这样可以保证一次扩容后与扩容前只有最高位一位差异，能够保证最高位为0的entry
      reindex后还是能够map到相同的位置，减少位置变动
    * n-1低位全部为1，确保hash低位具有唯一性，并且不会产生空置的数组位置，否则如果有0位，则肯定有index永远不可达到

2. 为什么重写equals方法时一定要重写hashCode方法？

    * put和get都要使用hashCode来确定在数组中的位置，且要比较entry.hash来判断key是否已经存在；
    * equals相等，则hashCode也必须相同；equals不等，hashCode可以相同，此时发生哈希冲突。