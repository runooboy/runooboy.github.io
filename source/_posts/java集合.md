---
title: java集合
date: 2019-09-13 16:05:25
category:
- javaSE
tags:
- 集合
keywords:
- 集合
- java
- javaSE
- Collection
- List
- HashList
---
关于集合，先看下面这张框架图，可以清晰的看到，整个集合主要包含两大块： `Collection`和`Map`，其中Collection接口是一个单列集合，用来存储一个一个的对象，Map接口是一个双列集合，用来存储一对（key - value）一对的数据。接下集合部分主要就分析这两个部分（不会详细讲集合中公有的方法，主要介绍几个实现类的区别和接口的区别）。
![](collections.png)
<!-- more -->

## Collection

> The root interface in the <i>collection hierarchy</i>.  A collection represents a group of objects, known as its <i>elements</i>.

Collection接口下主要有List接口和Set接口子类。
- List接口：存储有序的、可重复的数据(“动态”数组),List主要有ArrayList、LinkedList、Vector实现类。
- Set接口：存储无序的、不可重复的数据，Set主要有HashSet、LinkedHashSet、TreeSet实现类。


### List接口

> An ordered collection (also known as a <i>sequence</i>).  The user of this interface has precise control over where in the list each element is inserted.  The user can access elements by their integer index (position in the list), and search for elements in the list.

上面这段引用就是jdk8对List的定义-一个有序的集合。

#### ArrayList、LinkedList、Vector的异同

- 三个实现类都存储有序的、可重复的数据
- ArrayList相当于数据结构中的线性表，是List接口的主要实现类；具有线程不安全、效率高的特点；底层使用Object[] elementData存储数据。
- LinkedList相当于数据结构中的链表；处理插入、删除操作时，效率比ArrayList高；底层使用双向链表存储数据。
- Vector是List接口的比较老的实现类；线程安全但是效率低；底层使用Object[] elementData存储数据。

由于List下面的三个实现类都比较简单，这里就不做过多的赘述。主要是注意一下如果遍历的过程中要对List进行删除操作的时候最好是使用Iterator，不要使用for（不管是普通for还是增强for），会出现错误，具体错误如下：

```java
@Test
public void test4(){
    List<Employee> list = new ArrayList<>(16);
    for(int i = 0; i < 5; i++){
        list.add(new Employee("test",i,i));
    }

    for(int i = 0; i < list.size(); i++){
        Employee employee = list.get(i);
        if(employee.age == employee.salary) {
            list.remove(i);
        }
    }
    for(Employee employee:list){
        System.out.println(employee);
    }
    /**
     * 打印结果：
     * Employee{name='test', age=1, salary=1}
     * Employee{name='test', age=3, salary=3}
     */
}
```
上面的代码使用普通for循环进行删除操作，按照我们的本意是会将所有元素都删除，但是结果还剩下2个，原因是因为每次删除i位置的元素后，List中i之后的元素向前走一步，然后执行了一次i++，于是乎跳过了一个元素，慢慢的会积累多个出来。再看下增强for：
```java
@Test
public void test4(){
    List<Employee> list = new ArrayList<>(16);
    for(int i = 0; i < 5; i++){
        list.add(new Employee("test",i,i));
    }
    for(Employee employee:list){
        if(employee.age == employee.salary){
            if(list.remove(employee)){
                System.out.println("remove success");
            }
        }
    }
    for(Employee employee:list){
        System.out.println(employee);
    }
    /**
     * 打印结果：
     * run equals()....
     * remove success
     * 
     * java.util.ConcurrentModificationException
     * 	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:907)
     * 	at java.util.ArrayList$Itr.next(ArrayList.java:857)
     * 	at com.chapter12_collection.CollectionTest.test4(CollectionTest.java:70)
     */
}
```
发现增强for的确删除一个元素成功了，但是删除第二个元素的时候报错`ConcurrentModificationException`,于是乎增强for中删除元素也是不可行的。那我们再来看下正确的做法：
```java
@Test
public void test4(){
    List<Employee> list = new ArrayList<>(16);
    for(int i = 0; i < 5; i++){
        list.add(new Employee("test",i,i));
    }
    Iterator<Employee> iterator = list.iterator();
    while (iterator.hasNext()){
        Employee employee = iterator.next();
        if(employee.age == employee.salary){
            iterator.remove();
        }
    }
	/**
     * 打印结果为空
     */
}

@Test
public void test5(){
    List<Employee> list = new ArrayList<>(16);
    for(int i = 0; i < 5; i++){
        list.add(new Employee("test"+i,i*2,i*3));
    }
    Iterator<Employee> iterator = list.iterator();
    while (iterator.hasNext()){
        Employee employee = iterator.next();
        if(employee.age == 0){
            iterator.remove();
            continue;
        }
        System.out.println(employee);
    }
    /**
     * 打印结果：
     * Employee{name='test1', age=2, salary=3}
     * Employee{name='test2', age=4, salary=6}
     * Employee{name='test3', age=6, salary=9}
     * Employee{name='test4', age=8, salary=12}
     */
}
```
#### 关于ArrayList的天坑
ArrayList中的每一个元素存储的实际上是对象引用（之前在公司写代码的时候，做过类似下面的事），假如按照下面的方式使用ArrayList，则最后list中存储的元素都相同且都是最后一个元素，原因是list中所有的元素都指向同一块内存。
```java
@Test
public void test1(){
    List<Person> list = new ArrayList<>(16);
    Person p = new Person();
    for(int i = 0; i < 10; i++){
        p.setName("test"+i);
        p.setAge(i);
        list.add(p);
    }
    System.out.println(list);
}
```

