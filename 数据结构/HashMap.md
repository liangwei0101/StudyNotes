# HashMap
基于哈希表的一个Map接口实现，存储的对象是一个键值对对象(Entry<K,V>)；

来自 <http://www.cnblogs.com/chenpi/p/5280304.html#_label1> 

基于数组和链表实现，内部维护着一个数组table，该数组保存着每个链表的表头结点；查找时，先通过hash函数计算key的hash值，再根据key的hash值计算数组索引（取余法），然后根据索引找到链表表头结点，然后遍历查找该链表；

来自 <http://www.cnblogs.com/chenpi/p/5280304.html#_label1> 

画了个示意图，如下，左边的数组索引是根据key的hash值计算得到，不同hash值有可能产生一样的索引，即哈希冲突，此时采用链地址法处理哈希冲突，即将所有索引一致的节点构成一个单链表；

哈希表的存储过程如下:
	1. 根据 key 计算出它的哈希值 h。
	2. 假设箱子的个数为 n，那么这个键值对应该放在第 (h % n) 个箱子中。
	3. 如果该箱子中已经有了键值对，就使用开放寻址法或者拉链法解决冲突。
	
哈希表在自动扩容时，一般会创建两倍于原来个数的箱子，因此即使 key 的哈希值不变，对箱子个数取余的结果也会发生改变，因此所有键值对的存放位置都有可能发生改变，这个过程也称为重哈希(rehash)。

![Alt text](https://github.com/liangwei0101/StudyNotes/blob/master/images/hash01.jpg)

一般情况是通过hash(key)%len获得，也就是元素的key的哈希值对数组长度取模得到。比如上述哈希表中，12%16=12 , 28%16=12, 108%16=12, 140%16=12。所以12、28、108以及140都存储在数组下标为12的位置。

![Alt text](https://github.com/liangwei0101/StudyNotes/blob/master/images/hash02.jpg)

	从上图可以看出HashMap是Y轴方向是数组，X轴方向就是链表的存储方式。大家都知道数组的存储方式在内存的地址是连续的，大小固定，一旦分配不能被其他引用占用。它的特点是查询快，时间复杂度是O(1)，插入和删除的操作比较慢，时间复杂度是O(n)，链表的存储方式是非连续的，大小不固定，特点与数组相反，插入和删除快，查询速度慢。HashMap可以说是一种折中的方案吧。
	
	1、首先判断Key是否为Null，如果为null，直接查找Enrty[0]，如果不是Null，先计算Key的HashCode，然后经过二次Hash。得到Hash值，这里的Hash特征值是一个int值。
	2、根据Hash值，要找到对应的数组啊，所以对Entry[]的长度length求余，得到的就是Entry数组的index
	

	
	（1）判断key是否为null，若为null，调用putForNullKey(value)处理。这个方法代码如下：
	    /**
     * Offloaded version of put for null keys
     */
    private V putForNullKey(V value) {
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;
        addEntry(0, null, value, 0);
        return null;
    }
	从代码可以看出，如果key为null的值，默认就存储到table[0]开头的链表了。然后遍历table[0]的链表的每个节点Entry，如果发现其中存在节点Entry的key为null，就替换新的value，然后返回旧的value，如果没发现key等于null的节点Entry，就增加新的节点。
	

（2）先计算key的hashcode，在使用计算的结果二次hash，使用indexFor(hash, table.length)方法找到Entry数组的索引i的位置。
（3）接着遍历以table[i]为头结点的链表，如果发现已经存在节点的hash、key值与条件相同时，将该节点的value值替换为新的value值，然后返回旧的value值。
（4）如果未找到hash、key值均相同的节点，则调用addEntry方法增加新的节点（头插法）。代码如下：
    void addEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
        if (size++ >= threshold)
            resize(2 * table.length);
    }

什么时候扩容：当向容器添加元素的时候，会判断当前容器的元素个数，如果大于等于阈值---即当前数组的长度乘以加载因子的值的时候，就要自动扩容啦。

扩容(resize)就是重新计算容量，向HashMap对象里不停的添加元素，而HashMap对象内部的数组无法装载更多的元素时，对象就需要扩大数组的长度，以便能装入更多的元素。当然Java里的数组是无法自动扩容的，方法是使用一个新的数组代替已有的容量小的数组，就像我们用一个小桶装水，如果想装更多的水，就得换大水桶。


HashMap为什么线程不安全

HashMap的自动扩容机制
HashMap内部存储使用了一个Node数组，而Node类包含一个类型为Node的next变量，也就是相当于一个链表，所有的hash值相同（即产生了冲突）的key值会存储到同一个链表中。HashMao内部的Node数组默认的大小是16，默认的负载因子是0.75，当HashMap中的元素超过16\0.75=12时，会把数组扩展为2*16=32，并重新计算每个元素在新数组中的位置

并发下的HashMap使用
	• 如果多个线程同时使用put方法添加元素，而且假设正好存在两个put的key值发生了碰撞，即hash值一样，那么根据hashmap的实现，这两个key会添加到数组的同一个位置，这样会造成其中一个线程的put的数据会被覆盖。
	• 如果多个线程同时检测到元素个数超过数组大小*loadfactor,这样会发生多个线程同时对Node数组进行扩容，都在重新计算元素位置以及复制数据，但是最终只有一个线程扩容后的数组会赋值给table，换言之，其他线程的都会丢失，并且各自线程的put数据也丢失了
	
	• 如果多个线程同时检测到元素个数超过数组大小*loadfactor,这样会发生多个线程同时对Node数组进行扩容，都在重新计算元素位置以及复制数据，但是最终只有一个线程扩容后的数组会赋值给table，换言之，其他线程的都会丢失，并且各自线程的put数据也丢失了

来自 <https://www.jianshu.com/p/5bdbbca3e0c0> 
