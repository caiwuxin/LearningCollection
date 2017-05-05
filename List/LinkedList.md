​	这一篇是有关LinkedList的学习，那么闲话不多扯，直接按照上一篇的博文的模式来分析LinkedList的实现和功能。

### 成员变量

``` java
	transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;

	private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

由定义的成员变量和Node类的结构可以看出LinkedList是以双向链表为实现形式的List结构。

### 构造方法

1. 无参构造

   ``` java
   public LinkedList() {
       }
   ```

   没有任何其他的操作，仅仅构造一个空的List。

2. 参数为Collection的构造

   ``` java
   public LinkedList(Collection<? extends E> c) {
           this();
           addAll(c);
       }
   ```

   很显然，先构造一个空List，然后执行*addAll(c)*方法，将Collection c添加入List。关于[addAll()](#addAll)方法的介绍请看下文。

### 成员方法

因为对LinkedList的操作均是传递到对双向链表的操作，所以先来看对链表的基础操作：

#### 操作双向链表方法

1. 添加第一个节点

   ``` java
   private void linkFirst(E e) {
           final Node<E> f = first;
           final Node<E> newNode = new Node<>(null, e, f);
           first = newNode;
           if (f == null)
               last = newNode;
           else
               f.prev = newNode;
           size++;
           modCount++;
       }
   ```

   ​	先将f指向原来初始节点对象对应的内存空间，再新建新的初始节点，定义前置为null，内容为e，后置为f（原来的初始节点），同时令first指向新建的初始节点的内存空间。

   ![当前状态](http://opgm519yg.bkt.clouddn.com/state.png)

   ​	因此如果之前没有初始节点，只需要将last指向newNode即可。如果有初始节点，将初始节点f的prev指向newNode即可完成。再将List大小+1，更改次数+1.

2. 作为最后一个元素节点添加

   ``` java
   void linkLast(E e) {
           final Node<E> l = last;
           final Node<E> newNode = new Node<>(l, e, null);
           last = newNode;
           if (l == null)
               first = newNode;
           else
               l.next = newNode;
           size++;
           modCount++;
       }
   ```

   ​	插入过程与第一个很类似。先将 l 指向原来的终节点，再新建一个新的终节点，并将last指向新建的终节点。如果原来没有终节点，则将first也指向新建节点，此时链表中只有一个节点。若原来有终节点，将原来终节点的后置节点指向新建的终节点，完成连接。最后将表大小+1，更改次数+1.

3. 解绑链表的第一个节点

   ``` java
   private E unlinkFirst(Node<E> f) {
           // assert f == first && f != null;
           final E element = f.item;
           final Node<E> next = f.next;
           f.item = null;
           f.next = null; // help GC
           first = next;
           if (next == null)
               last = null;
           else
               next.prev = null;
           size--;
           modCount++;
           return element;
       }
   ```

   ​	从链表角度来看，解绑初始节点，无非是将first节点指向第二个节点，再将第二个节点next的prev设为null。而java实现中，需要将初始节点f的item和next设为null，以便对象f没有对象引用方便GC。当只有链表只有一个节点时，需要将last设为null。

4. 解绑链表的最后一个节点

   ``` java
   private E unlinkLast(Node<E> l) {
           // assert l == last && l != null;
           final E element = l.item;
           final Node<E> prev = l.prev;
           l.item = null;
           l.prev = null; // help GC
           last = prev;
           if (prev == null)
               first = null;
           else
               prev.next = null;
           size--;
           modCount++;
           return element;
       }
   ```

   ​	同样从双向链表的角度来看，唯一值得关注的操作就是让l的item引用和prev引用为null，方便GC。其他操作是将last指向prev节点。如果prev为null，则将first设为null，否则另prev的后置节点为null。

5. 在一个节点前插入一个元素

   ``` java
   void linkBefore(E e, Node<E> succ) {
           // assert succ != null;
           final Node<E> pred = succ.prev;
           final Node<E> newNode = new Node<>(pred, e, succ);
           succ.prev = newNode;
           if (pred == null)
               first = newNode;
           else
               pred.next = newNode;
           size++;
           modCount++;
       }
   ```

   ​	定义succ之前的前置节点为pred，新建插入节点。将succ的前置节点设为新建节点。如果succ没有前置节点，则将first指向新建节点；若有，则将pred的后置节点指向新建节点newNode。

6. 移除链表中一个非空节点

   ``` java
   E unlink(Node<E> x) {
           // assert x != null;
           final E element = x.item;
           final Node<E> next = x.next;
           final Node<E> prev = x.prev;

           if (prev == null) {
               first = next;
           } else {
               prev.next = next;
               x.prev = null;
           }

           if (next == null) {
               last = prev;
           } else {
               next.prev = prev;
               x.next = null;
           }

           x.item = null;
           size--;
           modCount++;
           return element;
       }
   ```

   ​	移除节点x，定义x的前置节点为prev，后置节点为next。正常移除时需要将prev的后置引用指向next，next的前置引用指向prev。如果prev或next为空，则将first指向next或last指向prev。同时清除x残留的引用。大小-1，更改次数+1.

   #### 操作LinkedList方法

   <a id="addAll"></a> this this

   ​



