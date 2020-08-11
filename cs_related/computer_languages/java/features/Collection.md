# Collection总结

## 概述
Collection的类主要是用来存储不定量个拥有相同类型的数据的数据类型  
一般来说分为以下几个类型：
+ set
+ map
+ list
+ queue

## set
特点是不能包含重复的元素
### HashSet
内部通过一个
```
private transient HashMap<E,Object> map;
```
进行具体数据的存储写入直接通过，当然线程不安全
```
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```
### LinkedHashSet
全部通过父类HashSet进行实现
通过父类提供的一个特定的构造方法进行构造
```
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
```
将父类的map对象从普通的HashMap转换为LinkedHashMap，其本质上和HashSet没啥区别

### TreeSet

内部通过一个map进行实现具体的实现代码
```
      public TreeSet() {
        this(new TreeMap<E,Object>());
    }
```
其中的大部分内容是在TreeMap中实现，所以同样也是线程不安全的

### CopyOnWriteArraySet
内部还是通过对应的其他集合类实现
CopyOnWriteArrayList
具体的实现就是直接调用方法
```
    public boolean add(E e) {
        return al.addIfAbsent(e);
    }
```
本来由于CopyOnWriteArrayList的线程安全实现，所以也是线程安全的

### ConcurrentSkipListSet
通过对应的ConcurrentSkipListMap进行实现  
实现方法通过调用
```
   public boolean add(E e) {
        return m.putIfAbsent(e, Boolean.TRUE) == null;
    }
```
实现

## Map

### ConcurrentSkipListMap
内部通过一个跳表进行实现
存在关键元素head,代表了跳表的表头
```
 private transient volatile HeadIndex<K,V> head;
```
每次循环开始的时候通过方法findPredecessor找到K的前继节点

## List

### ArrayList
### CopyOnWriteArrayList
内部维护一个array
```
private transient volatile Object[] array;
```
插入数据的时候通过一个重入锁进行保护，确保只有一个线程在进行插入操作，在所有进行数据更新/插入/删除的地方逻辑都一致
```
  public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray(); //获取到原有的数据
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1); //复制一份新的，加过元素的数据
            newElements[len] = e;
            setArray(newElements); //使得新的数据生效，通过volatile确保新的list的可见性
            return true;
        } finally {
            lock.unlock();
        }
    }
```
基本上分为以下几个步骤：
+ 获取到重入锁
+ 获取到原始数据
+ 复制获取一份新数据，并作对应的修改
+ 将新数据生效
+ 释放重入锁

这样一来能确保数据的线程安全性，并且在读写之间没有冲突，当然写写之间还是互斥的