### Set接口

#### HashSet、LinkedHashSet、TreeSet

- HashSet是Set接口的典型实现，大多数时候Set集合都使用这个实现类。
	HashSet是按Hash算法来存储集合中的元素，因此具有很好的存取、查找、删除性能。底层就是HashMap;
	HashSet具有以下特点：不能保证元素的排列顺序；HashSet不是线程安全的；集合元素可以是null；
	HashSet集合判断两个元素相等的标准：两个对象通过hashCode()方法比较相等，并且两个对象的equals()方法返回值也相等。
	对于存放在Set容器中的对象，对应的类一定要重写equals()和hashCode()方法，以实现对象相等规则。既“相等的对象必须具有相同的hashCode”
- LinkedHashSet是HaseSet的子类
	LinkedHashSet根据元素的hashCode值来决定元素的存储位置，但它同时使用双向链表维护元素的次序，这使得元素看起来是以插入顺序保存的。
	LinkedHashSet插入性能略低于HashSet，但在迭代访问Set里的全部元素时有很好的性能。
	LinkedHashSet不允许集合元素重复。
- TreeSet是SortedSet接口的实现类，TreeSet可以确保集合元素处于排序状态。
	TreeSet底层就是TreeMap，而TreeMap底层使用红黑树结构（JDK1.8以后）存储key,TreeSet只是使用TreeMap的key
	TreeSet两种排序方法：自然排序和Comparetor比较器排序，默认使用自然排序。

#### HashSet
HashSet的底层初始化：
```java
private transient HashMap<E,Object> map;
public HashSet() {
    map = new HashMap<>();
}
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}
public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}
```
可以看到HashSet底层就是一个HashMap，那这里就不多关于HashMap的东西，主要看下对HashSet的测试:
```java
@Test
public void test6(){
    Set<Employee> set = new HashSet<>(10);
    System.out.println(set.size());
    for(int i = 0; i < 5; i++){
        set.add(new Employee("test",(i%2 == 0) ? 2 : 1,(i%2 == 0) ? 2 : 1));
    }
    Iterator<Employee> iterator = set.iterator();
    while (iterator.hasNext()){
        Employee employee = iterator.next();
        System.out.println(employee);
    }
    /**
     * 打印结果：
     * 0
     * run hashCode ... 
     * run hashCode ... 
     * run hashCode ... 
     * run equals()....
     * run hashCode ... 
     * run equals()....
     * run hashCode ... 
     * run equals()....
     * Employee{name='test', age=2, salary=2}
     * Employee{name='test', age=1, salary=1}
     * 2
     */
	/**
     * Employee的equals()重写时加了一句:
     * System.out.println("run equals()....");
     * Employee的hashCode()重写时加了一句:
     * System.out.println("run hashCode ... ");
     * 上面对List进行测试的时候也加了。。。
     */
}
```
从上面的打印结果可以看到，初始化后set的size为0，后面每次添加一个元素size都会增长1，而再添加过程中如果遇到hashCode相同情况下会进行equals判断，元素内的值相同会导致添加失败。所以最终set中只有2个元素。

#### LinkedHashSet

