# synchronized

## 用法

1. 同步范围
    * 同步方法
    * 同步代码块

## 原理

1. 同步方法

   用ACC_SYNCHRONIZED标识符标记

2. 同步代码块

   用monitorenter、monitorexit两个指令标识

## 锁范围

* 同步方法
    1. 普通方法

       锁的是对应的对象实例。

    2. 静态方法

       锁的是对应的类。

* 同步代码块

    1. synchronized(ClassName.class)

       锁的是对应的类，无论有多少个实例。

    2. synchronized(instance)

       锁的是对应的对象实例。

    3. synchronized(this)

       第二种的特例，锁的是当前对象的实例。

   
