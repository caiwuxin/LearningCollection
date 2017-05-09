	这一篇是有关LinkedList的学习，那么闲话不多扯，直接按照上一篇的博文的模式来分析LinkedList的实现和功能。

成员变量

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

由定义的成员变量和Node类的结构可以看出LinkedList是以双向链表为实现形式的List结构。

构造方法

1. 无参构造
       public LinkedList() {
           }
   没有任何其他的操作，仅仅构造一个空的List。
2. 参数为Collection的构造
       public LinkedList(Collection<? extends E> c) {
               this();
               addAll(c);
           }
   很显然，先构造一个空List，然后执行addAll(c)方法，将Collection c添加入List。关于addAll()方法的介绍请看下文。

成员方法

因为对LinkedList的操作均是传递到对双向链表的操作，所以先来看对链表的基础操作：

操作双向链表方法

1. 添加第一个节点
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
   	先将f指向原来初始节点对象对应的内存空间，再新建新的初始节点，定义前置为null，内容为e，后置为f（原来的初始节点），同时令first指向新建的初始节点的内存空间。
   
   	因此如果之前没有初始节点，只需要将last指向newNode即可。如果有初始节点，将初始节点f的prev指向newNode即可完成。再将List大小+1，更改次数+1.
2. 作为最后一个元素节点添加
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
   	插入过程与第一个很类似。先将 l 指向原来的终节点，再新建一个新的终节点，并将last指向新建的终节点。如果原来没有终节点，则将first也指向新建节点，此时链表中只有一个节点。若原来有终节点，将原来终节点的后置节点指向新建的终节点，完成连接。最后将表大小+1，更改次数+1.
3. 解绑链表的第一个节点
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
   	从链表角度来看，解绑初始节点，无非是将first节点指向第二个节点，再将第二个节点next的prev设为null。而java实现中，需要将初始节点f的item和next设为null，以便对象f没有对象引用方便GC。当只有链表只有一个节点时，需要将last设为null。
4. 解绑链表的最后一个节点
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
   	同样从双向链表的角度来看，唯一值得关注的操作就是让l的item引用和prev引用为null，方便GC。其他操作是将last指向prev节点。如果prev为null，则将first设为null，否则另prev的后置节点为null。
5. 在一个节点前插入一个元素
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
   	定义succ之前的前置节点为pred，新建插入节点。将succ的前置节点设为新建节点。如果succ没有前置节点，则将first指向新建节点；若有，则将pred的后置节点指向新建节点newNode。
