## PriorityQueue源码
PriorityQueue 是根据优先级先进先出的列。
```java
  public void priorityQueue() {
    PriorityQueue queue = new PriorityQueue();
    queue.add("file");
    queue.add("data");
    queue.add("apple");
    queue.add("zone");
    queue.add("easy");
    System.out.println("init queue: " + queue.toString());
    while(!queue.isEmpty()) {
      System.out.println("poll:" + queue.poll());
      for (Object s : queue) {
        System.out.print(s.toString() + " ");
      }
      System.out.println("");
    }
  }
// log result:
init queue: [apple, easy, data, zone, file]
poll:apple
data easy file zone 
poll:data
easy zone file 
poll:easy
file zone 
poll:file
zone 
poll:zone
```
以上的结果可能与想像中的不一样。查看源码得到了结果，PriorityQueue的内部成员数组queue是以二叉树的方式存储的。其存储顺序如下：
```
      0
     / \
    1   2
   / \ / \
  3  4 5  6
```
## add 
```java
 @Override
    public boolean add(E o) {
        return offer(o);
    }
       public boolean offer(E o) {
        if (o == null) {
            throw new NullPointerException("o == null");
        }
        growToSize(size + 1);
        elements[size] = o;
        siftUp(size++);
        return true;
    }
        private void siftUp(int childIndex) {
        E target = elements[childIndex];
        int parentIndex;
        while (childIndex > 0) {
            parentIndex = (childIndex - 1) / 2;
            E parent = elements[parentIndex];
            if (compare(parent, target) <= 0) {
                break;
            }
            elements[childIndex] = parent;
            childIndex = parentIndex;
        }
        elements[childIndex] = target;
    }
```
add的关键函数是siftUp在siftUp中可以看到其添加的方式是以二叉树向上查找其根结点，并不断比较其大小，将其添加在合适的结点当中。当欲加入的元素小于其父节点时，
就将两个节点的位置交换。这个算法保证了如果只执行add操作，那么queue这个二叉树是有序的：该二叉树中的任意一个节点都小于以该节点为根节点的子数中的任意其它节点。
这也就保证了queue[0]，即队顶元素总是所有元素中最小的。
## poll 
```java
   public E poll() {
        if (isEmpty()) {
            return null;
        }
        E result = elements[0];
        removeAt(0);
        return result;
    }
    private void removeAt(int index) {
        size--;
        E moved = elements[size];
        elements[index] = moved;
        siftDown(index);
        elements[size] = null;
        if (moved == elements[index]) {
            siftUp(index);
        }
    }
      private void siftDown(int rootIndex) {
      E target = elements[rootIndex];
      int childIndex;
      while ((childIndex = rootIndex * 2 + 1) < size) {
          if (childIndex + 1 < size
                  && compare(elements[childIndex + 1], elements[childIndex]) < 0) {
              childIndex++;
          }
          if (compare(target, elements[childIndex]) <= 0) {
              break;
          }
          elements[rootIndex] = elements[childIndex];
          rootIndex = childIndex;
      }
      elements[rootIndex] = target;
  }
```
在poll时重新对二叉树排序，从代码可以看排序时只对其同一个二叉树的分支上排序，这样就保证了在remove时根结点保持最小。







