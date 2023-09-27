# HashMap
> 主要关于HashMap在1.7与1.8之后GET和PUT时代码的分析

- **java1.7之前实现概述**

  - 采用数组+单向链表，不支持多线程操作

  - 每次的扩容大小都是2的幂次方，保证了键值的哈希值与容器大小的与操作的分布性；

  - 如果初始化没有指定，默认为0.75，达到阈值就会发生扩容；

  - 每个Entry是由key,value,hashcode,next组成，后续的1.8是Node;

  - PUT源码分析

    ~~~java
    public V put(K key, V value) {
        // 当插入第一个元素的时候，需要先初始化数组大小
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);//下文描述
        }
        // 如果 key 为 null，最终会将这个 entry 放到 table[0] 中
        if (key == null)
            return putForNullKey(value);
        // 1. 求 key 的 hash 值
        int hash = hash(key);
        // 2. 找到对应的数组下标
        int i = indexFor(hash, table.length); //下文描述
        // 3. 遍历一下对应下标处的链表，看是否有重复的 key 已经存在
        //    如果有，直接覆盖，put 方法返回旧值就结束了
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i); //下文
        return null;
    }
    ~~~

  - 容器初始化源码：（inflateTable）

    ~~~java
    private void inflateTable(int toSize) {
        // 保证数组大小一定是 2 的 n 次方。如果大小为20，那么初始为32
        int capacity = roundUpToPowerOf2(toSize);
        // 计算扩容阈值：capacity * loadFactor
        threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
        // 算是初始化数组吧
        table = new Entry[capacity];
        initHashSeedAsNeeded(capacity); //ignore
    }
    ~~~

  - 计算具体位置 ： (indexFor)

    ~~~java
    static int indexFor(int hash, int length) {
        // 键值的hashcode值与数组的大小的与操作决定index
        return hash & (length-1);
    }
    ~~~

  - 添加节点（addEntry）

    1.首先判断是否需要扩容，如果需要，先扩容，后把数据存放在修改后index的第一个位置；

    2.如果不扩容，那么看index是否重复，然后放在第一个位置

    ~~~java
    void addEntry(int hash, K key, V value, int bucketIndex) {
        // 如果当前 HashMap 大小已经达到了阈值，并且新值要插入的数组位置已经有元素了，那么要扩容
        if ((size >= threshold) && (null != table[bucketIndex])) {
            // 扩容，后面会介绍一下
            resize(2 * table.length);
            // 扩容以后，重新计算 hash 值
            hash = (null != key) ? hash(key) : 0;
            // 重新计算扩容后的新的下标
            bucketIndex = indexFor(hash, table.length);
        }
        createEntry(hash, key, value, bucketIndex);//下文
    }

    // 这个很简单，其实就是将新值放到链表的表头，然后 size++
    void createEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        size++;
    }
    ~~~


  - 扩容：（resize）

    ~~~java
    void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }
        // 新的数组
        Entry[] newTable = new Entry[newCapacity];
        // 将原来数组中的值迁移到新的更大的数组中
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }
    ~~~

  - GET分析

    1.根据key值计算hashcode值；

    2.找到对应的下标hashcode值与容量的大小-1；即为index值；

    3.遍历该数组处的链表，找到key值相同的key;

    ~~~java
    public V get(Object key) {
        // 如果键值为NULL，那么会遍历Table[0]下的节点就可以获取到
        if (key == null)
            return getForNullKey();
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
    }
    ~~~

- **java1.8之后实现概述**

  - 采用数组+链表+红黑树，当一个entry（实际是Node）下的链表节点多于8个时，会使用红黑树，降低查找的时间；

  - 我们通过判断该位置下第一个元素是TreeNode还是Node来决定是以链表还是红黑树的方式去遍历；

  - PUT源码分析： （与1.7中不同的是，1.8是先插入后扩容）

    ~~~java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
     
    // 第三个参数 onlyIfAbsent 如果是 true，那么只有在不存在该 key 时才会进行 put 操作
    // 第四个参数 evict 我们这里不关心
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p;
        int n, i;
        // 第一次 put 值的时候，会触发下面的 resize()，是初始化数组
        // 第一次 resize 和后续的扩容有些不一样
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 找到具体的数组下标，如果此位置没有值，那么直接初始化一下 Node 并放置在这个位置就可以了
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
     
        else {// 数组该位置有数据
            Node<K,V> e; K k;
            // 首先，判断该位置的第一个数据和我们要插入的数据，key 值是否相同
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && 
                 key.equals(k))))
                e = p;
            // 如果该节点是代表红黑树的节点，调用红黑树的插值方法，本文不展开说红黑树
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 到这里，说明数组该位置上是一个链表
                for (int binCount = 0; ; ++binCount) {
                    // 插入到链表的最后面(Java7 是插入到链表的最前面)
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // TREEIFY_THRESHOLD 为 8，所以，如果新插入的值是链表中的第 9 个
                        // 会触发下面的 treeifyBin，也就是将链表转换为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 如果在该链表中找到了
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        // 此时 break，那么 e 为链表中[与要插入的新值的 key "相等"]的 node
                        break;
                    p = e;
                }
            }
            // e!=null 说明存在旧值的key与要插入的key"相等"
            // 对于我们分析的put操作，下面这个 if 其实就是进行 "值覆盖"，然后返回旧值
            if (e != null) {
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 如果 HashMap 由于新插入这个值导致 size 已经超过了阈值，需要进行扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
    ~~~

  - GET分析

    1.计算key的hash值，index = hashcode & (length-1)

    2.判断该位置的元素是否为要找的元素；

    3.判断当前位置元素的类型，如果是Node，用链表的方式遍历，如果是TreeNode，用红黑树的方式取；

    4.遍历取出相等的元素；

    ~~~java
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            // 判断第一个节点是不是就是需要的
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                // 判断是否是红黑树
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                // 链表遍历
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
    ~~~