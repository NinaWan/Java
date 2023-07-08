# HashMap JDK8

## 底层结构

数组+链表/红黑树，当链表长度>=7时，将链表转化为红黑树

## put(K key, V value)

put(key, value) => putVal(hash, key, value, onlyIfAbsent, evict)

evict - false if the table is in creation mode

* 调用hash(key)，求出key对应的哈希值；

```
static final int hash(Object key){
    int h;
    return(key==null)?0:(h=key.hashCode())^(h>>>16);
}
```

* 调用putVal(hash, key, value, onlyIfAbsent, evict)
    1. 如果是第一次put，调用resize()方法来初始化table；
    2. (n-1)&hash，找到数组上对应的位置，判断该位置是否已有node
        * 如果没有，在该位置上插入新node；
        * 如果有：
            * 判断node.hash==hash && (node.key==key || node.key.equals(key))
                * 如果成立，设e=node;
                * 如果不成立:
                    * 如果node是tree node，则调用putTreeVal方法；
                    * 否则，遍历链表，e=node.next：
                        * 如果node不存在，在链表尾部插入新node；
                            * 如果链表长度大于等于7，调用treeifyBin方法将链表转化成红黑树；
                        * 如果node已存在，break遍历；
            * 如果e!=null 且 (!onlyIfAbsent 或者 旧值为null)，则用新值替换e中的旧值，并直接返回；
    3. modCount++；
    4. 如果新的size大于threshold，调用resize()方法来扩容；

## get(Object key)

get(key) => getNode(hash, key)

* 调用hash(key)，求出key对应的哈希值；
* 调用getNode(hash, key)
    * 如果table为null或者长度为0，或者hash对应数组位置为空，返回null；
    * 否则：
        * 如果数组对应位置第一个node即为所找的node，返回node；
        * 否则：
            * 如果该数组位置仅有一个node，返回null；
            * 如果该数组位置还有其他node：
                * 如果是TreeNode，调用getTreeNode(hash, key)；
                * 否则，遍历链表：
                    * 如果找到对应的node，返回node；
                    * 否则，返回null。

## resize()

1. pre-check of oldCap and oldThr
    * oldCap>0，
        * 如果oldCap>=最大容量，，返回oldTab;
        * 如果oldCap<最大容量 and newCap = 2*oldCap < 最大容量 and oldCap>=默认初始容量，newThr = 2*oldThr (double
          threshold);
    * oldCap<=0 and oldThr>0，newCap = oldThr; // initial capacity was placed in threshold
    * oldCap<=0 and oldThr<=0，zero initial threshold signifies using defaults:
        * newCap = DEFAULT_INITIAL_CAPACITY;
        * newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
2. if newThr==0，newThr = newCap<最大容量 and newCap*loadFactor<最大容量 ? newCap*loadFactor:Integer.MAX_VALUE;
3. threshold = newThr, table = newTab = new Node[newCap];
4. if oldTab!=null: // transfer data
    * for i in [0...oldCap): // 遍历oldTab
        * if e=oldTab[i]!=null:
            * set oldTab[i] = null;
            * if e.next==null, set newTab[e.hash&(newCap-1)] = e; // only one entry in oldTab[i]
            * else if e is a TreeNode, call e.split(this, newTab, i, oldCap);
            * else 遍历该链表
                * if e.hash&oldCap==0，将该entry插入到newTab[i]的链表尾部; // 原数组位置
                * if e.hash&oldCap==1，将该entry插入到newTab[i+oldCap]的链表尾部; // 原数组位置后移oldCap
5. return newTab;

