package com.javaee.onlinehosbackend.extrautil;


import java.io.IOException;
import java.io.InvalidObjectException;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.*;
import java.util.function.BiConsumer;
import java.util.function.BiFunction;

/**
 * 我的哈希map
 * hash map 源码分析，基于哈希表的结构
 * 什么是hash表：根据键直接获取值所在地址从而快速查询到值
 * 哈希表、散列表，用于快速操作和查找，时间复杂度为O(1)
 * 无序哈希表，可空键空值
 * 初始容量：桶的数量
 * 负载因子：参与哈希扩容的参数，其与初始容量的乘积小于哈希表中数据条目数的时候，哈希表将会扩容
 *
 * @author 邓应来
 */
public class MyHashMapK, V> implements MapK, V> {
    private static final long serialVersionUID = 362498820763181265L;


    /**
     *
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
     * 最大容量，如果两个构造函数中的任何一个带参数地隐式指定了更高的值，则使用最大容量。必须是2的幂<= 1<<30
     * 1、什么叫隐式地指定了更高的值？
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 默认负载因子。
     * 1、为什么是0.75
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * 链表树化阈值，须大于2，并且应该至少为8，
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * 非树化阈值，红黑树变为链表时的阈值，应小于链表树化阈值，且最多为6
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * 可以对桶进行树化的最小表容量。
     * (否则，如果一个bin中的节点太多，则会调整表的大小。)
     * 应该至少为4 * TREEIFY_THRESHOLD， 4倍链表树化阈值，以避免调整大小和树化阈值之间的冲突
     * 【树化最小表容量】
     */
    static final int MIN_TREEIFY_CAPACITY = 64;

    /**
     * 哈希表节点对象
     */
    static class NodeK, V> implements Map.EntryK, V> {
        final int hash;
        final K key;
        V value;
        NodeK, V> next;

        Node(int hash, K key, V value, NodeK, V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        @Override
        public final K getKey() {
            return key;
        }

        @Override
        public final V getValue() {
            return value;
        }

        @Override
        public final String toString() {
            return key + "=" + value;
        }

        /**
         * 获取hash值，
         * 算法为 key的hash值 异或 value的hash值
         *
         * @return
         */
        @Override
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        @Override
        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        @Override
        public final boolean equals(Object o) {
            if (o == this) {
                return true;
            }

            if (o instanceof Map.Entry) {
                Map.Entry<?, ?> e = (Map.Entry<?, ?>) o;
                if (Objects.equals(key, e.getKey()) && Objects.equals(value, e.getValue())) {
                    return true;
                }

            }
            return false;
        }
    }

    /* ---------------- Static utilities -------------- */

    /**
     * 计算hash值
     * 减少高位特征在计算桶位置时的影响，存在高位不同但是低位相同的hash值，
     * 计算桶位置的时候，高位会被屏蔽，则相同的低位计算到的桶位置也是相同的，会出现hash冲突
     * 将高位特征与低位特征结合，可以实现hash值扰动，减少高位特征的影响，减少hash冲突，进而提高性能
     */
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

    /**
     * Returns x's Class if it is of the form "class C implements
     * ComparableC>", else null.
     * 判断一个对象是否实现了Comparable接口，是就返回这个对象的类型，否就返回null
     */
    static Class?> comparableClassFor(Object x) {
        if (x instanceof Comparable) {
            Class?> c;
            Type[] ts, as;
            Type t;
            ParameterizedType p;
            if ((c = x.getClass()) == String.class) // bypass checks
                return c;
            if ((ts = c.getGenericInterfaces()) != null) {
                for (int i = 0; i < ts.length; ++i) {
                    if (((t = ts[i]) instanceof ParameterizedType) &&
                            ((p = (ParameterizedType) t).getRawType() ==
                                    Comparable.class) &&
                            (as = p.getActualTypeArguments()) != null &&
                            as.length == 1 && as[0] == c) // type arg is c
                        return c;
                }
            }
        }
        return null;
    }

