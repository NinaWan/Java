# Map

## 扩容比较

### HashMap

2*n

### Hashtable

2*n+1

### ConcurrentHashMap

## [hash函数比较](https://hollischuang.github.io/toBeTopJavaer/#/basics/java-basic/hash-in-hashmap)

### HashMap VS Hashtable VS ConcurrentHashMap

### JDK1.7 VS JDK1.8

#### HashMap

**JDK1.8**

```
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

#### Hashtable

与JDK1.7类似，但是没有专门的hash方法，而是需要时直接调用key.hashCode()。

#### ConcurrentHashMap

hash方法改为spread方法

```
static final int spread(int h) {
    return (h ^ (h >>> 16)) & HASH_BITS;
}
```