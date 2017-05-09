## 深入集合之ArrayList

​	集合作为JDK中使用最频繁的几个类之一，对于其具体实现形式的了解一直浮于表面。因此查看几个集合类的源码(本人以**JDK8**为参考源码)后有了接下来的几篇博文。而ArrayList作为使用做频繁的List，则理所当然是第一篇解析目标：

​	一个类的组成都是成员变量和成员方法，再细分看就是构造方法和其他方法。而一个类除非使用静态工厂生成实例，都摆脱不了用构造方法生成实例。因此以后都是以从成员变量到构造方法到其他方法的顺序学习集合类。

### 成员变量

``` java
 /**
     * Default initial capacity.
     * 默认的初始容量，即对应数组的大小
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances.
     * 为空白实例准备的共享空数组（空ArrayList的elementData都指向这个空白数组）
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     * 为默认空实例准备的空数组，和EMPTY_ELEMENTDATA唯一不同的是当第一个元素被添加后
     * ，数组elementData的长度被扩展到10
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     * ArrayList中的元素实际存储的数组。ArrayList的容量为这个数组的长度。当一个空白
     * ArrayList且它的elementData为DEFAULTCAPACITY_EMPTY_ELEMENTDATA添加第一个元
     * 素时，数组长度扩展到10
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     * ArrayList的大小
     * @serial
     */
    private int size;
```



### 构造方法

1. 无参构造


   ``` java
    public ArrayList() {
           this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
       }
   ```

  初始化一个空的ArrayList，但是capacity在添加第一个元素时变为10.

2. 参数为capacity的构造方法

   ``` java
   public ArrayList(int initialCapacity) {
           if (initialCapacity > 0) {
               this.elementData = new Object[initialCapacity];
           } else if (initialCapacity == 0) {
               this.elementData = EMPTY_ELEMENTDATA;
           } else {
               throw new IllegalArgumentException("Illegal Capacity: "+
                                                  initialCapacity);
           }
       }
   ```

   根据初始化的capacity初始化不同的数组。