    /**
     * Returns k.compareTo(x) if x matches kc (k's screened comparable
     * class), else 0.
     * p>
     * 对象比较，
     *
     * @SuppressWarnings忽略编译警告
     */
    @SuppressWarnings({"rawtypes", "unchecked"}) // for cast to Comparable
    static int compareComparables(Class?> kc, Object k, Object x) {
        return (x == null || x.getClass() != kc ? 0 :
                ((Comparable) k).compareTo(x));
    }

    /**
     * 这个算法会将传入的任意数字转换为与其最接近的高位的2的幂次方值，但是会小于最大容量值
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }

    /* ---------------- Fields -------------- */

    /**
     * 表，hashmap基于数组结构，元素存放于数组中，这个数组称之为“表”，数组对象为hashmap中定义的节点对象，这个节点对象有链表的特性
     * hash表的节点集合
     * 该表在首次使用时初始化，并根据需要调整大小。
     * 分配时，长度始终是 2 的幂。（在某些操作中，我们还允许长度为零，以允许当前不需要的引导机制。
     */
    transient NodeK, V>[] table;

    /**
     * Holds cached entrySet(). Note that AbstractMap fields are used
     * for keySet() and values().
     */
    transient SetMap.EntryK, V>> entrySet;

    /**
     * 此映射中包含的键值映射数。即集合大小
     */
    transient int size;

    /**
     * hash表操作次数，相当于版本控制
     * put、remove、merge、clear等操作都会使得该值增加，
     * 在foreach、replaceAll等会影响hash表结构的操作当中，会判断操作期间这个值是否变化，如果变化了，会快速失败
     */
    transient int modCount;

    /**
     * 扩容阈值，通过表容量与负载因子计算得来
     * 初始化的时候不指定表容量，这个值为0，
     * 初始化时指定表容量，这个值等于计算后的表容量
     * 但是第一次put的时候会将表容量与负载因子相乘计算并给这个树形重新赋值
     */
    int threshold;

    /**
     * 哈希表的负载因子
     */
    final float loadFactor;

    /* ---------------- Public operations -------------- */

    /**
     * 构造一个具有指定初始容量和负载系数的空 HashMap
     *
     * @param initialCapacity 初始容量
     * @param loadFactor      负载系数
     * @throws IllegalArgumentException
     */
    public MyHashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0) {
            throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
        }

