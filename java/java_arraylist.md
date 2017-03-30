# java ArrayList
## 源码分析
#### 构造方法
```java
public ArrayList{
 array = EmptyArray.OBJECT;
}
public ArrayList(int capacity) {
    if (capacity < 0) {
        throw new IllegalArgumentException("capacity < 0: " + capacity);
    }
    array = (capacity == 0 ? EmptyArray.OBJECT : new Object[capacity]);
}
public ArrayList(Collection<? extends E> collection) {
    if (collection == null) {
        throw new NullPointerException("collection == null");
    }

    Object[] a = collection.toArray();
    if (a.getClass() != Object[].class) {
        Object[] newArray = new Object[a.length];
        System.arraycopy(a, 0, newArray, 0, a.length);
        a = newArray;
    }
    array = a;
    size = a.length;
}
```
从上面的代码可以看出ArrayList()构造方法，构造了一个Empty的Array,而ArrayList(capacity)指定了初始ArrayList的长度，<br>
第三种则是将Collection赋值给ArrayList。如果要比较构造方法的优劣的话需要再去看add源码
```java
@Override public boolean add(E object) {
    Object[] a = array;
    int s = size;
    if (s == a.length) {
        Object[] newArray = new Object[s +
                (s < (MIN_CAPACITY_INCREMENT / 2) ?
                 MIN_CAPACITY_INCREMENT : s >> 1)];
        System.arraycopy(a, 0, newArray, 0, s);
        array = a = newArray;
    }
    a[s] = object;
    size = s + 1;
    modCount++;
    return true;
}
```
从上代码分析，使用ArrayList(),在add时会重新分配一个大小为12的容量，之后add的时候如果容量不够，使用System.arraycopy函数进行1.5倍扩容。<br>
可想而知如果要添加到ArrayList的数据多的话会进行频繁的扩容操作，影响性能。<br>
而ArrayList(capacity)在一开始时分配了array的容量，会减少重新分配内存的操作。<br>
两种构造函数进行比较，如果ArrayList要存储的数据少则使用ArrayList(),如果可以预估到ArrayList要存储大量的数据，使用ArrayList(capacity)
#### 扩容
```java
public void ensureCapacity(int minimumCapacity) {
    Object[] a = array;
    if (a.length < minimumCapacity) {
        Object[] newArray = new Object[minimumCapacity];
        System.arraycopy(a, 0, newArray, 0, size);
        array = newArray;
        modCount++;
    }
}
```
### 与List的比较
1. ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。 <br>
2. 对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。 <br>
3. 对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。 <br>
4. 我们可以使用Collections.synchronized去保证Arraylist的线程安全。 List list=Collections.synchronizedList(new ArrayList())；<br>
5. CopyOnWriteArrayList是线程安全的List
