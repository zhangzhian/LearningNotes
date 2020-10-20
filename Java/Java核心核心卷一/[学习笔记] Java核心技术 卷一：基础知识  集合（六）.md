# [学习笔记] Java核心技术 卷一：基础知识  集合（六）

---

 - iterator方法用于返回一个实现了 Iterator 接口的对象。可以使用这个迭代器对象依次访
问集合中的元素。
 - Java库中的具体集合
  ![Java库中的具体集合](http://img.blog.csdn.net/20171023093101685?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 - 集合框架中的类
![集合框架中的类](http://img.blog.csdn.net/20171023093901418?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpZHVfMzIyMzc3MTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 - 在 Java 程序设计语言中，所有链表实际上都是双向链接的(doubly linked)—即每个结点还存放着指向前驱结点的引用
 - 树集是一个有序集合( sorted collection)  可以以任意顺序将元素插入到集合中。在对集合进行遍历时，每个值将自动地按照排序后的顺序呈现。将一个元素添加到树中要比添加到散列表中慢
 - 要使用树集，必须能够比较元素。这些元素必须实现Comparable接口或者构造集时必须提供一个 Comparator 
 - 在 Java SE 6 中引人了 Deque 接口， 并由 ArrayDeque 和 LinkedList 类实现。这两个类都提供了双端队列，而且在必要时可以增加队列的长度。
 - 优先级队列（priority queue) 中的元素可以按照任意的顺序插人，却总是按照排序的顺序进行检索。
 - 有 3 种视图： 键集、值集合（不是一个集） 以及键 / 值对集。
 - `Set<K> keySet()`
 - `Collection<V> values0`
 - `Set<Map.Entry<K, V» entrySetO`
 -   WeakHashMap 使用弱引用 （ weak references)保存键。WeakReference对象将引用保存到另外一个对象中， 在这里，就是散列键。
 - EmimSet 是一个枚举类型元素集的高效实现。 由于枚举类型只有有限个实例， 所以EnumSet 内部用位序列实现。如果对应的值在集中， 则相应的位被置为 1。
 - 类 IdentityHashMap 中， 键的散列值是用 System.identityHashCode 方法计算的。 这是 Object.hashCode 方法根据对象的内存地址来计算散列码时所使用的方式。 而且， 在对两个对象进行比较时， IdentityHashMap 类使用 ==, 而不使用 equals。
 - Arrays 类的静态方法 asList 将返回一个包装了普通 Java 数组的 List 包装器。它是一个视图对象， 带有访问底层数组的 get 和 set方法。改变数组大小的所有方法（add 和 remove 方法）都会抛出异常。
 - Col1ections.nCopies(n, anObject) 将返回一个实现了 List 接口的不可修改的对象， 并给人一种包含n个元素， 每个元素都像是一个 anObject 的错觉。
 - Collections.singleton(anObject)
则将返回一个视图对象。这个对象实现了 Set 接口（与产生 List 的 ncopies 方法不同） 。返回的对象实现了一个不可修改的单元素集， 而不需要付出建立数据结构的开销。
 - 类库的设计者使用视图机制来确保常规集合的线程安全， 而不是实现线程安全的集合
类。例如， Collections 类的静态 synchronizedMap方法可以将任何一个映射表转换成具有同步访问方法的 Map
 - Collections 类有一个算法 shuffle, 其功能与排序刚好相反，即随机地混排列表中元素的顺序。
 - Collections 类中的 sort 方法可以对实现了 List 接口的集合进行排序
 - Collections类的binarySearch方法实现了二分查找算法。 注意，集合必须是排好序的，否则算法将返回错误的答案。
 -  Collections类中包含的算法。前面介绍的查找集合中最大元素的示例就在其中。另外还包括：将一个列表中的元素复制到另外一个列表中； 用一个常量值填充容器；逆置一个列表的元素顺序。
 -  toArray方法返回的数组是一个 Object[] 数组， 不能改变它的类型。实际上， 必须使用toArray方法的一个变体形式，提供一个所需类型而且长度为0的数组。这样一来，返回的数组就会创建为相同的数组类型
 -  编写自己的算法，以集合作为参数的任何方法，应该尽可能地使用接口，而不要使用具体的实现。
 -  属性映射（property map) 是一个类型非常特殊的映射结构,它有下面 3 个特性：
  - 键与值都是字符串。
  - 表可以保存到一个文件中，也可以从文件中加载。
  - 使用一个默认的辅助表。
实现属性映射的 Java 平台类称为 Properties。
 -  BitSet 类用于存放一个位序列，BitSet 类提供了一个便于读取、设置或清除各个位的接口。使用这个接口可以避免屏蔽和其他麻烦的位操作。

    