        if (initialCapacity > MAXIMUM_CAPACITY) {
            initialCapacity = MAXIMUM_CAPACITY;
        }
        // 防止传入的负载因子值为NaN
        if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
            throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
        }
        this.loadFactor = loadFactor;
        // 初始化扩容阈值
        this.threshold = tableSizeFor(initialCapacity);
    }

    /**
     * 构造一个具有指定初始容量和默认负载因子0.75f的空hashmap
     *
     * @param initialCapacity 初始容量。
     * @throws IllegalArgumentException
     */
    public MyHashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     */
    public MyHashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

    /**
     * Constructs a new <tt>HashMap</tt> with the same mappings as the
     * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
     * default load factor (0.75) and an initial capacity sufficient to
     * hold the mappings in the specified <tt>Map</tt>.
     *
     * @param m the map whose mappings are to be placed in this map
     * @throws NullPointerException if the specified map is null
     */
    public MyHashMap(Map? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }

    /**
     * 将一个map的元素添加到当前对象当中
     *
     * @param m     传入的map集合
     * @param evict false when initially constructing this map, else
     *              true (relayed to method afterNodeInsertion).
     */
    final void putMapEntries(Map? extends K, ? extends V> m, boolean evict) {
        int s = m.size();
        if (s > 0) {
            if (table == null) { // pre-size
                // 原集合大小 / 负载因子 + 1  相当于得到 一个扩容之后的大小
                float ft = ((float) s / loadFactor) + 1.0F;
                //计算扩容大小
                int t = ((ft < (float) MAXIMUM_CAPACITY) ? (int) ft : MAXIMUM_CAPACITY);
                //如果扩容之后的大小大于要调整的下一个容量值，就调整下一个容量值
                if (t > threshold) {
                    threshold = tableSizeFor(t);
                }

            } else if (s > threshold) {
                resize();
            }
            for (Map.Entry? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }

    /**
     * Returns the number of key-value mappings in this map.
     *
     * @return the number of key-value mappings in this map
     */
    @Override
    public int size() {
        return size;
    }

    /**
     * Returns <tt>true</tt> if this map contains no key-value mappings.
     *
     * @return <tt>true</tt> if this map contains no key-value mappings
     */
    @Override
    public boolean isEmpty() {
        return size == 0;
    }

    /**
     * 通过key值获取到映射的value值，
     * 如果返回null，不一定表示没有获取到映射的值，也可能是key值映射到了值为null的value值
     *
     * @see #put(Object, Object)
     */
    @Override
    public V get(Object key) {
        NodeK, V> e = getNode(hash(key), key);
        return e == null ? null : e.value;
    }

    /**
     * 通过key获取到对应的节点
     *
     * @param hash hash for key
     * @param key  the key
     * @return the node, or null if none
     */
    final NodeK, V> getNode(int hash, Object key) {
        NodeK, V>[] tab;
        NodeK, V> first, e;
        int n;
        K k;
        // 通过hash计算得到表索引
        if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k)))) {
                return first;
            }
            if ((e = first.next) != null) {
                // 如果桶对应的是红黑树 就从树中获取
               /* if (first instanceof TreeNode){
                    return ((TreeNodeK,V>)first).getTreeNode(hash, key);
                }*/

                //hash冲突，遍历链表找到指定的节点
                do {
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                        return e;
                    }
                } while ((e = e.next) != null);
            }
        }
        return null;
    }

    /**
     * 判断集合中是否有映射
     * specified key.
     *
     * @param key The key whose presence in this map is to be tested
     * @return <tt>true</tt> if this map contains a mapping for the specified
     * key.
     */
    @Override
    public boolean containsKey(Object key) {
        return getNode(hash(key), key) != null;
    }

    /**
     * 向集合中添加元素，将键值关联映射，如果键已经有对应的值，那么就将值替换
     */
    @Override
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * 键-值映射
     *
     * @param hash         hash for key
     * @param key          the key
     * @param value        the value to put
     * @param onlyIfAbsent 如果为 true，则不会更新相同key的值
     * @param evict        如果为 false，则表处于创建模式
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        NodeK, V>[] tab;
        NodeK, V> p;
        int n, i;
        if ((tab = table) == null || (n = tab.length) == 0) {
            //表为空则初始化，这里可以看出来，hashmap是懒加载的，只有在首次填充数据的时候才会创建表
            n = (tab = resize()).length;
        }

        //表索引计算（hash计算），在表中指定索引位置添加节点
        if ((p = tab[i = (n - 1) & hash]) == null) {
            tab[i] = newNode(hash, key, value, null);
        } else {
            //这个索引的位置已经存在节点（hash冲突）
            NodeK, V> e;
            K k;
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))) {
                e = p;
            }

            // 如果节点已经是红黑树的节点，则使用红黑树处理
           /* else if (p instanceof TreeNode){
                //树形节点有树形节点的添加方法
                e = ((TreeNodeK,V>)p).putTreeVal(this, tab, hash, key, value);
            }*/

            else {
                // 遍历链表，直到找到相同的key或者找到空位
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        //在链表末尾添加节点
                        p.next = newNode(hash, key, value, null);
                        //链表中节点数量大于树化阈值，
                        if (binCount >= TREEIFY_THRESHOLD - 1) {
                            treeifyBin(tab, hash);
                        }
                        break;
                    }
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                        break;
                    }
                    p = e;
                }
            }
            // 集合中已经有完全相同的key
            if (e != null) {
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null) {
                    // 更新key映射的value
                    e.value = value;
                }
                //这里是linkedhashmap操作
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 如果元素超过了扩容阈值，则需要重新hash并扩容
        if (++size > threshold) {
            resize();
        }
        // 这里是linkedhashmap操作的回调
        afterNodeInsertion(evict);
        return null;
    }

    /**
     * 初始化表或者扩容
     * 扩容按照2倍扩容，重新hash后的索引index结果只有两种：
     * 1、index不变；2、index + oldCap （旧索引加上旧容量）
     *
     * @return the table
     */
    final NodeK, V>[] resize() {
        NodeK, V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            } else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY) {
                //扩容，新表容量*2 ，新阈值*2
                newThr = oldThr << 1;
            }
        } else if (oldThr > 0) {
            // 初始化hash表的时候指定了容量大小，此时扩容阈值为2的幂次方，令表容量等于扩容阈值
            newCap = oldThr;
        } else {
            // 初始化不指定表容量，使用默认的表容量
            newCap = DEFAULT_INITIAL_CAPACITY;
            // 根据负载因子和表容量计算扩容阈值
            newThr = (int) (DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            // 初始化的时候指定了表容量大小，扩容阈值等于计算后的表容量值，需要重新计算并赋值
            // 扩容阈值 = 表容量 * 负载因子（初始值是默认的0.75）
            float ft = (float) newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float) MAXIMUM_CAPACITY ? (int) ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes", "unchecked"})
        NodeK, V>[] newTab = (NodeK, V>[]) new Node[newCap];
        table = newTab;
        //旧表扩容
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                NodeK, V> e;
                if ((e = oldTab[j]) != null) {
                    //旧节点置空
                    oldTab[j] = null;
                    if (e.next == null) {
                        // 重新hash
                        newTab[e.hash & (newCap - 1)] = e;
                    }
                 /*   else if (e instanceof TreeNode){

                        ((TreeNodeK,V>)e).split(this, newTab, j, oldCap);
                    }*/
                    else {
                        // 桶中保存的是链表，新表扩容后重新hash，各个key经过hash计算后的索引值会变更，需要重新保存链表节点
                        NodeK, V> loHead = null, loTail = null;
                        NodeK, V> hiHead = null, hiTail = null;
                        NodeK, V> next;
                        do {
                            next = e.next;
                            //扩容之后的hash计算结果是不相同的，同一链表的元素重新hash之后可能会放到不同的桶中
                            if ((e.hash & oldCap) == 0) {
                                // 如果hash按位与旧容量值为0，则改节点对应的桶不需要改变
                                if (loTail == null) {
                                    loHead = e;
                                } else {
                                    loTail.next = e;
                                }
                                loTail = e;
                            } else {
                                // 这里节点对应的桶位置需要偏移一个旧容量的数
                                if (hiTail == null) {
                                    hiHead = e;
                                } else {
                                    hiTail.next = e;
                                }
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
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

    /**
     * 将hash表中指定索引处的链表树化
     * 如果表容量小于【最小树化容量】，就不进行树化，而是直接重新hash扩容
     */
    final void treeifyBin(NodeK, V>[] tab, int hash) {
        int n, index;
        NodeK, V> e;
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            //重新hash扩容
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
           /* TreeNodeK,V> hd = null, tl = null;
            do {
                TreeNodeK,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);*/
            throw new RuntimeException("暂未实现树化逻辑");
        }
    }

    /**
     * Copies all of the mappings from the specified map to this map.
     * These mappings will replace any mappings that this map had for
     * any of the keys currently in the specified map.
     *
     * @param m mappings to be stored in this map
     * @throws NullPointerException if the specified map is null
     */
    @Override
    public void putAll(Map<? extends K, ? extends V> m) {
        putMapEntries(m, true);
    }

    /**
     * 移除元素
     */
    @Override
    public V remove(Object key) {
        NodeK, V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ? null : e.value;
    }

    /**
     * 移除表元素
     *
     * @param hash       hash for key
     * @param key        the key
     * @param value      值，如果需要匹配的话
     * @param matchValue 如果为 true，则仅在值相等时删除
     * @param movable    如果为 false，则在删除时不移动其他节点
     * @return the node, or null if none
     */
    final NodeK, V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) {
        NodeK, V>[] tab;
        NodeK, V> p;
        int n, index;
        if ((tab = table) != null && (n = tab.length) > 0 && (p = tab[index = (n - 1) & hash]) != null) {
            Node, V> node = null, e;
            K k;
            V v;
            if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))) {
                node = p;
            } else if ((e = p.next) != null) {
               /* if (p instanceof TreeNode){
                    node = ((TreeNodeK,V>)p).getTreeNode(hash, key);
                }

                else {
                    do {
                        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }*/

                do {
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);

            }
            if (node != null && (!matchValue || (v = node.value) == value || (value != null && value.equals(v)))) {

                // 移除红黑树节点
               /* if (node instanceof TreeNode){
                    ((TreeNodeK,V>)node).removeTreeNode(this, tab, movable);
                } else*/

                if (node == p) {
                    // 这里表示要移除元素就在桶中链表头节点
                    tab[index] = node.next;
                } else {
                    p.next = node.next;
                }
                ++modCount;
                --size;
                // linkedlis的操作了
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }

    /**
     * 从此映射中移除所有映射。
     * 此调用返回后，映射将为空。
     */
    @Override
    public void clear() {
        NodeK, V>[] tab;
        modCount++;
        if ((tab = table) != null && size > 0) {
            size = 0;
            for (int i = 0; i < tab.length; ++i) {
                tab[i] = null;
            }
        }
    }

    @Override
    public SetK> keySet() {
        // 忽略这个方法
        return null;
    }

    @Override
    public CollectionV> values() {
        // 忽略这个方法
        return null;
    }

    /**
     * 判断集合中是否有指定的value
     * 这个方法比较耗时，因为需要遍历桶，然后还要遍历桶中的链表
     *
     * @param value value whose presence in this map is to be tested
     * @return <tt>true</tt> if this map maps one or more keys to the
     * specified value
     */
    @Override
    public boolean containsValue(Object value) {
        NodeK, V>[] tab;
        V v;
        if ((tab = table) != null && size > 0) {
            for (int i = 0; i < tab.length; ++i) {
                for (NodeK, V> e = tab[i]; e != null; e = e.next) {
                    if ((v = e.value) == value || (value != null && value.equals(v))) {
                        return true;
                    }
                }
            }
        }
        return false;
    }


    /**
     * 忽略这个方法
     */
    @Override
    public SetMap.EntryK, V>> entrySet() {
        return null;
    }


    @Override
    public V getOrDefault(Object key, V defaultValue) {
        NodeK, V> e;
        return (e = getNode(hash(key), key)) == null ? defaultValue : e.value;
    }

    //存入元素，如果集合没有的话
    @Override
    public V putIfAbsent(K key, V value) {
        return putVal(hash(key), key, value, true, true);
    }

    @Override
    public boolean remove(Object key, Object value) {
        return removeNode(hash(key), key, value, true, true) != null;
    }

    // 替换value
    @Override
    public boolean replace(K key, V oldValue, V newValue) {
        NodeK, V> e;
        V v;
        if ((e = getNode(hash(key), key)) != null && ((v = e.value) == oldValue || (v != null && v.equals(oldValue)))) {
            e.value = newValue;
            afterNodeAccess(e);
            return true;
        }
        return false;
    }

    @Override
    public V replace(K key, V value) {
        NodeK, V> e;
        if ((e = getNode(hash(key), key)) != null) {
            V oldValue = e.value;
            e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
        return null;
    }


    //忽略
    @Override
    public V compute(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        return null;
    }

    /**
     * 合并
     *
     * @param key               key with which the resulting value is to be associated
     * @param value             the non-null value to be merged with the existing value
     *                          associated with the key or, if no existing value or a null value
     *                          is associated with the key, to be associated with the key
     * @param remappingFunction the function to recompute a value if present
     * @return
     */
    @Override
    public V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
        if (value == null) {
            throw new NullPointerException();
        }
        if (remappingFunction == null) {
            throw new NullPointerException();
        }
        int hash = hash(key);
        NodeK, V>[] tab;
        NodeK, V> first;
        int n, i;
        int binCount = 0;
        // TreeNodeK,V> t = null;
        NodeK, V> old = null;
        if (size > threshold || (tab = table) == null || (n = tab.length) == 0) {
            n = (tab = resize()).length;
        }

        if ((first = tab[i = (n - 1) & hash]) != null) {
           /* if (first instanceof TreeNode)
                old = (t = (TreeNodeK,V>)first).getTreeNode(hash, key);
            else*/

            {
                // 遍历链表找到key对应的节点
                NodeK, V> e = first;
                K k;
                do {
                    if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                        old = e;
                        break;
                    }
                    ++binCount;
                } while ((e = e.next) != null);
            }
        }
        // hash表中已经存在对应的key
        if (old != null) {
            V v;
            if (old.value != null) {
                v = remappingFunction.apply(old.value, value);
            } else {
                v = value;
            }

            if (v != null) {
                old.value = v;
                afterNodeAccess(old);
            } else {
                // 合并方法将会移除value为空的节点
                removeNode(hash, key, null, false, true);
            }

            return v;
        }
        if (value != null) {
           /* if (t != null){
                t.putTreeVal(this, tab, hash, key, value);
            } else */
            {
                tab[i] = newNode(hash, key, value, first);

                if (binCount >= TREEIFY_THRESHOLD - 1) {
                    treeifyBin(tab, hash);
                }

            }
            ++modCount;
            ++size;
            afterNodeInsertion(true);
        }
        return value;
    }

    /**
     * 循环
     *
     * @param action The action to be performed for each entry
     */
    @Override
    public void forEach(BiConsumer<? super K, ? super V> action) {
        NodeK, V>[] tab;
        if (action == null) {
            throw new NullPointerException();
        }
        if (size > 0 && (tab = table) != null) {
            int mc = modCount;
            for (int i = 0; i < tab.length; ++i) {
                for (NodeK, V> e = tab[i]; e != null; e = e.next) {
                    action.accept(e.key, e.value);
                }
            }
            if (modCount != mc) {
                throw new ConcurrentModificationException();
            }

        }
    }

    /**
     * @param function 应用于每个条目的函数
     */
    @Override
    public void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
        NodeK, V>[] tab;
        if (function == null) {
            throw new NullPointerException();
        }
        if (size > 0 && (tab = table) != null) {
            int mc = modCount;
            for (int i = 0; i < tab.length; ++i) {
                for (NodeK, V> e = tab[i]; e != null; e = e.next) {
                    e.value = function.apply(e.key, e.value);
                }
            }
            if (modCount != mc) {
                throw new ConcurrentModificationException();
            }

        }
    }


    /**
     * Returns a shallow copy of this <tt>HashMap</tt> instance: the keys and
     * values themselves are not cloned.
     *
     * @return a shallow copy of this map
     */
    @SuppressWarnings("unchecked")
    @Override
    public Object clone() {
        MyHashMapK, V> result;
        try {
            result = (MyHashMapK, V>) super.clone();
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError(e);
        }
        // 初始化
        result.reinitialize();
        result.putMapEntries(this, false);
        return result;
    }

    // 序列化 HashSet 时也会使用这些方法
    final float loadFactor() {
        return loadFactor;
    }

    /**
     * 获取集合容量
     *
     * @return
     */
    final int capacity() {
        return (table != null) ? table.length : (threshold > 0) ? threshold : DEFAULT_INITIAL_CAPACITY;
    }

    /**
     * Save the state of the <tt>HashMap</tt> instance to a stream (i.e.,
     * serialize it).
     *
     * @serialData The <i>capacity</i> of the HashMap (the length of the
     * bucket array) is emitted (int), followed by the
     * <i>size</i> (an int, the number of key-value
     * mappings), followed by the key (Object) and value (Object)
     * for each key-value mapping.  The key-value mappings are
     * emitted in no particular order.
     */
    private void writeObject(java.io.ObjectOutputStream s) throws IOException {
        int buckets = capacity();
        // Write out the threshold, loadfactor, and any hidden stuff
        s.defaultWriteObject();
        s.writeInt(buckets);
        s.writeInt(size);
        internalWriteEntries(s);
    }

    abstract class HashIterator {
        NodeK, V> next;        // next entry to return
        NodeK, V> current;     // current entry
        int expectedModCount;  // for fast-fail
        int index;             // current slot

        HashIterator() {
            expectedModCount = modCount;
            NodeK, V>[] t = table;
            current = next = null;
            index = 0;
            if (t != null && size > 0) { // advance to first entry
                do {
                } while (index < t.length && (next = t[index++]) == null);
            }
        }

        public final boolean hasNext() {
            return next != null;
        }

        final NodeK, V> nextNode() {
            NodeK, V>[] t;
            NodK, V> e = next;
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            if (e == null)
                throw new NoSuchElementException();
            if ((next = (current = e).next) == null && (t = table) != null) {
                do {
                } while (index < t.length && (next = t[index++]) == null);
            }
            return e;
        }

        public final void remove() {
            NodeK, V> p = current;
            if (p == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            current = null;
            K key = p.key;
            removeNode(hash(key), key, null, false, false);
            expectedModCount = modCount;
        }
    }

    final class KeyIterator extends HashIterator
            implements IteratorK> {
        @Override
        public final K next() {
            return nextNode().key;
        }
    }

    final class ValueIterator extends HashIterator
            implements IteratorV> {
        @Override
        public final V next() {
            return nextNode().value;
        }
    }

    final class EntryIterator extends HashIterator
            implements IteratorMap.EntryK, V>> {
        @Override
        public final Map.EntryK, V> next() {
            return nextNode();
        }
    }

    /* ------------------------------------------------------------ */
    // spliterators


    /**
     * 创建一个常规（非树节点），就是调用node的构造方法
     *
     * @param hash
     * @param key
     * @param value
     * @param next
     * @return
     */
    NodeK, V> newNode(int hash, K key, V value, NodeK, V> next) {
        return new Node<>(hash, key, value, next);
    }


    /**
     * 重置为初始默认状态。 由 clone 和 readObject 调用。
     */
    void reinitialize() {
        table = null;
        entrySet = null;
        modCount = 0;
        threshold = 0;
        size = 0;
    }

    // 这里是linkedhashmap操作的回调
    void afterNodeAccess(Node, V> p) {
    }

    // 这里是linkedhashmap操作的回调
    void afterNodeInsertion(boolean evict) {
    }

    void afterNodeRemoval(Node, V> p) {
    }

    // Called only from writeObject, to ensure compatible ordering.
    void internalWriteEntries(java.io.ObjectOutputStream s) throws IOException {
        NodeK, V>[] tab;
        if (size > 0 && (tab = table) != null) {
            for (int i = 0; i < tab.length; ++i) {
                for (NodeK, V> e = tab[i]; e != null; e = e.next) {
                    s.writeObject(e.key);
                    s.writeObject(e.value);
                }
            }
        }
    }

}
