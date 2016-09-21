#java HashMap源码理解
转载博客：http://zhangshixi.iteye.com/blog/672697<br>
看了HashMap的源码才明白它的实现机制，HashMap利用哈希值来计算存储位置。<br>
HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/java/hashmap.jpg)<br>
###HashMap的存取实现：
#### 1. 存储
```java
public V put(K key, V value) {
  // HashMap允许存放null键和null值。
  // 当key为null时，调用putForNullKey方法，将value放置在数组第一个位置。
  if (key == null)
    return putForNullKey(value);  
      // 根据key的keyCode重新计算hash值。
       int hash = hash(key.hashCode());
        // 搜索指定hash值在对应table中的索引。
      int i = indexFor(hash, table.length);
      // 如果 i 索引处的 Entry 不为 null，通过循环不断遍历 e 元素的下一个元素。
      for (Entry<K,V> e = table[i]; e != null; e = e.next) {
          Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
              V oldValue = e.value;
              e.value = value;
              e.recordAccess(this);
              return oldValue;
        }
      }
    // 如果i索引处的Entry为null，表明此处还没有Entry。
    modCount++;
    // 将key、value添加到i索引处。
    addEntry(hash, key, value, i);
    return null;
  }
```
从上面的源代码中可以看出：当我们往HashMap中put元素的时候，先根据key的hashCode重新计算hash值，根据hash值得到这个元素在数组中的位置（即下标），
如果数组该位置上已经存放有其他元素了，那么在这个位置上的元素将以链表的形式存放，新加入的放在链头，最先加入的放在链尾。
如果数组该位置上没有元素，就直接将该元素放到此数组中的该位置上。<br>
addEntry(hash, key, value, i)方法根据计算出的hash值，将key-value对放在数组table的i索引处。addEntry 是HashMap 提供的一个包访问权限的方法，
代码如下：<br>
```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    // 获取指定 bucketIndex 索引处的 Entry
    Entry<K,V> e = table[bucketIndex];
    // 将新创建的 Entry 放入 bucketIndex 索引处，并让新的 Entry 指向原来的 Entry
    table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
    // 如果 Map 中的 key-value 对的数量超过了极限
    if (size++ >= threshold)
     // 把 table 对象的长度扩充到原来的2倍。
         resize(2 * table.length);
  }
```
当系统决定存储HashMap中的key-value对时，完全没有考虑Entry中的value，仅仅只是根据key来计算并决定每个Entry的存储位置。我们完全可以把 Map 集合中的 value 当成 key 的附属，
当系统决定了 key 的存储位置之后，value 随之保存在那里即可。<br>
hash(int h)方法根据key的hashCode重新计算一次散列。此算法加入了高位计算，防止低位不变，高位变化时，造成的hash冲突。<br>
####2. 读取
```java
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    int hash = hash(key.hashCode());
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
        e != null;e = e.next) {
         Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
            return e.value;
        }
    return null;
}
```
有了上面存储时的hash算法作为基础，理解起来这段代码就很容易了。从上面的源代码中可以看出：从HashMap中get元素时，首先计算key的hashCode，
找到数组中对应位置的某一元素，然后通过key的equals方法在对应位置的链表中找到需要的元素。<br>
归纳起来简单地说，HashMap 在底层将 key-value 当成一个整体进行处理，这个整体就是一个 Entry 对象。HashMap 底层采用一个 Entry[] 数组来保存所有的 key-value 对，
当需要存储一个 Entry 对象时，会根据hash算法来决定其在数组中的存储位置，在根据equals方法决定其在该数组位置上的链表中的存储位置；
当需要取出一个Entry时，也会根据hash算法找到其在数组中的存储位置，再根据equals方法从该位置上的链表中取出该Entry。<br>
对于 HashMap 及其子类而言，它们采用 Hash 算法来决定集合中元素的存储位置。当系统开始初始化 HashMap 时，系统会创建一个长度为 capacity 的 Entry 数组，
这个数组里可以存储元素的位置被称为“桶（bucket）”，每个 bucket 都有其指定索引，系统可以根据其索引快速访问该 bucket 里存储的元素。
无论何时，HashMap 的每个“桶”只存储一个元素（也就是一个 Entry），由于 Entry 对象可以包含一个引用变量（就是 Entry 构造器的的最后一个参数）
用于指向下一个 Entry，因此可能出现的情况是：HashMap 的 bucket 中只有一个 Entry，但这个 Entry 指向另一个 Entry ——这就形成了一个 Entry 链。<br>
如图 1 所示：<br>
![](https://github.com/MerlinYu/blog/raw/master/blog_file/java/hashmap_table.jpg)<br>

####3. HashMap的性能参数：
HashMap 包含如下几个构造器：<br>

1. HashMap()：构建一个初始容量为 16，负载因子为 0.75 的 HashMap。
2. HashMap(int initialCapacity)：构建一个初始容量为 initialCapacity，负载因子为 0.75 的 HashMap。
3. HashMap(int initialCapacity, float loadFactor)：以指定初始容量、指定的负载因子创建一个 HashMap。
4. HashMap的基础构造器HashMap(int initialCapacity, float loadFactor)带有两个参数，它们是初始容量initialCapacity和加载因子loadFactor。
5. initialCapacity：HashMap的最大容量，即为底层数组的长度。
6. loadFactor：负载因子loadFactor定义为：散列表的实际元素数目(n)/ 散列表的容量(m)。
7. 负载因子衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之愈小。对于使用链表法的散列表来说，
查找一个元素的平均时间是O(1+a)，因此如果负载因子越大，对空间的利用更充分，然而后果是查找效率的降低；如果负载因子太小，
那么散列表的数据将过于稀疏，对空间造成严重浪费。<br>
HashMap的实现中，通过threshold字段来判断HashMap的最大容量：<br>
```java
threshold = (int)(capacity * loadFactor);
```
结合负载因子的定义公式可知，threshold就是在此loadFactor和capacity对应下允许的最大元素数目，超过这个数目就重新resize，以降低实际的负载因子。默认的的负载因子0.75是对空间和时间效率的一个平衡选择。当容量超出此最大容量时， resize后的HashMap容量是容量的两倍：
```java
if (size++ >= threshold)
  resize(2 * table.length);
```
以上内容都是转载。<br>
概括来说：
通过Hash算法，计算出hasgIndex 确认Entry保存数据的位置，也就是table[] 的下标，如果再有Entry 的通过Hash算法算出的hashIndex 一样则把这个Entry以链表的形式保存。<br》
这样以链表的形式保存，相对于List读取数据的速度就快一些，但是遍历的速度会慢一些。<br>
