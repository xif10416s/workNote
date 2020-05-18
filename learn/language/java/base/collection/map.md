####   HashMap
* 以key-value方式存储数据，可以快速存放和快速读取

* hashmap的比较重要的两个参数initialCapacity（初始容量），loadFactor（负载因子），对于性能有影响，都可以通过构造函数进行初始化
  
  * initialCapacity 默认初始大小为 1 << 4  ,  通常推荐初始化时指定，根据实际业务情况设置合理大小，避免rehash操作
    * initialCapacity  = 大于 （ 业务预估最大数量（10） /  负载因子（0.75））  的最近的一个2的倍数  ==》 16
  * loadFactor 默认为0.75f ， 表示容量到达75%时，需要进行扩容，称为rehash,内部数据结构重建，容量扩大一倍
    * 通常0.75不用改动，对于时间和空间的使用比较平衡
  
  
  
######   传统 HashMap的缺点
* (1)JDK 1.8 以前 HashMap 的实现是 数组+链表，即使哈希函数取得再好，也很难达到元素百分百均匀分布。

* (2)当 HashMap 中有大量的元素都存放到同一个桶中时，这个桶下有一条长长的链表，这个时候 HashMap 就相当于一个单链表，假如单链表有 n 个元素，遍历的时间复杂度就是 O(n)，完全失去了它的优势。

* (3)针对这种情况，JDK 1.8 中引入了红黑树（查找时间复杂度为 O(logn)）来优化这个问题

  *	#### HashMap是数组+链表+红黑树（JDK1.8增加了红黑树部分）实现的



######  红黑树切换

| TREEIFY_THRESHOLD                                            | UNTREEIFY_THRESHOLD                                          | MIN_TREEIFY_CAPACITY                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 链表  转  红黑树的阈值                                       | 一个树的链表还原阈值                                         | 哈希表的最小树形化容量                                       |
| TREEIFY_THRESHOLD = 8                                        | UNTREEIFY_THRESHOLD = 6                                      | MIN_TREEIFY_CAPACITY = 64                                    |
| 1.  当桶中元素个数超过这个值时 需要使用红黑树节点替换链表节点 | 1.当扩容时，桶中元素个数小于这个值 就会把树形的桶元素 还原（切分）为链表结构 | 1.当哈希表中的容量大于这个值时，表中的桶才能进行树形化    否则桶内元素太多时会扩容，而不是树形化 |

* TREEIFY_THRESHOLD = 8 

  * 根据泊松分布概率质量函数,一个哈希桶达到 9 个元素的概率小于一千万分之一.

* UNTREEIFY_THRESHOLD = 6

####  主要成员 & 方法

````java
transient Node<K,V>[] table;  --- 数组，用来存放元素

// 计算key的hash值 ，hashcode是int 32为，
// 为了让hashcode 高16位于 与 低16位都参与寻址运算 (n-1) & hash, 主要是因为n-1的二进制数，左边全为0，高16位被忽略了
//  key的hashcode右移16位 异或 
/**
1111 1111 1111 1111 1111 1010 0111 1100   <== 原始值
0000 0000 0000 0000 1111 1111 1111 1111   <== 右移16位
1111 1111 1111 1111 0000 0101 1000 0011 -> int值，32位 ，异或 ,将 高16的特征加入低16位

0000 0000 0000 0000 0000 0000 0000 1111  <== n -1 
*
*/
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

// 添加一个元素
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 如果是第一次添加元素
        if ((tab = table) == null || (n = tab.length) == 0)
            // 初始化 tab 数组，并返回长度n 
            n = (tab = resize()).length;
        // n为2的指数，n-1的二进制数全是1， 与hash值与操作为 0~n-1的索引位置
        //  通过key的hash值 与 数组的长度n-1 的与操作 确定索引位置
        if ((p = tab[i = (n - 1) & hash]) == null) // 寻址优化 通过 二进制运算
            // 数组索引位置没有元素就直接新建元素，并插入数组
            tab[i] = newNode(hash, key, value, null);
   		//  如果索引位置已经有元素
        else {
            Node<K,V> e; K k;
            // 如果是key相同，则将旧的元素p 放入临时变量e，最后直接替换
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 如果旧元素是树节点，则添加入树
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 普通链表节点处理，将节点添加到链表的末端
                for (int binCount = 0; ; ++binCount) {
                    // 找到最后一个节点，添加新元素
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // 如果冲突链表元素大于等于 8-1的时候，将节点转换成树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 如果与链表中的某个key重复，则拿出来准备替换
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //  添加元素的key已经存在的情况，直接替换
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
    // 超过了容量 就扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

// 将链表节点树形化
final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
    // 如果当前哈希表为空，或者哈希表中元素的个数小于 进行树形化的阈值(默认为 64)，就去新建/扩容
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            //红黑树的头、尾节点
            TreeNode<K,V> hd = null, tl = null;
            do {
                // //新建一个树形节点，内容和当前链表节点 e 一致
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
             //让桶的第一个元素指向新建的红黑树头结点，以后这个桶里的元素就是红黑树而不是链表了
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
````



  

































####  参考

* https://blog.csdn.net/lianhuazy167/article/details/66967698