3. 参数为collection的构造方法

   ``` java
   public ArrayList(Collection<? extends E> c) {
           elementData = c.toArray();
           if ((size = elementData.length) != 0) {
               // c.toArray might (incorrectly) not return Object[] (see 6260652)
               if (elementData.getClass() != Object[].class)
                   elementData = Arrays.copyOf(elementData, size, Object[].class);
           } else {
               // replace with empty array.
               this.elementData = EMPTY_ELEMENTDATA;
           }
       }
   ```

   ​	因为ArrayList的操作实际都转化到对elementData的操作。所以可以看到这个构造方法的关键在[toArray()](#toArray)和[Arrays.copyOf()](#copy)的操作。除此之外，无非是对集合元素的类型判断，默认为Object类型，如果不是，则重新造型。


### 其他成员方法

在众多方法中，先介绍几个使用频率最高，参与最紧密的。

##### toArray() <a id="toArray"></a> 

``` java 
 public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }
```

​	将ArrayList转化成数组的关键无疑还在[Arrays.copyOf()](#copy)中，只是默认转化的数组类型和elementData元素类型相同。

##### Arrays.copyOf() <a id="copy"></a>   

``` java
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```

​	如果newType是Object类，那么新建Object数组，长度为newLength。如果不是，则使用Native方法，根据ComponentType和长度生成对应类型数组。再使用Native方法将数据从源数组转移到新建数组。

##### 数组动态扩容

``` java
/*	public方法，对象可以在外部调用扩容。
	一个数组有三种情况：
	1、初始值为EMPTY_ELEMENTDATA  
	2、初始值为DEFAULTCAPACITY_EMPTY_ELEMENTDATA
	3、非空数组
	如果源数组为情况1或3，将入参minCapacity代入扩容。
	如果源数组为情况2，那么默认最小扩展为10，将minCapacity与10比较后扩容。此处可以看出源数组为			DEFAULTCAPACITY_EMPTY_ELEMENTDATA的差异。
*/
public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
	/*
		数组增加前做的扩容操作。如果源数组是DEFAULTCAPACITY_EMPTY_ELEMENTDATA，默认扩容至大小为10.
		如果minCapacity比DEFAUL_CAPACITY大，则扩容至minCapacity。
	*/
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
	//最终调用的数组扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     *	数组扩容的实际操作
     *	将minCapacity和和源数组长度的1.5倍(实际为右移一位，近似看为0.5倍)比较，取较大值。
     *	如果增长后的newCapacity比MAX_ARRAY_SIZE大，那么进入hugeCapacity()方法取扩展后的数组大小。
     *	最后使用Arrays.copyOf()生成一个新的长度为newCapacity的数组，且内容为源数组内容。
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
	//扩容更小的参数都超出int，那么抛出Error
	//否则给一个最大限定值。
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

```

​	以上代码可以呼应构造方法中，当elementData为DEFAULTCAPACITY_EMPTY_ELEMENTDATA时，只要发生数组扩容，默认扩容到数组长度为10。

##### List增删方法

``` java
E elementData(int index) {
        return (E) elementData[index];
    }

    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }

    //添加到数组尾部，先进行数组扩容，再直接对index为size++处赋值。
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
	//先进行数组扩容，再使用Native方法将数组从index以后的值复制到以index+1为开头处，再将index赋上
	//新增的element
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }

    //几乎是增加的反过程，还是arraycopy方法。将index后的值复制到index起始的位置。
	//需要注意消除引用，防止内存无法回收。
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

    //遍历找出第一个Object的索引值index，再根据index删除。
	//同样需要注意消除应用
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    //不需要返回值的删除方法。
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }

    
    public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    //操作过程相似：数组扩容，将新增c转换成的数组a添加到elementData尾部。同样通过arraycopy方法。
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    //step1:数组扩容
	//step2:数组index后内容后移
	//step3:将新增数组c内容复制到elementData中。
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

    
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // clear to let GC do its work
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }
```

​	从以上增删操作代码，可以看出涉及到的操作主要关于增加时的数组动态扩容和使用System.arraycopy()方法完成的数组移动(本质上也是数组复制)和数组复制，和删除时的同样使用System.arraycopy()完成数组移动和消除数组引用。

##### 迭代器

``` java
 public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    /**
     * Returns a list iterator over the elements in this list (in proper
     * sequence).
     *
     * <p>The returned list iterator is <a href="#fail-fast"><i>fail-fast</i></a>.
     *
     * @see #listIterator(int)
     */
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    /**
     * 返回一个迭代器
     */
    public Iterator<E> iterator() {
        return new Itr();
    }

    /**
     * 重写Iteartor接口的实现类
     */
    private class Itr implements Iterator<E> {
        int cursor;       // 下一个返回元素的位置
        int lastRet = -1; // 最后一个返回元素的位置
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }
		//初始化时，从index为0开始返回。
        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }
		//移除当前指向位置元素
        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

    /**
     * 支持从某个index开始遍历，可以向前访问。
     */
    private class ListItr extends Itr implements ListIterator<E> {
        ListItr(int index) {
            super();
            cursor = index;
        }

        public boolean hasPrevious() {
            return cursor != 0;
        }

        public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor - 1;
        }

        @SuppressWarnings("unchecked")
        public E previous() {
            checkForComodification();
            int i = cursor - 1;
            if (i < 0)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i;
            return (E) elementData[lastRet = i];
        }

        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.set(lastRet, e);
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                ArrayList.this.add(i, e);
                cursor = i + 1;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }
```

使用迭代器本意是想在不暴露类的内部实现的前提下，可以保证一致的遍历访问。

```java
//遍历删除问题
public static List<String> initialList(){
		List<String> list = new ArrayList<>();
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
```

* 方法1：虽然删除没问题，但可能遇到索引值与原数组索引不同的情况.
* 方法2：删除抛出 java.util.ConcurrentModificationException。因为foreach内部使用iteartor实现，导致modCount和expectedModCount不一致。
* 方法3：遍历删除最优解。

##### SubList

截取ArrayList从fromIndex到toIndex的部分。个人认为实现没有其他难点，关键点在于**所有的增删操作都是作用于源ArrayList**，因此使用时请注意这点。

### 结语

* ArrayList内部实现是基于数组，因此需要重点关注**动态扩容**和**数组拷贝**，从最外看到的index的改变其实是数组下标的改变，而数组下标的改变其实是使用Native方法arraycopy()。
* get()是常数级的时间复杂度，这比较于LinkedList强大很多。
* add(E e)直接添加到数组尾部，常数级时间复杂度。
* add(int index,E e) O(n)时间复杂度。越靠近数组首部需要移动的元素项越多。
* remove(int index) O(n)时间复杂度。越靠近数组首部需要移动的元素项越多。
* remove(Object o) O(n)时间复杂度。但是需要先遍历找到o，比上一种remove耗时长。

