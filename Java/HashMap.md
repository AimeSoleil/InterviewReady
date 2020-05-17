
> <转自：https://www.cnblogs.com/Young111/p/11519952.html?utm_source=gold_browser_extension>

#### HashMap的数据结构

> 哈希表结构，数组 + 链表的实现，可以结合数组和链表的优点。当链表长度超过8时，链表转换为红黑树。

#### HashMap的工作原理

HashMap 底层是 hash 数组和单向链表实现，数组中的每个元素都是链表，由 Node 内部类（实现 Map.Entry接口）实现，HashMap 通过 put & get 方法存储和获取。

**存储对象时**，将 K/V 键值传给 put() 方法：
①、调用 hash(K) 方法计算 K 的 hash 值，然后结合数组长度，计算得数组下标；
②、调整数组大小（当容器中的元素个数大于 capacity * loadfactor 时，容器会进行扩容resize 为 2n）；
③、i.如果 K 的 hash 值在 HashMap 中不存在，则执行插入，若存在，则发生碰撞；
   ii.如果 K 的 hash 值在 HashMap 中存在，且它们两者 equals 返回 true，则更新键值对；
   iii. 如果 K 的 hash 值在 HashMap 中存在，且它们两者 equals 返回 false，则插入链表的尾部（尾插法）或者红黑树中（树的添加方式）。
> （JDK 1.7 之前使用头插法、JDK 1.8 使用尾插法）（注意：当碰撞导致链表大于 TREEIFY_THRESHOLD = 8 时，就把链表转换成红黑树）

**获取对象时，**将 K 传给 get() 方法：①、调用 hash(K) 方法（计算 K 的 hash 值）从而获取该键值所在链表的数组下标；②、顺序遍历链表，equals()方法查找相同 Node 链表中 K 值对应的 V 值。

#### 当两个对象的 hashCode 相同会发生什么？

> 因为 hashCode 相同，不一定就是相等的（equals方法比较），所以两个对象所在数组的下标相同，"碰撞"就此发生。又因为 HashMap 使用链表存储对象，这个 Node 会存储到链表中。为什么要重写 hashcode 和 equals 方法？推荐看下。

#### 你知道 hash 的实现吗？为什么要这样实现？

> JDK 1.8 中，是通过 hashCode() 的高16位 异或 低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度，功效和质量来考虑的，减少系统的开销，也不会造成因为高位没有参与下标的计算，从而引起的碰撞。

#### 为什么要用异或运算符？

> 保证了对象的 hashCode 的 32 位值只要有一位发生改变，整个 hash() 返回值就会改变。尽可能的减少碰撞。

#### HashMap 的 table 的容量如何确定？loadFactor 是什么？该容量如何变化？这种变化会带来什么问题？

①、table 数组大小是由 capacity 这个参数确定的，默认是16，也可以构造时传入，最大限制是1<<30；
②、loadFactor 是装载因子，主要目的是用来确认table 数组是否需要动态扩展，默认值是0.75，比如table 数组大小为 16，装载因子为 0.75 时，threshold 就是12，当 table 的实际大小超过 12 时，table就需要动态扩容；
③、扩容时，调用 resize() 方法，将 table 长度变为原来的两倍（注意是 table 长度，而不是 threshold）
④、如果数据很大的情况下，扩展时将会带来性能的损失，在性能要求很高的地方，这种损失很可能很致命。

#### HashMap中put方法的过程?

调用哈希函数获取Key对应的hash值，再计算其数组下标；
如果没有出现哈希冲突，则直接放入数组；如果出现哈希冲突，则以链表的方式放在链表后面；
如果链表长度超过阀值( TREEIFY THRESHOLD==8)，就把链表转成红黑树，链表长度低于6，就把红黑树转回链表;
如果结点的key已经存在，则替换其value即可；
如果集合中的键值对大于12，调用resize方法进行数组扩容。

#### 数据扩容的过程

> 创建一个新的数组，其容量为旧数组的两倍，并重新计算旧数组中结点的存储位置。结点在新数组中的位置只有两种，原下标位置或原下标+旧数组的大小。

#### 拉链法导致的链表过深问题为什么不用二叉查找树代替，而选择红黑树？为什么不一直使用红黑树？

之所以选择红黑树是为了解决二叉查找树的缺陷，二叉查找树在特殊情况下会变成一条线性结构（这就跟原来使用链表结构一样了，造成很深的问题），遍历查找会非常慢。
而红黑树在插入新数据后可能需要通过左旋，右旋、变色这些操作来保持平衡，引入红黑树就是为了查找数据快，解决链表查询深度的问题，我们知道红黑树属于平衡二叉树，但是为了保持“平衡”是需要付出代价的，但是该代价所损耗的资源要比遍历线性链表要少，所以当长度大于8的时候，会使用红黑树，如果链表长度很短的话，根本不需要引入红黑树，引入反而会慢。

#### 说说你对红黑树的见解？

* 每个节点非红即黑
* 根节点总是黑色的
* 如果节点是红色的，则它的子节点必须是黑色的（反之不一定）
* 每个叶子节点都是黑色的空节点（NIL节点）
* 从根节点到叶节点或空子节点的每条路径，必须包含相同数目的黑色节点（即相同的黑色高度）

#### jdk8中对HashMap做了哪些改变？

在java 1.8中，如果链表的长度超过了8，那么链表将转换为红黑树。（桶的数量必须大于64，小于64的时候只会扩容）
发生hash碰撞时，java 1.7 会在链表的头部插入，而java 1.8会在链表的尾部插入
在java 1.8中，Entry被Node替代。

#### HashMap，LinkedHashMap，TreeMap 有什么区别？

**HashMap** HashMap无序
**LinkedHashMap** 保存了记录的插入顺序，在用 Iterator 遍历时，先取到的记录肯定是先插入的；遍历比 HashMap 慢；继承于HashMap, 是基于HashMap和双向链表来实现的。
**TreeMap** 实现 SortMap 接口，能够把它保存的记录根据键排序（默认按键值升序排序，也可以指定排序的比较器）

#### HashMap & TreeMap & LinkedHashMap 使用场景？

一般情况下，使用最多的是 HashMap。
HashMap：在 Map 中插入、删除和定位元素时；
TreeMap：在需要按自然顺序或自定义顺序遍历键的情况下；
LinkedHashMap：有顺序地去存储key-value; 默认是插入顺序

#### 面试问红黑树，我脸都绿了
https://mp.weixin.qq.com/s/JQqdzMJ5cL3AAbZgBXpAIw

#### HashMap 容量为什么总是为 2 的次幂？
https://mp.weixin.qq.com/s/UZRk3VGt96BTwIRktjSO0g

#### 为什么要重写 hashcode 和 equals 方法？
https://mp.weixin.qq.com/s/5pVKo8QlJ3pYbImC5FlZvA