LinkedHashSet的底层初始化：
```java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
	public LinkedHashSet(int initialCapacity, float loadFactor) {
	    super(initialCapacity, loadFactor, true);
	}
	public LinkedHashSet(int initialCapacity) {
	    super(initialCapacity, .75f, true);
	}
	public LinkedHashSet() {
	    super(16, .75f, true);
	}
	public LinkedHashSet(Collection<? extends E> c) {
	    super(Math.max(2*c.size(), 11), .75f, true);
	    addAll(c);
	}
}

```
父类HashSet下的带有三个参数的构造器：
```java
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```
显然，LinkedHashSet底层是使用`LinkedHashMap`实现的，也就是说它带有`LinkedHashMap`的特点——可以找到添加过程中的前后元素。具体可以看<a href="#LinkedHashMap">LinkedHashMap</a>
#### TreeSet
TreeSet的底层初始化：
```java
public TreeSet() {
    this(new TreeMap<E,Object>());
}
public TreeSet(Comparator<? super E> comparator) {
    this(new TreeMap<>(comparator));
}
public TreeSet(Collection<? extends E> c) {
    this();
    addAll(c);
}
public TreeSet(SortedSet<E> s) {
    this(s.comparator());
    addAll(s);
}
```
使用TreeSet：
```java
@Test
public void test7(){
    Set<Employee> set = new TreeSet<>();
    for(int i = 0; i < 5; i++){
        set.add(new Employee("test",i,i));
    }
    Iterator<Employee> iterator = set.iterator();
    while (iterator.hasNext()){
        Employee employee = iterator.next();
        System.out.println(employee);
    }
    /**
     * 报错:
     * java.lang.ClassCastException: com.chapter12_collection.Employee cannot be cast to java.lang.Comparable
     *
     * 	at java.util.TreeMap.compare(TreeMap.java:1294)
     * 	at java.util.TreeMap.put(TreeMap.java:538)
     * 	at java.util.TreeSet.add(TreeSet.java:255)
     * 	at com.chapter12_collection.CollectionTest.test7(CollectionTest.java:115)
     * 	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
     * 	...
     */
}
```
发现必须得把Employee转成Comparable，那么就让Employee实现Comparable接口呗。于是乎重写里面的compareTo（）方法：
```java
// 先按name排序，再按age排序
@Override
public int compareTo(Object o) {
    if(o instanceof Employee){
        Employee employee = (Employee) o;
        if(this.name == employee.getName()){
            return Integer.compare(this.age, employee.getAge());
        }else {
            return this.name.compareTo(employee.getName());
        }
    }else {
        throw new RuntimeException("输入的类型不是Employee");
    }
}
```
再来跑测试代码：
```java
@Test
public void test7(){
    Set<Employee> set = new TreeSet<>();
    for(int i = 5; i > 0; i--){
        set.add(new Employee("test"+i,i,i));
    }
    Iterator<Employee> iterator = set.iterator();
    while (iterator.hasNext()){
        Employee employee = iterator.next();
        System.out.println(employee);
    }
    /**
     * 打印结果：
     * Employee{name='test1', age=1, salary=1}
     * Employee{name='test2', age=2, salary=2}
     * Employee{name='test3', age=3, salary=3}
     * Employee{name='test4', age=4, salary=4}
     * Employee{name='test5', age=5, salary=5}
     */
}
```
显然，此时是按照name从小到大排序的，而我们插入的时候是从大到小的，也就是说TreeSet是可以实现Comparable比较器排序的，关于自然排序就看下下面这段代码吧：
```java
@Test
public void test8(){
    Set<Integer> set = new TreeSet<>();
    set.add(11);
    set.add(-1);
    set.add(33);
    set.add(22);
    set.add(44);
    Iterator<Integer> iterator = set.iterator();
    while (iterator.hasNext()){
        System.out.print(iterator.next() + " ");
    }
    /**
     * 打印结果：
     * -1 11 22 33 44 
     */
}
```
## Map
Map接口主要HashMap、LinkedHashMap、TreeMap、Hashtable、Properties实现类

### Map的遍历方式
方式一,这是最常见的并且在大多数情况下也是最可取的遍历方式。在键值都需要时使用:
```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
  System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());
}
```

方式二，在for-each循环中遍历keys或values(据说该方法比entrySet遍历在性能上稍好（快了10%），我还没有测试过)：
```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
//遍历map中的键
for (Integer key : map.keySet()) {
  System.out.println("Key = " + key);
}
//遍历map中的值
for (Integer value : map.values()) {
  System.out.println("Value = " + value);
}
```
方式三，使用迭代器遍历（该种方式看起来冗余却有其优点所在。首先，在老版本java中这是惟一遍历map的方式。另一个好处是，你可以在遍历时调用iterator.remove()来删除entries，另两个方法则不能。）：
```java
// 不使用泛型
Map map = new HashMap();
Iterator entries = map.entrySet().iterator();
while (entries.hasNext()) {
  Map.Entry entry = (Map.Entry) entries.next();
  Integer key = (Integer)entry.getKey();
  Integer value = (Integer)entry.getValue();
  System.out.println("Key = " + key + ", Value = " + value);
}
```

