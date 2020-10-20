# Java基础总结

## 一、Java集合

### 1. 简介

Java集合大致可以分为Set、List、Queue和Map四种体系。

- Set代表无序、不可重复的集合；

- List代表有序、重复的集合；

- Map则代表具有映射关系的集合；

- Java 5 又增加了Queue体系集合，代表一种队列集合实现。

Java集合就像一种容器，可以把多个对象（实际上是对象的引用，但习惯上都称对象）“丢进”该容器中。从Java 5 增加了泛型以后，Java集合可以记住容器中对象的数据类型，使得编码更加简洁、健壮。

#### Java集合和数组的区别：

① 数组长度在初始化时指定，意味着只能保存定长的数据。而集合可以保存数量不确定的数据。同时可以保存具有映射关系的数据（即关联数组，键值对 key-value）。

② 数组元素即可以是基本类型的值，也可以是对象。集合里只能保存对象（实际上只是保存对象的引用变量），基本数据类型的变量要转换成对应的包装类才能放入集合类中。

#### Java集合类之间的继承关系:

Java的集合类主要由两个接口派生而出：Collection和Map,Collection和Map是Java集合框架的根接口。

[![img](https://camo.githubusercontent.com/b1974bdd58e78c12612907d0ad27b893f8fbdbb9/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d653766656266333634643864383233352e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/b1974bdd58e78c12612907d0ad27b893f8fbdbb9/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d653766656266333634643864383233352e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

ArrayList,HashSet,LinkedList,TreeSet是我们经常会有用到的已实现的集合类。

Map实现类用于保存具有映射关系的数据。Map保存的每项数据都是key-value对，也就是由key和value两个值组成。Map里的key是不可重复的，key用户标识集合里的每项数据。

[![img](https://camo.githubusercontent.com/517258d1f1e996c7a728c8fe067c439722fc61ce/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d303630353231303738343961373630332e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/517258d1f1e996c7a728c8fe067c439722fc61ce/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d303630353231303738343961373630332e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

图中，HashMap，TreeMap是我们经常会用到的集合类。

### 2. Collection接口

Collection接口是Set,Queue,List的父接口。Collection接口中定义了多种方法可供其子类进行实现，以实现数据操作。

#### 接口中定义的方法

[![img](https://camo.githubusercontent.com/7d400226aea928038bdbf8aaecafd37c9507c84d/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d343134333332666665343733333237342e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/7d400226aea928038bdbf8aaecafd37c9507c84d/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d343134333332666665343733333237342e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

Collection用法有：添加元素，删除元素，返回Collection集合的个数以及清空集合等。 其中重点介绍iterator()方法，该方法的返回值是Iterator。

#### 使用Iterator遍历集合元素

Iterator接口经常被称作迭代器，它是Collection接口的父接口。但Iterator主要用于遍历集合中的元素。 Iterator接口中主要定义了2个方法：

[![img](https://camo.githubusercontent.com/94d08bd4a166618a5059617b18c0986b8c83b423/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d363337333761326438313731336134372e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/94d08bd4a166618a5059617b18c0986b8c83b423/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d363337333761326438313731336134372e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

下面程序简单示范了通过Iterator对象逐个获取元素的逻辑。

```java
public class IteratorExample {
	public static void main(String[] args){
		//创建集合，添加元素  
		Collection<Day> days = new ArrayList<Day>();
		for(int i =0;i<10;i++){
			Day day = new Day(i,i*60,i*3600);
			days.add(day);
		}
		//获取days集合的迭代器
		Iterator<Day> iterator = days.iterator();
		while(iterator.hasNext()){//判断是否有下一个元素
			Day next = iterator.next();//取出该元素
			//逐个遍历，取得元素后进行后续操作
			.....
		}
	}

}
```

**注意：** 当使用Iterator对集合元素进行迭代时，把集合元素的值传给了迭代变量（就如同参数传递是值传递，基本数据类型传递的是值，引用类型传递的仅仅是对象的**引用变量**。

下面的程序演示了这一点：

```java
public class IteratorExample {
  public static void main(String[] args) {
            List<MyObject> list = new ArrayList<>();
            for (int i = 0; i < 10; i++) {
                list.add(new MyObject(i));
            }

            System.out.println(list.toString());

            Iterator<MyObject> iterator = list.iterator();//集合元素的值传给了迭代变量，仅仅传递了对象引用。保存的仅仅是指向对象内存空间的地址
            while (iterator.hasNext()) {
                MyObject next = iterator.next();
                next.num = 99;
            }

            System.out.println(list.toString());
    }
    static class MyObject {
        int num;

        MyObject(int num) {
            this.num = num;
        }

        @Override
        public String toString() {
            return String.valueOf(num);
        }
    }
}
```

输出结果如下：

> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>
> [99, 99, 99, 99, 99, 99, 99, 99, 99, 99]

下面具体介绍Collection接口的三个子接口Set，List，Queue。

### 3. Set集合

Set集合与Collection集合基本相同，没有提供任何额外的方法。实际上Set就是Collection，只是行为略有不同（Set不允许包含重复元素）。

Set集合不允许包含相同的元素，如果试图把两个相同的元素加入同一个Set集合中，则添加操作失败，add()方法返回false，且新元素不会被加入。

### 4. List集合

List集合代表一个元素有序、可重复的集合，集合中每个元素都有其对应的顺序索引。List集合允许使用重复元素，可以通过索引来访问指定位置的集合元素 。List集合默认按元素的添加顺序设置元素的索引，例如第一个添加的元素索引为0，第二个添加的元素索引为1......

List作为Collection接口的子接口，可以使用Collection接口里的全部方法。而且由于List是有序集合，因此List集合里增加了一些根据索引来操作集合元素的方法。

#### 接口中定义的方法

```java
void add(int index, Object element)//在列表的指定位置插入指定元素（可选操作）。

boolean addAll(int index, Collection<? extends E> c) //将集合c 中的所有元素都插入到列表中的指定位置index处。

Object get(index)//返回列表中指定位置的元素。

int indexOf(Object o)//返回此列表中第一次出现的指定元素的索引；如果此列表不包含该元素，则返回 -1。

int lastIndexOf(Object o)//返回此列表中最后出现的指定元素的索引；如果列表不包含此元素，则返回 -1。

Object remove(int index)//移除列表中指定位置的元素。

Object set(int index, Object element)//用指定元素替换列表中指定位置的元素。

List subList(int fromIndex, int toIndex)//返回列表中指定的 fromIndex（包括 ）和 toIndex（不包括）之间的所有集合元素组成的子集。

Object[] toArray()//返回按适当顺序包含列表中的所有元素的数组（从第一个元素到最后一个元素）。
```

除此之外，Java 8还为List接口添加了如下两个默认方法。

```java
void replaceAll(UnaryOperator operator) //根据operator指定的计算规则重新设置List集合的所有元素。

void sort(Comparator c) //根据Comparator参数对List集合的元素排序。
```

### 5.Queue集合

Queue用户模拟队列这种数据结构，队列通常是指“先进先出”(FIFO，first-in-first-out)的容器。队列的头部是在队列中存放时间最长的元素，队列的尾部是保存在队列中存放时间最短的元素。新元素插入（offer）到队列的尾部，访问元素（poll）操作会返回队列头部的元素。通常，队列不允许随机访问队列中的元素。

#### 接口中定义的方法

[![img](https://camo.githubusercontent.com/ea9536a97f141da1ccde21e817c1e88525152571/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d303530353535343933306361393832652e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/ea9536a97f141da1ccde21e817c1e88525152571/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d303530353535343933306361393832652e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

### 6. Map集合

Map用户保存具有映射关系的数据，因此Map集合里保存着两组数，一组值用户保存Map里的key,另一组值用户保存Map里的value，key和value都可以是任何引用类型的数据。Map的key不允许重复，即同一个Map对象的任何两个key通过equals方法比较总是返回false。

如下图所描述，key和value之间存在单向一对一关系，即通过指定的key,总能找到唯一的、确定的value。从Map中取出数据时，只要给出指定的key，就可以取出对应的value。

#### Map与Set、List的关系

**与Set集合的关系**

如果把Map里的所有key放在一起看，它们就组成了一个Set集合（所有的key没有顺序，key与key之间不能重复），实际上Map确实包含了一个keySet()方法，用户返回Map里所有key组成的Set集合。

**与List集合的关系**

如果把Map里的所有value放在一起来看，它们又非常类似于一个List：元素与元素之间可以重复，每个元素可以根据索引来查找，只是Map中索引不再使用整数值，而是以另外一个对象作为索引。

#### 接口中定义的方法

[![img](https://camo.githubusercontent.com/bc04f53016c688e34b13170f19c6257bb27a0dda/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d643234393435313665316436386136642e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/bc04f53016c688e34b13170f19c6257bb27a0dda/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d643234393435313665316436386136642e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

Map中还包括一个内部类Entry，该类封装了一个key-value对。Entry包含如下三个方法：

[![img](https://camo.githubusercontent.com/1fe243d5c5e69b601f9c2c3925a54a4527d72812/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d656365646431383830616639643430612e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)](https://camo.githubusercontent.com/1fe243d5c5e69b601f9c2c3925a54a4527d72812/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f333938353536332d656365646431383830616639643430612e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

Map集合最典型的用法就是成对地添加、删除key-value对，然后就是判断该Map中是否包含指定key，是否包含指定value，也可以通过Map提供的keySet()方法获取所有key组成的集合，然后使用foreach循环来遍历Map的所有key，根据key即可遍历所有的value。

### 7. ArrayList

以数组实现。节约空间，但数组有容量限制。超出限制时会增加50%容量，用System.arraycopy()复制到新的数组，因此最好能给出数组大小的预估值。默认第一次插入元素时创建大小为10的数组。

按数组下标访问元素—get(i)/set(i,e) 的性能很高，这是数组的基本优势。

直接在数组末尾加入元素—add(e)的性能也高，但如果按下标插入、删除元素—add(i,e), remove(i), remove(e)，则要用System.arraycopy()来移动部分受影响的元素，性能就变差了，这是基本劣势。

然后再来学习一下官方文档：

> **Resizable-array**implementation of the List interface. Implements all optional list operations, and permits all elements, including null. In addition to implementing the List interface, this class provides methods to manipulate the size of the array that is used internally to store the list. (This class is roughly equivalent to Vector, except that it is unsynchronized.)

ArrayList是一个相对来说比较简单的数据结构，最重要的一点就是它的自动扩容，可以认为就是我们常说的“动态数组”。

#### add函数

当我们在ArrayList中增加元素的时候，会使用`add`函数。他会将元素放到末尾。具体实现如下：

```
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

我们可以看到他的实现其实最核心的内容就是`ensureCapacityInternal`。这个函数其实就是**自动扩容机制的核心**。我们依次来看一下他的具体实现

```java
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // 扩展为原来的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 如果扩为1.5倍还不满足需求，直接扩为需求值
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

当增加数据的时候，如果ArrayList的大小已经不满足需求时，那么就将数组变为原长度的1.5倍，之后的操作就是把老的数组拷到新的数组里面。例如，默认的数组大小是10，也就是说当我们`add`10个元素之后，再进行一次add时，就会发生自动扩容，数组长度由10变为了15具体情况如下所示：

#### set和get函数

Array的set和get函数就比较简单了，先做index检查，然后执行赋值或访问操作：

```java
public E set(int index, E element) {
    rangeCheck(index);
    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
public E get(int index) {
    rangeCheck(index);
    return elementData(index);
}
```

#### remove函数

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 把后面的往前移
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    // 把最后的置null
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}
```



### 8. LinkedList

以双向链表实现。链表无容量限制，但双向链表本身使用了更多空间，也需要额外的链表指针操作。

按下标访问元素—get(i)/set(i,e) 要遍历链表将指针移动到位(如果i>数组大小的一半，会从末尾移起)。

插入、删除元素时修改前后节点的指针即可，但还是要遍历部分链表的指针才能移动到下标所指的位置，只有在链表两头的操作—add()，addFirst()，removeLast()或用iterator()上的remove()能省掉指针的移动。

LinkedList是一个简单的数据结构，与ArrayList不同的是，他是基于链表实现的。

> Doubly-linked list implementation of the List and Deque interfaces. Implements all optional list operations, and permits all elements (including null).

#### set和get函数

```java
public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```

这两个函数都调用了`node`函数，该函数会以O(n/2)的性能去获取一个节点，具体实现如下所示：

```java
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
```

就是判断index是在前半区间还是后半区间，如果在前半区间就从head搜索，而在后半区间就从tail搜索。而不是一直从头到尾的搜索。如此设计，将节点访问的复杂度由O(n)变为O(n/2)。



### 9. HashMap

在官方文档中是这样描述HashMap的：

> Hash table based**implementation of the Map interface**. This implementation provides all of the optional map operations, and permits null values and the null key. (The HashMap class is roughly equivalent to Hashtable, except that it is**unsynchronized**and**permits nulls**.) This class makes no guarantees as to the order of the map; in particular, it does not guarantee that the order will remain constant over time.

几个关键的信息：基于Map接口实现、允许null键/值、非同步、不保证有序(比如插入的顺序)、也不保证序不随时间变化。

#### 两个重要的参数

在HashMap中有两个很重要的参数，容量(Capacity)和负载因子(Load factor)

> - **Initial capacity** The capacity is **the number of buckets** in the hash table, The initial capacity is simply the capacity at the time the hash table is created.
> - **Load factor** The load factor is **a measure of how full the hash table is allowed to get** before its capacity is automatically increased.

简单的说，Capacity就是bucket的大小，Load factor就是bucket填满程度的最大比例。如果对迭代性能要求很高的话，不要把`capacity`设置过大，也不要把`load factor`设置过小。

> **当bucket中的entries的数目大于`capacity*load factor`时就需要调整bucket的大小为当前的2倍。**

#### put函数的实现

大致的思路为：

1. 对key的hashCode()做hash，然后再计算index;
2. 如果没碰撞直接放到bucket里；
3. 如果碰撞了，以链表的形式存在buckets后；
4. 如果碰撞导致链表过长(大于等于`TREEIFY_THRESHOLD`)，就把**链表**转换成**红黑树**；
5. 如果节点已经存在就替换old value(保证key的唯一性)
6. 如果bucket满了(超过`load factor*current capacity`)，就要resize。

具体代码的实现如下：

```java
public V put(K key, V value) {
    // 对key的hashCode()做hash
    return putVal(hash(key), key, value, false, true);
}
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // tab为空则创建
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 计算index，并对null做处理
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        // 节点存在
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 该链为树
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 该链为链表
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 写入
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 超过load factor*current capacity，resize
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

#### get函数的实现

在理解了put之后，get就很简单了。大致思路如下：

1. bucket里的第一个节点，直接命中；

2. 如果有冲突，则通过key.equals(k)去查找对应的entry

   若为树，则在树中通过key.equals(k)查找，O(logn)；

   若为链表，则在链表中通过key.equals(k)查找，O(n)。

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 直接命中
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 未命中
        if ((e = first.next) != null) {
            // 在树中get
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 在链表中get
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

#### hash函数的实现

在get和put的过程中，计算下标时，先对hashCode进行hash操作，然后再通过hash值进一步计算下标，如下图所示：

[![hash](https://cloud.githubusercontent.com/assets/1736354/6957712/293b52fc-d932-11e4-854d-cb47be67949a.png)](https://cloud.githubusercontent.com/assets/1736354/6957712/293b52fc-d932-11e4-854d-cb47be67949a.png)

在对hashCode()计算hash时具体实现是这样的：

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

可以看到这个函数大概的作用就是：高16bit不变，低16bit和高16bit做了一个异或。其中代码注释是这样写的：

> Computes key.hashCode() and spreads (XORs) higher bits of hash to lower. Because the table uses power-of-two masking, sets of hashes that vary only in bits above the current mask will always collide. (Among known examples are sets of Float keys holding consecutive whole numbers in small tables.) So we apply a transform that spreads the impact of higher bits downward. There is a tradeoff between**speed, utility, and quality**of bit-spreading. Because many common sets of hashes are already**reasonably distributed**(so don’t benefit from spreading), and because**we use trees to handle large sets of collisions in bins**, we just XOR some shifted bits in the cheapest possible way to reduce systematic lossage, as well as to incorporate impact of the highest bits that would otherwise never be used in index calculations because of table bounds.

在设计hash函数时，因为目前的table长度n为2的幂，而计算下标的时候，是这样实现的(使用`&`位操作，而非`%`求余)：

```java
(n - 1) & hash
```

设计者认为这方法很容易发生碰撞。为什么这么说呢？不妨思考一下，在n - 1为15(0x1111)时，其实散列真正生效的只是低4bit的有效位，当然容易碰撞了。

因此，设计者想了一个顾全大局的方法(综合考虑了速度、作用、质量)，就是把高16bit和低16bit异或了一下。设计者还解释到因为现在大多数的hashCode的分布已经很不错了，就算是发生了碰撞也用`O(logn)`的tree去做了。仅仅异或一下，既减少了系统的开销，也不会造成的因为高位没有参与下标的计算(table长度比较小时)，从而引起的碰撞。

如果还是产生了频繁的碰撞，会发生什么问题呢？作者注释说，他们使用树来处理频繁的碰撞(we use trees to handle large sets of collisions in bins)，在[JEP-180](http://openjdk.java.net/jeps/180)中，描述了这个问题：

> Improve the performance of java.util.HashMap under high hash-collision conditions by**using balanced trees rather than linked lists to store map entries**. Implement the same improvement in the LinkedHashMap class.

之前已经提过，在获取HashMap的元素时，基本分两步：

1. 首先根据hashCode()做hash，然后确定bucket的index；
2. 如果bucket的节点的key不是我们需要的，则通过keys.equals()在链中找。

在Java 8之前的实现中是用链表解决冲突的，在产生碰撞的情况下，进行get时，两步的时间复杂度是O(1)+O(n)。因此，当碰撞很厉害的时候n很大，O(n)的速度显然是影响速度的。

因此在Java 8中，利用红黑树替换链表，这样复杂度就变成了O(1)+O(logn)了，这样在n很大的时候，能够比较理想的解决这个问题。

#### RESIZE的实现

当put时，如果发现目前的bucket占用程度已经超过了Load Factor所希望的比例，那么就会发生resize。在resize的过程，简单的说就是把bucket扩充为2倍，之后重新计算index，把节点再放到新的bucket中。resize的注释是这样描述的：

> Initializes or doubles table size. If null, allocates in accord with initial capacity target held in field threshold. Otherwise, because we are using power-of-two expansion, the elements from each bin must either**stay at same index**, or**move with a power of two offset**in the new table.

大致意思就是说，当超过限制的时候会resize，然而又因为我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。

怎么理解呢？例如我们从16扩展为32时，具体的变化如下所示：
[![rehash](https://cloud.githubusercontent.com/assets/1736354/6958256/ceb6e6ac-d93b-11e4-98e7-c5a5a07da8c4.png)](https://cloud.githubusercontent.com/assets/1736354/6958256/ceb6e6ac-d93b-11e4-98e7-c5a5a07da8c4.png)

因此元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：
[![resize](https://cloud.githubusercontent.com/assets/1736354/6958301/519be432-d93c-11e4-85bb-dff0a03af9d3.png)](https://cloud.githubusercontent.com/assets/1736354/6958301/519be432-d93c-11e4-85bb-dff0a03af9d3.png)

因此，我们在扩充HashMap的时候，不需要重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”。可以看看下图为16扩充为32的resize示意图：

[![resize16-32](https://cloud.githubusercontent.com/assets/1736354/6958677/d7acbad8-d941-11e4-9493-2c5e69d084c0.png)](https://cloud.githubusercontent.com/assets/1736354/6958677/d7acbad8-d941-11e4-9493-2c5e69d084c0.png)

这个设计确实非常的巧妙，既省去了重新计算hash值的时间，而且同时，由于新增的1bit是0还是1可以认为是随机的，因此resize的过程，均匀的把之前的冲突的节点分散到新的bucket了。

下面是代码的具体实现：

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 超过最大值就不再扩充了，就只好随你碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

#### 总结

**1. 什么时候会使用HashMap？他有什么特点？**
是基于Map接口的实现，存储键值对时，它可以接收null的键值，是非同步的，HashMap存储着Entry(hash, key, value, next)对象。

**2. 你知道HashMap的工作原理吗？**
通过hash的方法，通过put和get存储和获取对象。存储对象时，我们将K/V传给put方法时，它调用hashCode计算hash从而得到bucket位置，进一步存储，HashMap会根据当前bucket的占用情况自动调整容量(超过`Load Facotr`则resize为原来的2倍)。获取对象时，我们将K传给get，它调用hashCode计算hash从而得到bucket位置，并进一步调用equals()方法确定键值对。如果发生碰撞的时候，Hashmap通过链表将产生碰撞冲突的元素组织起来，在Java 8中，如果一个bucket中碰撞冲突的元素超过某个限制(默认是8)，则使用红黑树来替换链表，从而提高速度。

**3. 你知道get和put的原理吗？equals()和hashCode()的都有什么作用？**
通过对key的hashCode()进行hashing，并计算下标( `(n-1) & hash`)，从而获得buckets的位置。如果产生碰撞，则利用key.equals()方法去链表或树中去查找对应的节点

**4. 你知道hash的实现吗？为什么要这样实现？**
在Java 1.8的实现中，是通过hashCode()的高16位异或低16位实现的：`(h = k.hashCode()) ^ (h >>> 16)`，主要是从速度、功效、质量来考虑的，这么做可以在bucket的n比较小的时候，也能保证考虑到高低bit都参与到hash的计算中，同时不会有太大的开销。

**5. 如果HashMap的大小超过了负载因子(`load factor`)定义的容量，怎么办？**
如果超过了负载因子(默认**0.75**)，则会重新resize一个原来长度两倍的HashMap，并且重新调用hash方法。

### 10. LinkedHashMap

> **Hash table** and **linked list** implementation of the Map interface, with predictable iteration order. This implementation differs from HashMap in that it maintains a **doubly-linked list** running through all of its entries. This linked list defines the iteration ordering, which is normally the order in which keys were inserted into the map (**insertion-order**).

LinkedHashMap是Hash表和链表的实现，并且依靠着双向链表保证了迭代顺序是插入的顺序。

#### 三个重点实现的函数

在HashMap中提到了下面的定义：

```java
// Callbacks to allow LinkedHashMap post-actions
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
void afterNodeRemoval(Node<K,V> p) { }
```

LinkedHashMap继承于HashMap，因此也重新实现了这3个函数，顾名思义这三个函数的作用分别是：节点访问后、节点插入后、节点移除后做一些事情。

**afterNodeAccess函数**

```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    // 如果定义了accessOrder，那么就保证最近访问节点放到最后
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

就是说在进行put之后就算是对节点的访问了，那么这个时候就会更新链表，把最近访问的放到最后，保证链表。

**afterNodeInsertion函数**

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    // 如果定义了溢出规则，则执行相应的溢出
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```

如果用户定义了`removeEldestEntry`的规则，那么便可以执行相应的移除操作。

**afterNodeRemoval函数**

```java
void afterNodeRemoval(Node<K,V> e) { // unlink
    // 从链表中移除节点
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```

这个函数是在移除节点后调用的，就是将节点从双向链表中删除。

我们从上面3个函数看出来，基本上都是为了**保证双向链表中的节点次序或者双向链表容量**所做的一些额外的事情，目的就是保持双向链表中节点的顺序要从eldest到youngest。

#### put和get函数

`put`函数在LinkedHashMap中未重新实现，只是实现了`afterNodeAccess`和`afterNodeInsertion`两个回调函数。`get`函数则重新实现并加入了`afterNodeAccess`来保证访问顺序，下面是`get`函数的具体实现：

```java
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}
```

值得注意的是，在accessOrder模式下，只要执行get或者put等操作的时候，就会产生`structural modification`。官方文档是这么描述的：

> A structural modification is any operation that adds or deletes one or more mappings or, in the case of access-ordered linked hash maps, affects iteration order. In insertion-ordered linked hash maps, merely changing the value associated with a key that is already contained in the map is not a structural modification. **In access-ordered linked hash maps, merely querying the map with get is a structural modification**.

总之，LinkedHashMap不愧是HashMap的儿子，和老子太像了，当然，青出于蓝而胜于蓝，LinkedHashMap的其他的操作也基本上都是为了维护好那个具有访问顺序的双向链表。


### 11. TreeMap

> A **Red-Black tree** based NavigableMap implementation. The map is sorted according to the natural ordering of its keys, or by a Comparator provided at map creation time, depending on which constructor is used.
> This implementation provides guaranteed **log(n) time cost **for the containsKey, get, put and remove operations. Algorithms are adaptations of those in Cormen, Leiserson, and Rivest’s Introduction to Algorithms.

HashMap不保证数据有序，LinkedHashMap保证数据可以保持插入顺序，而如果我们希望Map可以**保持key的大小顺序**的时候，我们就需要利用TreeMap了。

使用红黑树的好处是能够使得树具有不错的平衡性，这样操作的速度就可以达到log(n)的水平了。具体红黑树的实现不在这里赘述，可以参考[数据结构之红黑树](http://dongxicheng.org/structure/red-black-tree/)、[wikipedia-红黑树](http://zh.wikipedia.org/wiki/红黑树)等的实现。

#### put函数

> Associates the specified value with the specified key in this map.If the map previously contained a mapping for the key, the old value is replaced.

如果存在的话，old value被替换；如果不存在的话，则新添一个节点，然后对做红黑树的平衡操作。

```java
public V put(K key, V value) {
    Entry<K,V> t = root;
    if (t == null) {
        compare(key, key); // type (and possibly null) check
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
        // 如果该节点存在，则替换值直接返回
    if (cpr != null) {
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    else {
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    // 如果该节点未存在，则新建
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    
    // 红黑树平衡调整
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}
```

#### get函数

get函数则相对来说比较简单，以log(n)的复杂度进行get。

```java
final Entry<K,V> getEntry(Object key) {
    // Offload comparator-based version for sake of performance
    if (comparator != null)
        return getEntryUsingComparator(key);
    if (key == null)
        throw new NullPointerException();
    @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
    Entry<K,V> p = root;
        // 按照二叉树搜索的方式进行搜索，搜到返回
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            p = p.left;
        else if (cmp > 0)
            p = p.right;
        else
            return p;
    }
    return null;
}
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p==null ? null : p.value);
}
```

#### successor后继

 TreeMap是如何保证其迭代输出是有序的呢？其实从宏观上来讲，就相当于树的中序遍历(LDR)。我们先看一下迭代输出的步骤

```java
for(Entry<Integer, String> entry : tmap.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

根据[The enhanced for statement](http://docs.oracle.com/javase/specs/jls/se8/html/jls-14.html#jls-14.14.2)，for语句会做如下转换为：

```java
for(Iterator<Map.Entry<String, String>> it = tmap.entrySet().iterator() ; tmap.hasNext(); ) {
    Entry<Integer, String> entry = it.next();
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

在**it.next()的调用中会使用nextEntry**调用`successor`这个是过的后继的重点，具体实现如下：

```java
static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
    if (t == null)
        return null;
    else if (t.right != null) {
        // 有右子树的节点，后继节点就是右子树的“最左节点”
        // 因为“最左子树”是右子树的最小节点
        Entry<K,V> p = t.right;
        while (p.left != null)
            p = p.left;
        return p;
    } else {
        // 如果右子树为空，则寻找当前节点所在左子树的第一个祖先节点
        // 因为左子树找完了，根据LDR该D了
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        // 保证左子树
        while (p != null && ch == p.right) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}
```

怎么理解这个successor呢？只要记住，这个是中序遍历就好了，L-D-R。具体细节如下：

> **a. 空节点，没有后继**
> **b. 有右子树的节点，后继就是右子树的“最左节点”**
> **c. 无右子树的节点，后继就是该节点所在左子树的第一个祖先节点**

a.好理解，不过b, c，有点像绕口令啊，没关系，上图举个例子就懂了！

**有右子树的节点**，节点的下一个节点，肯定在右子树中，而右子树中“最左”的那个节点则是右子树中最小的一个，那么当然是**右子树的“最左节点”**，就好像下图所示：
[![treemap1](https://cloud.githubusercontent.com/assets/1736354/7045283/652d00c4-de2f-11e4-8475-1a2f46afc380.png)](https://cloud.githubusercontent.com/assets/1736354/7045283/652d00c4-de2f-11e4-8475-1a2f46afc380.png)

**无右子树的节点**，先找到这个节点所在的左子树(右图)，那么这个节点所在的左子树的父节点(绿色节点)，就是下一个节点。
[![treemap2](https://cloud.githubusercontent.com/assets/1736354/7045284/68279686-de2f-11e4-8310-c9f76b3f52ab.png)](https://cloud.githubusercontent.com/assets/1736354/7045284/68279686-de2f-11e4-8310-c9f76b3f52ab.png)



## 二、Java泛型

### 1. 简介

#### 引入泛型的目的

了解引入泛型的动机，就先从语法糖开始了解。

**语法糖**（Syntactic Sugar），也称糖衣语法，指在计算机语言中添加的某种语法，这种语法对语言的功能并没有影响，但是更方便程序员使用。Java中最常用的语法糖主要有泛型、变长参数、条件编译、自动拆装箱、内部类等。虚拟机并不支持这些语法，它们在编译阶段就被还原回了简单的基础语法结构，这个过程成为解语法糖。

**泛型的目的：** Java 泛型就是把一种语法糖，通过泛型使得在编译阶段完成一些类型转换的工作，避免在运行时强制类型转换而出现`ClassCastException`，即类型转换异常。

##### 泛型初探

JDK 1.5 时才增加了泛型，并在很大程度上都是方便集合的使用，使其能够记住其元素的数据类型。

在泛型（Generic type或Generics）出现之前，是这么写代码的：

```java
public static void main(String[] args)
{
    List list = new ArrayList();
    list.add("123");
    list.add("456");

    System.out.println((String)list.get(0));
}
```

当然这是完全允许的，因为List里面的内容是Object类型的，自然任何对象类型都可以放入、都可以取出，但是这么写会有两个问题：

1、当一个对象放入集合时，集合不会记住此对象的类型，当再次从集合中取出此对象时，该对象的编译类型变成了Object。

2、运行时需要人为地强制转换类型到具体目标，实际的程序一个不小心就会出现java.lang.ClassCastException。

所以，泛型出现之后，上面的代码就改成了大家都熟知的写法：

```java
public static void main(String[] args)
{
    List<String> list = new ArrayList<String>();
    list.add("123");
    list.add("456");

    System.out.println(list.get(0));
}
```

这就是泛型。泛型是对Java语言类型系统的一种扩展，有点类似于C++的模板，可以把类型参数看作是使用参数化类型时指定的类型的一个占位符。引入泛型，是对Java语言一个较大的功能增强，带来了很多的好处。

##### 泛型的好处

- 类型安全。类型错误现在在编译期间就被捕获到了，而不是在运行时当作java.lang.ClassCastException展示出来，将类型检查从运行时挪到编译时有助于开发者更容易找到错误，并提高程序的可靠性。

- 消除了代码中许多的强制类型转换，增强了代码的可读性。

- 为较大的优化带来了可能。

### 2. 泛型的使用

#### 泛型类和泛型接口

下面是JDK 1.5 以后，List接口，以及ArrayList类的代码片段。

```java
//定义接口时指定了一个类型形参，该形参名为E
public interface List<E> extends Collection<E> {
   //在该接口里，E可以作为类型使用
   public E get(int index) {}
   public void add(E e) {} 
}

//定义类时指定了一个类型形参，该形参名为E
public class ArrayList<E> extends AbstractList<E> implements List<E> {
   //在该类里，E可以作为类型使用
   public void set(E e) {
   .......................
   }
}
```

这就是**泛型的实质：允许在定义接口、类时声明类型形参，类型形参在整个接口、类体内可当成类型使用，几乎所有可使用普通类型的地方都可以使用这种类型形参。**

下面具体讲解泛型类的使用。泛型接口的使用与泛型类几乎相同，可以比对自行学习。

**泛型类** 

定义一个容器类，存放键值对key-value，键值对的类型不确定，可以使用泛型来定义，分别指定为K和V。

```java
public class Container<K, V> {

    private K key;
    private V value;

    public Container(K k, V v) {
        key = k;
        value = v;
    }

    public K getkey() {
        return key;
    }

    public V getValue() {
        return value;
    }

    public void setKey() {
        this.key = key;
    }

    public void setValue() {
        this.value = value;
    }

}
```

在使用Container类时，只需要指定K，V的具体类型即可，从而创建出逻辑上不同的Container实例，用来存放不同的数据类型。

```java
 public static void main(String[] args) {
     Container<String,String>  c1=new Container<String ,String>("name","hello");
     Container<String,Integer> c2=new Container<String,Integer>("age",22);
     Container<Double,Double>  c3=new Container<Double,Double>(1.1,1.3);
     System.out.println(c1.getKey() + " : " + c1.getValue());      
     System.out.println(c2.getKey() + " : " + c2.getValue());                                                               
     System.out.println(c3.getKey() + " : " + c3.getValue());
 }
```

在JDK 1.7 增加了泛型的“菱形”语法：**Java允许在构造器后不需要带完成的泛型信息，只要给出一对尖括号（<>）即可，Java可以推断尖括号里应该是什么泛型信息。**如下所示：

```java
Container<String,String> c1=new Container<>("name","hello");
Container<String,Integer> c2=new Container<>("age",22);
```

**泛型类派生子类**

当创建了带泛型声明的接口、父类之后，可以为该接口创建实现类，或者从该父类派生子类，需要注意：**使用这些接口、父类派生子类时不能再包含类型形参，需要传入具体的类型。**
错误的方式：

```java
public class A extends Container<K, V>{}
```

正确的方式：

```java
public class A extends Container<Integer, String>{}
```

也可以不指定具体的类型，如下：

```java
public class A extends Container{}
```

此时系统会把K,V形参当成Object类型处理。

#### 泛型的方法

前面在介绍泛型类和泛型接口中提到，可以在泛型类、泛型接口的方法中，把泛型中声明的类型形参当成普通类型使用。 如下面的方式：

```java
public class Container<K, V>
 {
    .......
    public K getkey() {
        return key;
    }
    public void setKey() {
        this.key = key;
    }
	.......
}
```

但在另外一些情况下，在类、接口中没有使用泛型时，定义方法时想定义类型形参，就会使用泛型方法。如下方式：

```java
public class Main{
  public static <T> void out(T t){
       System.out.println(t);
  }
  public static void main(String[] args){
       out("hansheng");
       out(123);
  }
}
```

**所谓泛型方法，就是在声明方法时定义一个或多个类型形参。** 泛型方法的用法格式如下：

```
修饰符<T, S> 返回值类型 方法名（形参列表）｛
   方法体
｝
```

**注意：** 方法声明中定义的形参只能在该方法里使用，而接口、类声明中定义的类型形参则可以在整个接口、类中使用。

```java
class Demo{  
  public <T> T fun(T t){   // 可以接收任意类型的数据  
   return t ;     // 直接把参数返回  
  }  
};  
public class GenericsDemo26{  
  public static void main(String args[]){  
    Demo d = new Demo() ; // 实例化Demo对象  
    String str = d.fun("汤姆") ; // 传递字符串  
    int i = d.fun(30) ;  // 传递数字，自动装箱  
    System.out.println(str) ; // 输出内容  
    System.out.println(i) ;  // 输出内容  
  }  
};
```

当调用`fun()`方法时，根据传入的实际对象，编译器就会判断出类型形参T所代表的实际类型。

#### 泛型构造器

正如泛型方法允许在方法签名中声明类型形参一样，Java也允许在构造器签名中声明类型形参，这样就产生了所谓的泛型构造器。
和使用普通泛型方法一样没区别，一种是显式指定泛型参数，另一种是隐式推断，如果是显式指定则以显式指定的类型参数为准，如果传入的参数的类型和指定的类型实参不符，将会编译报错。

```java
public class Person {
    public <T> Person(T t) {
        System.out.println(t);
    }

}
public static void main(String[] args){
        //隐式
        new Person(22);
        //显示
        new<String> Person("hello");
}
```

这里唯一需要特殊注明的就是，如果构造器是泛型构造器，同时该类也是一个泛型类的情况下应该如何使用泛型构造器：

因为泛型构造器可以显式指定自己的类型参数（需要用到菱形，放在构造器之前），而泛型类自己的类型实参也需要指定（菱形放在构造器之后），这就同时出现了两个菱形了，这就会有一些小问题，具体用法再这里总结一下。

以下面这个例子为代表

```java
public class Person<E> {
    public <T> Person(T t) {
        System.out.println(t);
    }

}
```

这种用法：`Person<String> a = new <Integer>Person<>(15);`这种语法不允许，会直接编译报错！

正确用法`Person<String> a = new <Integer>Person<String>(15);`

### 3. 类型通配符

顾名思义就是匹配任意类型的类型实参。

类型通配符是一个问号（？)，将一个问号作为类型实参传给List集合，写作：`List<?>`（意思是元素类型未知的List）。这个问号（？）被成为通配符，它的元素类型可以匹配任何类型。

```java
public void test(List<?> c){
  for(int i =0;i<c.size();i++){
    System.out.println(c.get(i));
  }
}
```

现在可以传入任何类型的List来调用test()方法，程序依然可以访问集合c中的元素，其类型是Object。

```java
List<?> c = new ArrayList<String>();
//编译器报错
c.add(new Object());
```

但是并不能把元素加入到其中。因为程序无法确定c集合中元素的类型，所以不能向其添加对象。

下面就该引入带限通配符，来确定集合元素中的类型。

#### 带限通配符

简单来讲，使用通配符的目的是来限制泛型的类型参数的类型，使其满足某种条件，固定为某些类。

主要分为两类即：**上限通配符**和**下限通配符**。

**上限通配符**

如果想限制使用泛型类别时，只能用某个特定类型或者是其子类型才能实例化该类型时，可以在定义类型时，**使用extends关键字指定这个类型必须是继承某个类，或者实现某个接口，也可以是这个类或接口本身。**

```java
它表示集合中的所有元素都是Shape类型或者其子类
List<? extends Shape>
```

这就是所谓的上限通配符，使用关键字extends来实现，实例化时，指定类型实参只能是extends后类型的子类或其本身。
例如：

```java
//Circle是其子类
List<? extends Shape> list = new ArrayList<Circle>();
```

这样就确定集合中元素的类型，虽然不确定具体的类型，但最起码知道其父类。然后进行其他操作。

**下限通配符**

如果想限制使用泛型类别时，只能用某个特定类型或者是其父类型才能实例化该类型时，可以在定义类型时，**使用super关键字指定这个类型必须是是某个类的父类，或者是某个接口的父接口，也可以是这个类或接口本身。**

```
它表示集合中的所有元素都是Circle类型或者其父类
List <? super Circle>
```

这就是所谓的下限通配符，使用关键字super来实现，实例化时，指定类型实参只能是extends后类型的子类或其本身。
例如：

```java
//Shape是其父类
List<? super Circle> list = new ArrayList<Shape>();
```

### 4. 类型擦除

```java
Class c1=new ArrayList<Integer>().getClass();
Class c2=new ArrayList<String>().getClass();
System.out.println(c1==c2);
```

程序输出：

> true。

这是因为不管为泛型的类型形参传入哪一种类型实参，对于Java来说，它们依然被当成同一类处理，在内存中也只占用一块内存空间。从Java泛型这一概念提出的目的来看，其只是作用于代码编译阶段，在编译过程中，对于正确检验泛型结果后，会将泛型的相关信息擦出，也就是说，成功编译过后的class文件中是不包含任何泛型信息的。泛型信息不会进入到运行时阶段。

**在静态方法、静态初始化块或者静态变量的声明和初始化中不允许使用类型形参。由于系统中并不会真正生成泛型类，所以instanceof运算符后不能使用泛型类。**



## 三、Java反射



## 四、Java注解
## 五、Java IO
## 六、RandomAccessFile
## 七、Java NIO
## 八、Java异常详解
## 九、Java抽象类和接口的区别
## 十、Java深拷贝和浅拷贝
## 十一、Java transient关键字

### 1. transient的作用及使用方法

我们都知道一个对象只要实现了Serilizable接口，这个对象就可以被序列化，java的这种序列化模式为开发者提供了很多便利，我们可以不必关系具体序列化的过程，只要这个类实现了Serilizable接口，这个类的所有属性和方法都会自动序列化。

然而在实际开发过程中，我们常常会遇到这样的问题，这个类的有些属性需要序列化，而其他属性不需要被序列化，打个比方，如果一个用户有一些敏感信息（如密码，银行卡号等），为了安全起见，不希望在网络操作（主要涉及到序列化操作，本地序列化缓存也适用）中被传输，这些信息对应的变量就可以加上transient关键字。换句话说，这个字段的生命周期仅存于调用者的内存中而不会写到磁盘里持久化。

总之，java 的transient关键字为我们提供了便利，你只需要实现Serilizable接口，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。

示例code如下：

```java
/**
 * @description 使用transient关键字不序列化某个变量
 *        注意读取的时候，读取数据的顺序一定要和存放数据的顺序保持一致
 */
public class TransientTest {
    
    public static void main(String[] args) {
        
        User user = new User();
        user.setUsername("Alexia");
        user.setPasswd("123456");
        
        System.out.println("read before Serializable: ");
        System.out.println("username: " + user.getUsername());
        System.err.println("password: " + user.getPasswd());
        
        try {
            ObjectOutputStream os = new ObjectOutputStream(
                    new FileOutputStream("C:/user.txt"));
            os.writeObject(user); // 将User对象写进文件
            os.flush();
            os.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            ObjectInputStream is = new ObjectInputStream(new FileInputStream("C:/user.txt"));
            user = (User) is.readObject(); // 从流中读取User的数据
            is.close();
            
            System.out.println("\nread after Serializable: ");
            System.out.println("username: " + user.getUsername());
            System.err.println("password: " + user.getPasswd());
            
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

class User implements Serializable {
    private static final long serialVersionUID = 8294180014912103005L;  
    
    private String username;
    private transient String passwd;
    
    public String getUsername() {
        return username;
    }
    
    public void setUsername(String username) {
        this.username = username;
    }
    
    public String getPasswd() {
        return passwd;
    }
    
    public void setPasswd(String passwd) {
        this.passwd = passwd;
    }

}
```

输出为：

```
read before Serializable: 
username: Alexia
password: 123456

read after Serializable: 
username: Alexia
password: null
```

### 2. transient使用小结

1）一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。

2）transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现Serializable接口。

3）被transient关键字修饰的变量不再能被序列化，一个**静态变量**不管是否被transient修饰，均不能被序列化。

第三点可能有些人很迷惑，因为发现在User类中的username字段前加上static关键字后，程序运行结果依然不变，即static类型的username也读出来为“Alexia”了，这不与第三点说的矛盾吗？实际上是这样的：第三点确实没错（一个静态变量不管是否被transient修饰，均不能被序列化），反序列化后类中static型变量username的值为当前JVM中对应static变量的值，这个值是JVM中的不是反序列化得出的，不相信？好吧，下面我来证明：

```java
/**
 * @description 使用transient关键字不序列化某个变量
 *        注意读取的时候，读取数据的顺序一定要和存放数据的顺序保持一致
 */
public class TransientTest {
    
    public static void main(String[] args) {
        
        User user = new User();
        user.setUsername("Alexia");
        user.setPasswd("123456");
        
        System.out.println("read before Serializable: ");
        System.out.println("username: " + user.getUsername());
        System.err.println("password: " + user.getPasswd());
        
        try {
            ObjectOutputStream os = new ObjectOutputStream(
                    new FileOutputStream("C:/user.txt"));
            os.writeObject(user); // 将User对象写进文件
            os.flush();
            os.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        try {
            // 在反序列化之前改变username的值
            User.username = "jmwang";
            
            ObjectInputStream is = new ObjectInputStream(new FileInputStream("C:/user.txt"));
            user = (User) is.readObject(); // 从流中读取User的数据
            is.close();
            
            System.out.println("\nread after Serializable: ");
            System.out.println("username: " + user.getUsername());
            System.err.println("password: " + user.getPasswd());
            
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

class User implements Serializable {
    private static final long serialVersionUID = 8294180014912103005L;  
    
    public static String username;
    private transient String passwd;
    
    public String getUsername() {
        return username;
    }
    
    public void setUsername(String username) {
        this.username = username;
    }
    
    public String getPasswd() {
        return passwd;
    }
    
    public void setPasswd(String passwd) {
        this.passwd = passwd;
    }

}
```

输出为：

```
read before Serializable: 
username: Alexia
password: 123456

read after Serializable: 
username: jmwang
password: null
```



### 3. transient使用细节

被transient关键字修饰的变量真的不能被序列化吗？

思考下面的例子：

```java
/**
 * @descripiton Externalizable接口的使用
 */
public class ExternalizableTest implements Externalizable {

    private transient String content = "是的，我将会被序列化，不管我是否被transient关键字修饰";

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(content);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException,
            ClassNotFoundException {
        content = (String) in.readObject();
    }

    public static void main(String[] args) throws Exception {
        
        ExternalizableTest et = new ExternalizableTest();
        ObjectOutput out = new ObjectOutputStream(new FileOutputStream(
                new File("test")));
        out.writeObject(et);

        ObjectInput in = new ObjectInputStream(new FileInputStream(new File("test")));
        et = (ExternalizableTest) in.readObject();
        System.out.println(et.content);

        out.close();
        in.close();
    }
}
```

content变量会被序列化吗？好吧，我把答案都输出来了，是的，运行结果就是：

```
是的，我将会被序列化，不管我是否被transient关键字修饰
```

这是为什么呢，不是说类的变量被transient关键字修饰以后将不能序列化了吗？

我们知道在Java中，对象的序列化可以通过实现两种接口来实现，若实现的是Serializable接口，则所有的序列化将会自动进行，若实现的是Externalizable接口，则没有任何东西可以自动序列化，需要在writeExternal方法中进行手工指定所要序列化的变量，这与是否被transient修饰无关。因此第二个例子输出的是变量content初始化的内容，而不是null。

## 十二、Java finally与return执行顺序

网上有很多人探讨Java中异常捕获机制try...catch...finally块中的finally语句是不是一定会被执行？很多人都说不是，回答是正确的，

**至少有两种情况下finally语句是不会被执行的：**

**（1）try语句没有被执行到，如在try语句之前就返回了，这样finally语句就不会执行，这也说明了finally语句被执行的必要而非充分条件是：相应的try语句一定被执行到。**

**（2）在try块中有`System.exit(0);`这样的语句，`System.exit(0);`是终止Java虚拟机JVM的，连JVM都停止了，所有都结束了，当然finally语句也不会被执行到。**

当然还有很多人探讨finally语句的执行与return的关系，颇为让人迷惑，不知道finally语句是在try的return之前执行还是之后执行？我也是一头雾水，我觉得他们的说法都不正确，我觉得应该是：**finally语句是在try的return语句执行之后，return返回之前执行**。这样的说法有点矛盾，也许是我表述不太清楚，下面我给出自己试验的一些结果和示例进行佐证，有什么问题欢迎大家提出来。

**1. finally语句在return语句执行之后return返回之前执行的。**

```java
public class FinallyTest1 {

    public static void main(String[] args) {        
        System.out.println(test1());
    }

    public static int test1() {
        int b = 20;
        try {
            System.out.println("try block");
            return b += 80; 
        }
        catch (Exception e) {
            System.out.println("catch block");
        }
        finally {
            System.out.println("finally block");
            if (b > 25) {
                System.out.println("b>25, b = " + b);
            }
        }
        return b;
    }
    
}
```

运行结果是：

```
try block
finally block
b>25, b = 100
100
```

说明return语句已经执行了再去执行finally语句，不过并没有直接返回，而是等finally语句执行完了再返回结果。

如果觉得这个例子还不足以说明这个情况的话，下面再加个例子加强证明结论：

```java
public class FinallyTest1 {

    public static void main(String[] args) {
        
        System.out.println(test11());
    }
    
    public static String test11() {
        try {
            System.out.println("try block");

           return test12();
      } finally {
           System.out.println("finally block");
       }
  }

  public static String test12() {
       System.out.println("return statement");

       return "after return";
   }
    
}
```

运行结果为：

```
try block
return statement
finally block
after return
```

说明try中的return语句先执行了但并没有立即返回，等到finally执行结束后再返回。

这里大家可能会想：如果finally里也有return语句，那么是不是就直接返回了，try中的return就不能返回了？看下面。

**2. finally块中的return语句会覆盖try块中的return返回。**

```java
public class FinallyTest2 {

    public static void main(String[] args) {
        System.out.println(test2());
    }

    public static int test2() {
        int b = 20;
        try {
            System.out.println("try block");
            return b += 80;
        } catch (Exception e) {
            System.out.println("catch block");
        } finally {
            System.out.println("finally block");
            if (b > 25) {
                System.out.println("b>25, b = " + b);
            }
            return 200;
        }
        // return b;
    }

}
```

运行结果是：

```
try block
finally block
b>25, b = 100
200
```

这说明finally里的return直接返回了，就不管try中是否还有返回语句，这里还有个小细节需要注意，finally里加上return过后，finally外面的return b就变成不可到达语句了，也就是永远不能被执行到，**所以需要注释掉否则编译器报错**。

这里大家可能又想：如果finally里没有return语句，但修改了b的值，那么try中return返回的是修改后的值还是原值？看下面。

**3. 如果finally语句中没有return语句覆盖返回值，那么原来的返回值可能因为finally里的修改而改变也可能不变。**

**测试用例1：**

```java
public class FinallyTest3 {

    public static void main(String[] args) {

        System.out.println(test3());
    }

    public static int test3() {
        int b = 20;

        try {
            System.out.println("try block");

            return b += 80;
        } catch (Exception e) {

            System.out.println("catch block");
        } finally {

            System.out.println("finally block");

            if (b > 25) {
                System.out.println("b>25, b = " + b);
            }

            b = 150;
        }

        return 2000;
    }

}
```

运行结果是：

```
try block
finally block
b>25, b = 100
100
```

**测试用例2：**

```
import java.util.*;

public class FinallyTest6
{
    public static void main(String[] args) {
        System.out.println(getMap().get("KEY").toString());
    }
     
    public static Map<String, String> getMap() {
        Map<String, String> map = new HashMap<String, String>();
        map.put("KEY", "INIT");
         
        try {
            map.put("KEY", "TRY");
            return map;
        }
        catch (Exception e) {
            map.put("KEY", "CATCH");
        }
        finally {
            map.put("KEY", "FINALLY");
            map = null;
        }
         
        return map;
    }
}
```

运行结果是：

```
FINALLY
```

为什么测试用例1中finally里的b = 150;并没有起到作用而测试用例2中finally的map.put("KEY", "FINALLY");起了作用而map = null;却没起作用呢？这就是Java到底是传值还是传址的问题了，具体请看[精选30道Java笔试题解答](http://www.cnblogs.com/lanxuezaipiao/p/3371224.html)，里面有详细的解答，简单来说就是：Java中只有传值没有传址，这也是为什么map = null这句不起作用。这同时也说明了返回语句是try中的return语句而不是 finally外面的return b;这句，不相信的话可以试下，将return b;改为return 294，对原来的结果没有一点影响。

这里大家可能又要想：是不是每次返回的一定是try中的return语句呢？那么finally外的return b不是一点作用没吗？请看下面。

**4. try块里的return语句在异常的情况下不会被执行，这样具体返回哪个看情况。**

```java
public class FinallyTest4 {

    public static void main(String[] args) {
        System.out.println(test4());
    }

    public static int test4() {
        int b = 20;
        try {
            System.out.println("try block");
            b = b / 0;
            return b += 80;
        } catch (Exception e) {
            b += 15;
            System.out.println("catch block");
        } finally {
            System.out.println("finally block");
            if (b > 25) {
                System.out.println("b>25, b = " + b);
            }
            b += 50;
        }
        return b;
    }

}
```

运行结果是：

```
try block
catch block
finally block
b>25, b = 35
85
```

这里因 为在return之前发生了除0异常，所以try中的return不会被执行到，而是接着执行捕获异常的catch 语句和最终的finally语句，此时两者对b的修改都影响了最终的返回值，这时return b;就起到作用了。当然如果你这里将return b改为return 300什么的，最后返回的就是300，这毋庸置疑。

这里大家可能又有疑问：如果catch中有return语句呢？当然只有在异常的情况下才有可能会执行，那么是在finally之前就返回吗？看下面。

**5. 当发生异常后，catch中的return执行情况与未发生异常时try中return的执行情况完全一样。**

```java
public class FinallyTest5 {

    public static void main(String[] args) {
        System.out.println(test5());
    }

    public static int test5() {
        int b = 20;
        try {
            System.out.println("try block");
            b = b /0;
            return b += 80;
        } catch (Exception e) {
            System.out.println("catch block");
            return b += 15;
        } finally {
            System.out.println("finally block");
            if (b > 25) {
                System.out.println("b>25, b = " + b);
            }
            b += 50;
        }
        //return b;
    }
}
```

运行结果如下：

```
try block
catch block
finally block
b>25, b = 35
35
```

说明了发生异常后，catch中的return语句先执行，确定了返回值后再去执行finally块，执行完了catch再返回，finally里对b的改变对返回值无影响，原因同前面一样，也就是说情况与try中的return语句执行完全一样。

**最后总结：finally块的语句在try或catch中的return语句执行之后返回之前执行，且finally里的修改语句可能影响也可能不影响try或catch中 return已经确定的返回值（根据返回值类型判断），若finally里也有return语句则覆盖try或catch中的return语句直接返回。**



## 十三、Java 8 新特性