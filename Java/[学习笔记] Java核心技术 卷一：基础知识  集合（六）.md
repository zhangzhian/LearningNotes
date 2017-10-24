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
 - 
    
    