6. 移除链表中一个非空节点
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
   	移除节点x，定义x的前置节点为prev，后置节点为next。正常移除时需要将prev的后置引用指向next，next的前置引用指向prev。如果prev或next为空，则将first指向next或last指向prev。同时清除x残留的引用。大小-1，更改次数+1.
   操作LinkedList方法
   常用方法
       public E getFirst() {
               final Node<E> f = first;
               if (f == null)
                   throw new NoSuchElementException();
               return f.item;
           }
       
       public E getLast() {
               final Node<E> l = last;
               if (l == null)
                   throw new NoSuchElementException();
               return l.item;
           }
       
       public E removeFirst() {
               final Node<E> f = first;
               if (f == null)
                   throw new NoSuchElementException();
               return unlinkFirst(f);
           }
       
       public E removeLast() {
               final Node<E> l = last;
               if (l == null)
                   throw new NoSuchElementException();
               return unlinkLast(l);
           }
       
       public void addFirst(E e) {
               linkFirst(e);
           }
       
       public void addLast(E e) {
               linkLast(e);
           }
       
       public int size() {
               return size;
           }
       
       public boolean add(E e) {
               linkLast(e);
               return true;
           }
       
       public boolean remove(Object o) {
               if (o == null) {
                   for (Node<E> x = first; x != null; x = x.next) {
                       if (x.item == null) {
                           unlink(x);
                           return true;
                       }
                   }
               } else {
                   for (Node<E> x = first; x != null; x = x.next) {
                       if (o.equals(x.item)) {
                           unlink(x);
                           return true;
                       }
                   }
               }
               return false;
           }
       
       	public E get(int index) {
               checkElementIndex(index);
               return node(index).item;
           }
       	
           public E set(int index, E element) {
               checkElementIndex(index);
               Node<E> x = node(index);
               E oldVal = x.item;
               x.item = element;
               return oldVal;
           }
   以上方法常用，且操作简单，都是对链表的简单操作。因此在这不加多余的解释，相信大家也能理解。
   addAll  <a id="addAll"></a>
       public boolean addAll(Collection<? extends E> c) {
               return addAll(size, c);
           }
       
       public boolean addAll(int index, Collection<? extends E> c) {
               checkPositionIndex(index);
       
               Object[] a = c.toArray();
               int numNew = a.length;
               if (numNew == 0)
                   return false;
       
               Node<E> pred, succ;
               if (index == size) {
                   succ = null;
                   pred = last;
               } else {
                   succ = node(index);
                   pred = succ.prev;
               }
       
               for (Object o : a) {
                   @SuppressWarnings("unchecked") E e = (E) o;
                   Node<E> newNode = new Node<>(pred, e, null);
                   if (pred == null)
                       first = newNode;
                   else
                       pred.next = newNode;
                   pred = newNode;
               }
       
               if (succ == null) {
                   last = pred;
               } else {
                   pred.next = succ;
                   succ.prev = pred;
               }
       
               size += numNew;
               modCount++;
               return true;
           }
       
       //将LinkedList转化成数组，实际实现是遍历链表，将链表中的值存入数组内。
       public Object[] toArray() {
               Object[] result = new Object[size];
               int i = 0;
               for (Node<E> x = first; x != null; x = x.next)
                   result[i++] = x.item;
               return result;
           }
       
       //找到处于index位置的节点，if-else判断是为了找出耗时较少的遍历途径，返回Node
       Node<E> node(int index) {
               // assert isElementIndex(index);
       
               if (index < (size >> 1)) {
                   Node<E> x = first;
                   for (int i = 0; i < index; i++)
                       x = x.next;
                   return x;
               } else {
                   Node<E> x = last;
                   for (int i = size - 1; i > index; i--)
                       x = x.prev;
                   return x;
               }
           }
   	作为addAll方法，实现的本质还是在链表的index位置插入内容为c的链表。定义succ为当前插入位置的节点，pred为插入位置的前置节点。插入过程就是遍历内容数组，新建一个节点，将pred的后置节点指向新建节点newNode。完成后，后移pred指向新建节点。数组循环完成后，建立最后一个新建节点和插入位置节点succ的关联。至此，操作完成。
   	而其他的成员方法，都是对链表的操作，均可以调用操作链表的方法，所以不再一一介绍，如果有兴趣可以自己查看源码。由于LinkedList采用双向链表实现，所以不存在数组扩容的问题。通过以上方法可以看出，LinkedList的增删操作开销很小，只需要遍历到目标位置，进行链表节点的增删，而对于在表起点和终点的增删，更只是常数时间的操作。但相较于ArrayList的get方法，LinkedList仍需要遍历链表才能找到索引对应值，效率不好。
   

迭代器的实现

	想要在不暴露LinkedList的内部实现的前提下，遍历访问List使用迭代器是一种很好的选择。接下来就是介绍LinkedList中的迭代器实现：

    public class LinkedList<E>
        extends AbstractSequentialList<E>
        implements List<E>, Deque<E>, Cloneable, java.io.Serializable
    {
      	//step 4
     	public ListIterator<E> listIterator(int index) {
            checkPositionIndex(index);
            return new ListItr(index);
        } 
    }



    public abstract class AbstractSequentialList<E> extends AbstractList<E> {
      //step 1	
      public Iterator<E> iterator() {
            return listIterator();
      }
      //step 3
      public abstract ListIterator<E> listIterator(int index); 
    }



    public abstract class AbstractList<E> extends AbstractCollection<E> implements List<E> {
     
      public Iterator<E> iterator() {
            return new Itr();
        }
    
        //step 2 
        public ListIterator<E> listIterator() {
            return listIterator(0);
        }
      
        public ListIterator<E> listIterator(final int index) {
            rangeCheckForAdd(index);
    
            return new ListItr(index);
        }
    }