```java
// 使用泛型
Map<Integer, Integer> map = new HashMap<>();
Iterator<Map.Entry<Integer, Integer>> iterator = map.entrySet().iterator();
while (iterator.hasNext()){
    Map.Entry<Integer, Integer> next = iterator.next();
    System.out.println(next);
}
```

方式四，通过键找值遍历（循环中通过key来拿到value效率低）：
```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
for (Integer key : map.keySet()) {
  Integer value = map.get(key);
  System.out.println("Key = " + key + ", Value = " + value);
}
```
### HashMap
HashMap初始化：
```java
static final int MAXIMUM_CAPACITY = 1 << 30;
static final float DEFAULT_LOAD_FACTOR = 0.75f;
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}

static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```
显然，在初始化时如果没有传入Map的大小，则只是改了下了一个加载因子；而在传入initialCapacity时则会计算出threshold（临界值），用于判断是否进行扩容。然后在使用put()的时候才会真正创建hashTable：
```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
/**
 * Associates the specified value with the specified key in this map.
 * If the map previously contained a mapping for the key, the old
 * value is replaced.
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 * @return the previous value associated with <tt>key</tt>, or
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
 *         (A <tt>null</tt> return can also indicate that the map
 *         previously associated <tt>null</tt> with <tt>key</tt>.)
 */
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

/**
 * Implements Map.put and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to put
 * @param onlyIfAbsent if true, don't change existing value
 * @param evict if false, the table is in creation mode.
 * @return previous value, or null if none
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
		// 此时p == tab[i = (n - 1) & hash]
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
			// 向红黑树中插入结点
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) { //没有终止条件的循环用于查看该索引上所有的node结点
                if ((e = p.next) == null) { //下一个为空时新建一个结点
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 此时需要进行替换value
					break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key，key的hash值发生冲突
            V oldValue = e.value;
			// key的hash发生冲突后必然会进行一次替换（onlyIfAbsent默认为false）
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
/**
 * Initializes or doubles table size.  If null, allocates in
 * accord with initial capacity target held in field threshold.
 * Otherwise, because we are using power-of-two expansion, the
 * elements from each bin must either stay at same index, or move
 * with a power of two offset in the new table.
 *
 * @return the table
 */
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY; //默认16
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY); // 默认0.75*16 = 12
    }
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
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
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
```
从上面的代码以及部分我自己标注的中文注释可以看到：默认是创建了一个长度为16的表`(Node<K,V>[])new Node[newCap]`；其临界值`threshold`为12。在后面每次调用put操作时也是调用上面三个方法。
当然，上面的代码也是可以看出，put首先调用key1所在类的hashCode()计算key1的哈希值，过程如下：
```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
此哈希值经过`i = ((tab = resize()).length - 1) & hash`后获得插入到哈希数组中的index，在put过程中会有如下情况：
1. 如果此位置的数据为空，此时的key1-value1添加成功。
2. 如果此位置的数据不为空，（意味这此位置有一个或者多个数据存在），比较key1和已经存在数据的哈希值：
	1. 如果key1的哈希值与已经存在的数据的哈希值都不相同，此时key1-value1添加到第一个位置，后续为原来的链表数据。
	2. 如果key1的哈希值与已经存在的的某个数据(key2-value2)的hash值相同，继续比较；调用key对象的equals()方法，
		1. 如果equals()返回false：，此时key1-value1添加到第一个位置，后续为原来的链表数据。
		2. 如果equals()返回true:此时使用value1替换value2并返回value2
		具体比较代码如下：
		```java
		if (e.hash == hash && 
			((k = e.key) == key || (key != null && key.equals(k))))
		```

当然，在这里还是得说下jdk1.8相较与jdk1.7在底层实现方面还是的一些不同：
1. 初始化时jdk1.7时直接创建了一个长度为16的数组，而1.8没有。
2. jdk1.8底层的数组时Node[]而非Entry[],当他们实际是差不多的。
3. jdk1.8是在首次调用put()方法的时候创建数组。
4. jdk1.7的底层结构只有数组+链表；jdk1.8的底层结构为数组+链表+红黑树
	当数组的某一个索引位置上的元素以链表的形式存在的数据个数>8且当前数组的长度>64时，此时此索引位置上的所有数据改为使用红黑树存储。

### <p name="LinkedHashMap">LinkedHashMap</p>
LinkedHashMap初始化：
```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
{
	public LinkedHashMap(int initialCapacity, float loadFactor) {
	    super(initialCapacity, loadFactor);
	    accessOrder = false;
	}
	public LinkedHashMap(int initialCapacity) {
	    super(initialCapacity);
	    accessOrder = false;
	}
	public LinkedHashMap() {
	    super();
	    accessOrder = false;
	}
	public LinkedHashMap(Map<? extends K, ? extends V> m) {
	    super();
	    accessOrder = false;
	    putMapEntries(m, false);
	}
	public LinkedHashMap(int initialCapacity,
	                         float loadFactor,
	                         boolean accessOrder) {
	    super(initialCapacity, loadFactor);
	    this.accessOrder = accessOrder;
	}
}
```
在初始化的过程中，我们发现LinkedHashMap就是HashMap，那么它和HashMap有什么区别呢，我们还是得看它的put()方法，然后在LinkedHashMap中却没有找到put()方法，这个时候怎么办呢？看它的父类，查看父类中的put()方法：
```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```
发现还是得找putVal方法，于是接着看：
```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
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
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```
初始化的过程中我们必定java必定会执行：`tab[i] = newNode(hash, key, value, null);`这句代码，于是去看newNode,点进去，发现HashMap中的newNode方法是这样子的：
```java
Node<K,V> newNode(int hash, K key, V value, Node<K,V> next) {
    return new Node<>(hash, key, value, next);
}
```
那LinkedHashMap怎么和HashMap一样啊，这个时候就该思考，会不会是子类中还有什么是我们没有看到的，接着看子类LinkedHashMap，发现LinkedHashMap中也有newNode方法，原来是重写了：
```java
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}
```
再接着点进Entry看看是什么东西：
```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```
哦，原来它生成了一个和HashMap一样的结点，还在`Entry<K,V> before, after;`这里使用了before和after去记录生成的结点的前后顺序。

### TreeMap
> A Red-Black tree based NavigableMap implementation. The map is sorted according to the Comparable natural ordering of its keys, or by a Comparator provided at map creation time, depending on which constructor is used.

上面这段话是java官方给出的说明，显然，TreeMap底层就是使用红黑树来实现的，并且是一个可以排序的Map，其中既可以使用自然排序，也可以是使用Comparator比较器来进行排序。
自然排序：
```java
@Test
public void test9(){
    Map<Integer, Object> map = new TreeMap<>();
    map.put(11, "dasd");
    map.put(-1, "dasda");
    map.put(22, "dasd");
    map.put(33, "dasd");
    map.put(44, "dasd");
    map.put(44, "dad");
    Iterator<Map.Entry<Integer, Object>> iterator = map.entrySet().iterator();
    while (iterator.hasNext()){
        Map.Entry<Integer, Object> next = iterator.next();
        System.out.println(next);
    }
    /**
     * 打印结果：
     * -1=dasda
     * 11=dasd
     * 22=dasd
     * 33=dasd
     * 44=dad
     */
}
```
Comparetor比较器排序：
```java
@Test
public void test10(){
    Map<Employee, Object> map = new TreeMap<>();
    for(int i = 5; i > 0; i--){
        map.put(new Employee("test"+i, i, i), "dasd");
    }
    Iterator<Map.Entry<Employee, Object>> iterator = map.entrySet().iterator();
    while (iterator.hasNext()){
        Map.Entry<Employee, Object> next = iterator.next();
        System.out.println(next);
    }
    /**
     * 打印结果：
     * Employee{name='test1', age=1, salary=1}=dasd
     * Employee{name='test2', age=2, salary=2}=dasd
     * Employee{name='test3', age=3, salary=3}=dasd
     * Employee{name='test4', age=4, salary=4}=dasd
     * Employee{name='test5', age=5, salary=5}=dasd
     */
}
```

### Properties
Properties利用了HashMap通过key查找迅速的特点，将这一特点用在了读取配置文件上，具体的使用如下：
在项目的资源文件夹下新建test.properties文件：
```xml
name=1baidu
```
测试代码：
```java
@Test
public void test1() throws Exception{
    Properties properties = new Properties();
    FileInputStream fileInputStream= new FileInputStream("test.properties");

    properties.load(fileInputStream);
    System.out.println("name="+properties.getProperty("name"));
	/**
     * 打印结果：
     * name=1baidu
     */
}
```