按照以上的执行顺序可以看到，LinkedList的iterator()的实现最终追溯到内部类ListItr。

    private class ListItr implements ListIterator<E> {
            private Node<E> lastReturned = null;//可以理解为当前访问的节点
            private Node<E> next;
            private int nextIndex;
            private int expectedModCount = modCount;
    
            ListItr(int index) {
                // assert isPositionIndex(index);
                next = (index == size) ? null : node(index);
                nextIndex = index;
            }
    		//判断是否有下个元素，将下个元素索引值和List大小比较
            public boolean hasNext() {
                return nextIndex < size;
            }
    		//返回索引指向的元素，并将next指向当前的后置节点，索引值+1。
            public E next() {
                checkForComodification();//迭代器和初始List一致性校验
                if (!hasNext())
                    throw new NoSuchElementException();
    
                lastReturned = next;
                next = next.next;
                nextIndex++;
                return lastReturned.item;
            }
    
            public boolean hasPrevious() {
                return nextIndex > 0;
            }
    		//返回当前next节点指向的前置节点，并将next指向当前的前置节点，索引值-1.
      		//主要应用于定义的另一种倒叙遍历方式
            public E previous() {
                checkForComodification();
                if (!hasPrevious())
                    throw new NoSuchElementException();
    
                lastReturned = next = (next == null) ? last : next.prev;
                nextIndex--;
                return lastReturned.item;
            }
    
            public int nextIndex() {
                return nextIndex;
            }
    
            public int previousIndex() {
                return nextIndex - 1;
            }
    		//把当前位置节点移除
            public void remove() {
                checkForComodification();
                if (lastReturned == null)
                    throw new IllegalStateException();
    
                Node<E> lastNext = lastReturned.next;
                unlink(lastReturned);
                if (next == lastReturned)
                    next = lastNext;
                else
                    nextIndex--;
                lastReturned = null;
                expectedModCount++;
            }
    
            public void set(E e) {
                if (lastReturned == null)
                    throw new IllegalStateException();
                checkForComodification();
                lastReturned.item = e;
            }
    
            public void add(E e) {
                checkForComodification();
                lastReturned = null;
                if (next == null)
                    linkLast(e);
                else
                    linkBefore(e, next);
                nextIndex++;
                expectedModCount++;
            }
    
            final void checkForComodification() {
                if (modCount != expectedModCount)
                    throw new ConcurrentModificationException();
            }
        }

既然提到迭代遍历，也可以顺带讨论List遍历时删除元素的问题：

    public static List<String> initialList(){
    		List<String> list = new LinkedList<>();
    		list.add("a");
    		list.add("b");
    		list.add("c");
    		list.add("d");
    		return list;
    	}
    	
    	public static void removeByFor(List<String> list){
    		for (int i = 0; i < list.size(); i++) {
    			if("b".equals(list.get(i))){
    				System.out.println(i);
    				list.remove(i);
    			}
    			if("d".equals(list.get(i))){
    				System.out.println(i);
    				list.remove(i);
    			}
    		}
    		System.out.println(list);
    	}
    	
    	public static void removeByForeach(List<String> list){
    		for (String string : list) {
    			if ("b".equals(string)) {
    				list.remove(string);
    			}
    		}
    		System.out.println(list);
    	}
    	
    	public static void removeByIterator(List<String>list){
    		Iterator<String> e = list.iterator();
    		while (e.hasNext()) {
    			String item = e.next();
    			if ("b".equals(item)) {
    				e.remove();
    			}
    		}
    		System.out.println(list);
    	}
    	
    	public static void main(String []args){
    		removeByFor(initialList());			//1
    		removeByForeach(initialList());		//2
    		removeByIterator(initialList());	//3
    	}

- 方法1：能正常删除，但是删除d时，索引index为2，和原List中的索引值不同，可能会对其他操作造成影响。且删除操作经历了两次遍历(外部一次，remove操作一次)，时间复杂度增加。
- 方法2：会抛出ConcurrentModificationException异常。因为foreach内部也是使用iterator进行遍历，方法2操作只更改了modCount，没有更改Iterator中的expectedModCount。
- 方法3：最合适的遍历删除，只经历一次遍历，时间复杂度低。

结语

LinkedList的实现是基于双向链表，因此它的一切性质都是与双向链表相关的：

- get(int index) O(n) 由于双向链表需要依次遍历到目标index，所以get效率不如ArrayList。
- add(E e) O(1) 添加到链表尾部，只需要建立关联即可。
- add(int index,E e) O(n) 由于需要遍历到目标index，再添加节点建立关联。但是效率比ArrayList的移动数组好了不少。
- remove(int index) O(n) 和add方法类似，耗时主要在遍历到目标index。整体效率比ArrayList的remove()方法好。


