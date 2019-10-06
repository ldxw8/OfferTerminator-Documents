# 技术面试必备基础知识-Java

> 在线阅读：[CS-Notes-Java](https://cyc2018.github.io/CS-Notes/#/README?id=%e2%98%95%ef%b8%8f-java)

## Java 基础

## Java 容器
### 容器概览
- 在另一则笔记 [Kofe | Java 技术手册 | 使用 Java 集合](https://www.kofes.cn/2017/09/Java-in-a-Nutshell.html#介绍集合-API) 中有对 Java 容器的概述，为建立宏观认识建议阅读。

### 容器的设计模式
#### 迭代器模式
- Collection 继承了 Iterable 接口，其中的 `iterator()` 方法能够产生一个 Iterator 对象，通过这个对象就可以迭代遍历 Collection 中的元素。
- 从 JDK 1.5 之后可以使用 `foreach()` 方法来遍历实现了 Iterable 接口的聚合对象
	
	```java
	List<String> list = new ArrayList<>();
	list.add("a");
	list.add("b");
	
	for (String str : list) {
	    System.out.println(item);
	}
	```

#### 适配器模式
- `java.util.Arrays` 的 asList() 可以把数组类型转换为 List 类型。

	```java
	@SafeVarargs
	public static <T> List<T> asList(T... a) {
		return new ArrayList<>(a);
	}
	```
	
- 应该注意的是 asList() 的参数为泛型的变长参数，不能使用 `基本类型` 数组作为参数，只能使用相应的 `包装类型` 数组。

	```java
	Integer[] arr = {1, 2, 3};
	List list = Arrays.asList(arr);
		
	// 或者直接传入参数
	List list = Arrays.asList(1, 2, 3);
	```


### 容器的源码解析
#### ArrayList
>  以下源码细节是基于 JDK 1.8 版本展开讨论的。

| ![CyC2018-CS-Notes-Java-ArrayList_1-1](img/CyC2018-CS-Notes-Java-ArrayList_1-1.png)|
| :---: |
| 图 2-1 ArrayList 概述 |

##### 参考资料
- [细雨蒙情. ArrayList (JDK1.8) 源码分析. jianshu.com](https://www.jianshu.com/p/ea4f943206ea)
- [十二页. ArrayList (JDK1.8/JDK1.7/JDK1.6). csdn.net](https://blog.csdn.net/u011392897/article/details/57105709)
- [YSOcean. ArrayList (JDK1.8). cnblogs.com](https://www.cnblogs.com/ysocean/p/8622264.html)

##### 基本性质
- 因为底层数组 elementData 的容量是不能改变的，为此容量不够时需要把 elementData 换成一个更大的数组，这个过程叫作 `扩容`。
-  ArrayList 所有方法都没有进行同步，它是 `线程不安全` 的。为此在多线程并发读写时需要外部同步。

	可使用 `Collections.synchronizedList()` 方法对 ArrayList 的实例进行封装。

	```java
	List list = Collections.synchronizedList(new ArrayList(...))
	```
	
- 对存储的元素无限制，允许 `null` 元素。

##### 源码细节
- 通过 ArrayList 实现的接口可知，其支持快速随机访问、能被克隆以及支持序列化 (将对象转换为字节流的过程)。

	且 ArrayList<E> 支持泛型，即支持构造任何类型的动态数组。

	```java
	public class ArrayList<E> extends AbstractList<E> 
		implements List<E>, RandomAccess, Cloneable, Serializable {
		// 忽略代码细节...
	}
	```

- 关键成员变量与构造函数解析：

	```java
	// 默认初始化容量
	private static final int DEFAULT_CAPACITY = 10;

	// 当用户指定该 ArrayList 容量为 0 时返回该空数组
	private static final Object[] EMPTY_ELEMENTDATA = {};
	
	// 当调用 ArrayList 的无参构造函数时返回该空数组
	private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
	
	// 数组对象 (非私有以简化嵌套类访问)
	transient Object[] elementData;     

	// 数组中元素的个数 (注意不是数组的长度)
	private int size;

	// 不带参数的构造方法
	public ArrayList() {
		super();
		// 将空的数组实例传给elementData
		this.elementData = EMPTY_ELEMENTDATA;
	}
	
	// 传入初始容量的构造方法
	public ArrayList(int initialCapacity) {
		super();
		if (initialCapacity < 0) {
			throw new IllegalArgumentException(
				"Illegal Capacity: "+ initialCapacity);
		}
		// 新建指定容量的 Object 类型数组
		this.elementData = new Object[initialCapacity];
	}

	// 传入外部集合的构造方法
	public ArrayList(Collection<? extends E> c) {
		// 持有传入集合的内部数组的引用
		elementData = c.toArray();

		// 更新集合元素个数大小
		if ((size = elementData.length) != 0) {
			// 判断引用的数组类型, 并将引用转换成 Object 数组引用
			if (elementData.getClass() != Object[].class) {
				elementData = Arrays
					.copyOf(elementData, size, Object[].class);
			}
		} else {
			// 使用空数组来替代
			this.elementData = EMPTY_ELEMENTDATA;
		}
	}
	```

##### 增删改查
- 增：
	- 添加时先检查容量是否足够，否则就进行扩容；
	- 添加元素到末尾。
- 插：
	- 插入位置安全性检查；
	- 容量检查；
	- (将插入位置后面的元素往后挪动)，插入元素。
- 改：直接对指定元素进行修改。
- 删：
	- 删除位置安全性检查；
	- (将删除位置后面的元素向前挪动)，删除元素。
- 查：直接返回指定下标的数组元素。

##### 动态扩容
- 扩容实际是新建一个容量更大 (原来数组长度 1.5 倍) 的数组。

	```java
	public boolean add(E e) {
		// 动态扩容的关键
		ensureCapacityInternal(size + 1);
		// 在 size+1 的位置进行赋值
		elementData[size++] = e;
		return true;
	}
	
	private void ensureCapacityInternal(int minCapacity) {
		// 如果此时还是空数组
		if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
			// 和默认容量比较， 取较大值
			minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
		}
		//数组已经初始化过就执行这一步
		ensureExplicitCapacity(minCapacity);
	}

	private void ensureExplicitCapacity(int minCapacity) {
		modCount++;
		//如果最小容量大于数组长度就扩增数组
		if (minCapacity - elementData.length > 0) {
			grow(minCapacity);
		}
	}

	// 集合最大容量
	private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
	
	// 增加数组长度
	private void grow(int minCapacity) {
		// 获取数组原先的容量
		int oldCapacity = elementData.length;
		
		// 新数组的容量, 在原来的基础上增加一半
		// 1) JDK 1.6 的计算公式是：(oldCapacity * 3) / 2 + 1
		// 若 oldCapacity = 10^9，那么上述计算结果会越界 -647483647
		// 2) JDK 1.8 / 1.7 是以下计算公式：
		int newCapacity = oldCapacity + (oldCapacity >> 1);
		
		// 检验新的容量是否小于最小容量
		if (newCapacity - minCapacity < 0) {
			newCapacity = minCapacity;
		}
		
		// 检验新的容量是否超过最大数组容量
		if (newCapacity - MAX_ARRAY_SIZE > 0) {
			newCapacity = hugeCapacity(minCapacity);
		}
		
		// 拷贝原来的数组到新数组
		elementData = Arrays.copyOf(elementData, newCapacity);
	}
	
	private static int hugeCapacity(int minCapacity) {
		if (minCapacity < 0) throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ? 
        	Integer.MAX_VALUE : MAX_ARRAY_SIZE;
	}
	```

	> 实际扩容可能不止 1.5 倍，只是 grow() 函数执行一趟扩容的大小为 1.5 倍，若容量还不足需则继续执行 grow() 扩容。

- 将原先数组的元素全部复制到新数组上， 然后舍弃原先的数组转而使用新数组。

	> Arrays.copyOf() 把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。

##### 总结
- 使用 ArrayList 时尽量指定大小，以减少扩容带来的 `数组复制操作` 。
- 每次添加元素 --> 检查容量 --> 是否扩容 --> 扩大 1.5 倍。
- 每次下标操作 --> 安全性检查 --> 数组越界就立即抛出异常。
- ArrayList 所有方法都没有进行同步，因此它是线程不安全的。

#### LinkedList
>  以下源码细节是基于 JDK 1.8 版本展开讨论的。

| ![CyC2018-CS-Notes-Java-LinkedList_1-1](img/CyC2018-CS-Notes-Java-LinkedList_1-1.png) |
| :---: |
| 图 2-2 LinkedList 概述 |

##### 参考资料
- [十二页. LinkedList (JDK1.8/JDK1.7/JDK1.6). csdn.net](https://blog.csdn.net/u011392897/article/details/57115818)
- [YSOcean. LinkedList (JDK1.8). cnblogs.com](https://www.cnblogs.com/ysocean/p/8657850.html)

#####  基本性质
- JDK 1.7 之后使用的是不带头结点的普通的双向链表，增加两个节点指针 first、last 分别指向首尾节点，示意图如下：

	| ![CyC2018-CS-Notes-Java-LinkedList_1-2](img/CyC2018-CS-Notes-Java-LinkedList_1-2.png) |
	| :----------------------------------------------------------: |
	|                图 2-2-1 不带头结点的双向链表                 |
	
-  与 ArrayList 一样，LinkedList 所有方法都没有进行同步，它是 `线程不安全` 的。为此在多线程并发读写时需要外部同步。

	可使用 `Collections.synchronizedList()` 方法对 LinkedList 的实例进行封装。

	```java
	List list = Collections.synchronizedList(new LinkedList(...))
	```
	
- 相比 ArrayList 此类还额外实现了 `Deque` 接口，可以充当一般的 `双端队列` 或者 `栈`，可以作为一些混合数据结构的基础。
- 理论上无容量限制，只受虚拟机自身限制影响，所以没有扩容方法。
- 对存储的元素无限制，允许 `null` 元素。

##### 源码细节
- 通过 LinkedList 实现的接口可知，其支持随机访问、能被克隆以及支持序列化 (将对象转换为字节流的过程)。

	且 LinkedList<E> 支持泛型，即支持构造任何类型的动态数组。

	```java
	public class LinkedList<E> extends AbstractSequentialList<E>
		implements List<E>, Deque<E>, Cloneable, Serializable {
		// 忽略代码细节...
	}
	```

- 关键成员变量与构造函数解析：

	```java
	// 由于 JDK 1.7 开始不再使用 header 节点
	// 因此默认构造方法不做声明，first 和 last 会被默认初始化为 null
	
	// 链表元素 (节点) 的个数
	transient int size = 0;

	/**
	 * 头节点引用
	 * 这是个固定不变的关系: 
	 * (first == null && last == null) || (first.prev == null && first.item != null) 
	 */
	transient Node<E> first;
 
	/**
	 * 尾节点引用
	 * 这是个固定不变的关系:
	 *  (first == null && last == null) || (last.next == null && last.item != null) 
	 */
	transient Node<E> last;
 
	// 默认构造方法 (无参构造方法)
	public LinkedList() {}
	
	public LinkedList(Collection<? extends E> c) {
		this();
		addAll(c);
	}
	```

- 结点由内部类 Node 表示：

	```java
	private static class Node<E> {
		E item; // 当前节点值
		Node<E> next; // 下一个节点
		Node<E> prev; // 上一个节点

		Node(Node<E> prev, E element, Node<E> next) {
			this.item = element;
			this.next = next;
			this.prev = prev;
		}
	}
	```
	
##### 增删改查
- 增：
	- 添加：在链表尾部插入 -- `linkLast()`
	- 插入：在链表中部插入 -- `linkBefore()`
- 删：
	- 给定数组下标：检查下标是否合法；删除目标节点。
	- 给定数据元素：检索链表中是否含有目标元素，有则删除目标结点。
- 查：
	- 检查下标是否合法；
	- 返回指定下标的结点的值。

		> 使用迭代器和 for 循环是存在差距的，不妨通过实现验证效果。注意迭代器的另一种形式就是使用 foreach 循环，底层实现也是使用的迭代器。
		
		```java
		LinkedList<Integer> linkedList = new LinkedList<>();
		// 向链表中添加 10 万个元素
		for(int i = 0 ; i < 100000 ; i++){
		    linkedList.add(i);
		}
		long beginTimeFor = System.currentTimeMillis();
		for(int i = 0 ; i < 100000 ; i++){
		    System.out.print(linkedList.get(i));
		}
		long endTimeFor = System.currentTimeMillis();
		System.out.println("\nFor 循环: " + (endTimeFor - beginTimeFor));
		
		long beginTimeIte = System.currentTimeMillis();
		Iterator<Integer> it = linkedList.listIterator();
		while(it.hasNext()){
		    System.out.print(it.next());
		}
		long endTimeIte = System.currentTimeMillis();
		System.out.println("\n迭代器: "+ (endTimeIte - beginTimeIte));		
		```

##### 单向队列/双向队列/栈
- 其实都是对链表的头结点和尾结点进行操作，它们都是基于以下方法实现操作：
	- 对链表两端操作：addFirst()、addLast()、reoveFirst()、removeLast()
	- 对链表中间操作：linkBefore()、unlink()

##### 总结
- LinkedList 是基于 `双向链表` 实现的，不论是增删改方法，还是栈、队列的实现，都可通过 `操作结点` 实现。
- 由于链表结构对内存要求低，LinkedList 不需要大块连续内存来满足扩容，能够自动地、动态地消耗内存；容量变小时会 `自动释放` 曾占用的内存。
- LinkedList 的所有方法都没有进行同步，它是 `线程不安全` 的。

#### HashMap
>  以下源码细节是基于 JDK 1.8 版本展开讨论的。

| ![CyC2018-CS-Notes-Java-HashMap_1-1](img/CyC2018-CS-Notes-Java-HashMap_1-1.png) |
| :---: |
| 图 2-3 HashMap 概述 |

##### 参考资料
- [十二页. HashMap JDK1.8 源码分析. csdn.net](https://blog.csdn.net/u011392897/article/details/60151323)
- [多读书多看报. HashMap JDK1.8 实现原理. cnblogs.com](https://www.cnblogs.com/duodushuduokanbao/p/9492952.html)
- [YSOcean. HashMap (JDK1.8). cnblogs.com](https://www.cnblogs.com/ysocean/p/8711071.html)

##### 基本性质
- HashMap 存储的是 key-value 的键值对，允许 key 为 null，也允许 value 为null。
- HashMap 内部为数组和链表的组合结构。若 Hash 冲突的概率比较高，就会导致同一个桶中的链表长度过长、遍历效率降低。

	> 通过源码可知：以内部类 Node 表示结点，存储着键值对，且包含了四个字段。其中 next 字段我们可以看出 Node 会相链组成一个链表。即数组中的每个位置被当成一个桶，一个桶存放一个链表。

	故在 JDK 1.8 中如果链表长度到达阀值 (默认是8)，就会将链表转换成 `红黑二叉树` 保存，以提高 Hash 冲突时的查找速度。整体结构如图 2-3-1 所示：

	| ![CyC2018-CS-Notes-Java-HashMap_1-2](img/CyC2018-CS-Notes-Java-HashMap_1-2.png) |
	| :---: |
	| 图 2-3-1 HashMap 的整体结构 |

- 链表添加时，新节点会放在链表末尾，而不是像 JDK 1.6/1.7 一样放在头部；
- 扩容操作也会尽量保证扩容后还在同一条链表上的节点之间的 `相对顺序` 不变。

##### 源码细节
- 关键成员变量与构造函数解析：

	```java
	/* 常量 */
	
	// 默认 Hash 表的初始容量
	static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
	// 默认 Hash 表最大容量
	static final int MAXIMUM_CAPACITY = 1 << 30;
	// 默认加载因子 (指哈希表可满足)
	static final float DEFAULT_LOAD_FACTOR = 0.75f;
	
	/* 新增红黑树有关的三个常量 */
 
	// 若一个 Hash 桶中的节点达到这个值，下次添加新节点时
	// 会把这个 Hash 桶的所有节点用红黑树保存
	// 若数组 table 的长度不足 64，那么也不转化为红黑树，改为扩容一次
	static final int TREEIFY_THRESHOLD = 8;
 
	// 若一棵红黑树的节点减少到这个值，那么就把它退化为链表保存
	static final int UNTREEIFY_THRESHOLD = 6;
 
	/**
	 * 转化为红黑树的另一个条件：
	 * 1) 当集合中的容量大于这个值时，表中的桶才能进行树形化，
	 * 否则桶内元素太多时会扩容，而不是树形化。
	 * 2) 为了避免进行扩容、树形化选择的冲突，
	 * 这个值不能小于 4 * TREEIFY_THRESHOLD。
	 */
	static final int MIN_TREEIFY_CAPACITY = 64;
	
	/* 变量 */
	
	// 初始化使用 (长度总是 2 的幂)
	transient Node<K,V>[] table;
	
	// 保存缓存的 entrySet()
	transient Set<Map.Entry<K,V>> entrySet;
	
	// 此映射中包含的键值映射的数量 (集合存储键值对的数量)
	transient int size;
	
	/**
	 * 跟前面ArrayList和LinkedList集合中的字段modCount一样，
	 * 记录集合被修改的次数 (主要用于迭代器中的快速失败)
	 */
	transient int modCount;
	
	/**
	 * threshold = Hash 表的初始容量 * 加载因子
	 * 当键值对的数量要超过阈值，即意味哈希表已处于饱和状态
	 * 若继续添加元素只会增加哈希冲突，使得 HashMap 性能下降
	 */
	int threshold;
	
	// 哈希表的加载因子
	final float loadFactor;
	```

- 哈希表其实是一个 Node 数组，数组中每个位置存放着单向链表的头结点。

	结点由内部类 Node 表示，且 Node 本质上是一个 Map，存储着键值对 (key-value)：

	> 注意：JDK 1.6  的结点名称是 Entry (仅命名不同)。

	- 普通结点 Node：

		```java
		static class Node<K,V> implements Map.Entry<K,V> {
			// 保存该桶的 hash 值
			// hash 值又变回 final (1.7 不是 final，1.6 是 final)
			final int hash; 
			final K key;
			V value;
			Node<K,V> next;
		
			Node(int hash, K key, V value, Node<K,V> next) {
				this.hash = hash;
				this.key = key;
				this.value = value;
				this.next = next;
			}
		}
		```
	
	- 红黑树结点 TreeNode：篇幅缘故可参考内部类 `TreeNode` 中的源码。

##### 增删改查
- 遍历元素：

	```java
	// 首先构造一个 HashMap 集合
	HashMap<String,Object> map = new HashMap<>();
	map.put("A","1");
	map.put("B","2");
	map.put("C","3");
	
	// 方法一：分别获取 key 集合和 value 集合
	// 根据 key 分别得到相应 value，
	// 遍历效率最低，适合单独取 key 或 value 时使用
	for(String key : map.keySet()){
	    System.out.println(key);
	}
	for(Object value : map.values()){
	    System.out.println(value);
	}

	// 方法二：得到 Entry 集合，然后遍历 Entry
	Set<Map.Entry<String,Object>> entrySet = map.entrySet();
	for(Map.Entry<String,Object> entry : entrySet){
		System.out.println( entry.getKey() );
		System.out.println( entry.getValue() );
	}

	// 方法三：迭代
	// 效率较前者高，且在遍历的过程中支持对集合中的元素进行删除
	Iterator<Map.Entry<String,Object>> iterator = map.entrySet().iterator();
	while(iterator.hasNext()){
	    Map.Entry<String,Object> mapEntry = iterator.next();
	    System.out.println( mapEntry.getKey() );
	    System.out.println( mapEntry.getValue() );
	}
	```

##### 实现原理
> 以下将针对具体问题出发，去探究 HashMap 的原理。

- HashMap 在构造器中做了哪些操作：
	- 设置加载因子 `localFacor`；
	- 设置键值对的容纳阈值 `threshold  = tableSizeFor(initalCapacity)`。

		```java	
		// 求不小于 cap 且满足 2^n 的数中最小的一个 (扩容也是 2^n 进行的)
		// 这个方法的原型就是 JDK 1.7 中的 Integer.highestOneBit 方法
		static final int tableSizeFor(int cap) {
			int n = cap - 1;
			n |= n >>> 1;
			n |= n >>> 2;
			n |= n >>> 4;
			n |= n >>> 8;
			n |= n >>> 16;
			return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ?
				MAXIMUM_CAPACITY : n + 1;
		}
		```

- 确定哈希桶数据索引位置 (计算 Hash 码)：
	- 定位数组下标是根据 Hash 码的低位值确定的；
	- key 的 Hash 码由 hashCode() 方法计算所得。

		```java
		/* JDK 1.8 -- 计算 Hash 码 */
		// 因为 JDK 1.8 使用了红黑树来保存冲突节点
		// 冲突的代价变小，Hash 函数只需移位异或一次
		static final int hash(Object key) {
			int h;
			h = key.hashCode();
			return (key == null) ? 0 : h ^ (h >>> 16);
		}
		
		/* JDK 1.7 -- 计算 Hash 码 */
		final int hash(Object k) {
			int h = hashSeed;
			// key 是 String 类型的就使用另外的哈希算法
			if (0 != h && k instanceof String) {
				return sun.misc.Hashing.stringHash32((String) k);
			}
			h ^= k.hashCode();
			// 扰动函数
			h ^= (h >>> 20) ^ (h >>> 12);
			return h ^ (h >>> 7) ^ (h >>> 4);
		}
		```
		
	- 把 Hash 码再代入求模公式中计算数组下标，即哈希桶数据索引位置：

		```java
		/* JDK 1.8 -- 计算数组下标 */
		// 取模运算已经整合于代码当中，例如：tab[hash & (n - 1)]
		
		/* JDK 1.7 -- 计算数组下标 */
		static int indexFor(int h, int length) {
			// 该项位运算等价于取模运算
			return h & (length-1);  
		}
		```
		
		| ![CyC2018-CS-Notes-Java-HashMap_1-3](img/CyC2018-CS-Notes-Java-HashMap_1-3.png) |
		| :---: |
		| 图 2-3-2 h & (length-1) 的计算原理 |
		
		> 注意：`h & (length-1)`  但要求 Entry 数组的长度大小满足 2 的幂。   
		> 原理：去掉 h 的高位值，只保留 h 的低位值作为数组下标。

-  HashMap 添加键值对时会进行什么操作：
	- Step.01：先检查哈希表 table 是否为空表，若为空表则先创建一个 table；
	- Step.02：计算 Hash 码 `hash(key)` --> 计算索引位置 `hash % (n-1)` --> 确定插入哈希数组的下标位置。
	- Step.03：计算出来的索引位置之前没有放过数据，则直接放入数据；
	- Step.04：若该索引位置不为空：
		- 1) 先判断索引第一个 key 与换入 key 是否相等，重复则直接覆盖；
		- 2) 再判断是否为红黑树，若是红黑树则直接插入树中；

			> if ( p instanceof TreeNode ) { ... }
			
		- 3) 若不是红黑树，就遍历每个结点，当链表长度大于 8 时则转换为红黑树，再插入结点；
		- 4) 在插入结点前，判断 put 的数据和之前是否重复，重复则覆盖原有 key 的 value，并返回原有 value。若 key 不相同，则插入一个 key，记录结构变化一次。

- HashMap 取键值对时会进行什么操作：
	- 判断哈希表 table 是否为空表，若为空表则返回 `null`；
	- 判断索引处第一个 key 与传入 key 是否相等，若相等直接返回 Node；
	- 若不相等，判断链表是否为红黑树，若是则直接从树中取值；否则就遍历链表取值。

##### 扩容机制
- 使用的 2 的幂的扩容 (指长度扩为原来的 2 倍)，故元素的位置要么在原位置，要么在原位置再移到 2 的幂的位置。
- 假设 `oldCapacity = 2^4 = 16` 与 `newCapacity = 2^5 = 32`。`x` 代表不用管，`?` 代表 0 或者 1 任意，二进制下标从最右边的第 0 位开始计算。

	如图 1-4 所示，我们从 B 中可以看出，当触发扩容时 oldCapacity  变为 newCapacity，(newCapacity - 1) 的 Mask 掩码范围在高位上多了 1 bit。
	
	- JDK 1.8 版本改进的结点迁移方式非常巧妙，其不仅保证了相对顺序不改变，且省去了重新计算 Hash 码的时间。

		> JDK 1.6/1.7 版本的结点迁移方式是 `遍历链表`，即一个一个重新添加到新链表的头部，会颠倒原来链表中结点的 `相对顺序`。

	- 因为新增的 1 bit 是 1 或者 0，则可以认为是随机的过程。因此在扩容 resize() 的执行过程，则可以把之前冲突的结点均匀地分散到新的哈希桶中，如图 2-3-4 所示。

	| ![CyC2018-CS-Notes-Java-HashMap_1-4](img/CyC2018-CS-Notes-Java-HashMap_1-4.png) |
	| :---: |
	| 图 2-3-3 扩容机制中的结点迁移方式探究 |
	
	| ![CyC2018-CS-Notes-Java-HashMap_1-5](img/CyC2018-CS-Notes-Java-HashMap_1-5.png) |
	| :---: |
	| 图 2-3-4 分布均匀的哈希桶 |

##### 总结
- 基于 JDK 1.8 的 HashMap 是由 `数组+链表+红黑树` 组成，当链表长度超过 8 时会自动转换成红黑树，当红黑树节点个数小于 6 时，又会转化成链表。

	相对于 HashMap 早期版本的 JDK 实现，新增 `红黑树` 作为底层数据结构，在数据量较大且哈希碰撞较多时，能够极大的增加检索的效率。
	
- 允许 key 和 value 都为 null。key 重复会被覆盖，value 允许重复。

	> (null, null), (“name”, null), (“sex”, null), 

- HashMap 的所有方法都没有进行同步，它是 `线程不安全` 的。
- 遍历 HashMap 得到元素的顺序不是按照插入的顺序输出的。

#### ConcurrentHashMap

## Java 并发
### 线程状态
-  Java 语言定义了 5 种线程状态，在任意时间点，一个线程有且只能拥有一种状态。

	| ![线程状态转换](img/CyC2018-CS-Notes-Java-_3-1.png) |
	| :-: |
	| 图 3-1 线程状态转换 |

#### 新建 / New
- 创建后尚未启动的线程。

#### 运行 / Runable
- 可能正在运行，也可能正在等待 CPU 时间片。包含了操作系统线程状态中的 Running 和 Ready。

#### 阻塞 / Blocked
- 等待获取一个排它锁，如果其线程释放了锁就会结束此状态。

#### 无限期等待 / Waiting 
- 处于这种状态的线程不会被分配 CPU 执行时间，需等待其它线程 `显式地` 唤醒，否则不会被分配 CPU 时间片。

  | 进入方法 | 退出方法 |
  | :--- | :--- |
  | 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
  | 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕 |
  | LockSupport.park() 方法 | LockSupport.unpark(Thread) |

#### 限期等待 / Timed Waiting
- 处于这种状态的线程不会被分配 CPU 执行时间，不过无需等待其它线程显式地唤醒，在一定时间之后会被 `系统自动唤醒`。

	| 进入方法 | 退出方法 |
	| :--- | :--- |
	| Thread.sleep() 方法 | 时间结束 |
	| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
	| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕 |
	| LockSupport.parkNanos() 方法 | LockSupport.unpark(Thread) |
	| LockSupport.parkUntil() 方法 | LockSupport.unpark(Thread) |
	
	- 调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用使一个线程 `睡眠` 进行描述。
	- 调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用 `挂起` 一个线程进行描述。
	- 睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。
	- 阻塞和等待的区别在于，`阻塞` 是 `被动` 的，它是在等待获取一个排它锁。而 `等待` 是 `主动` 的，通过调用 Thread.sleep() 和 Object.wait() 等方法进入。
	
#### 死亡 / Terminated
- 可以是线程结束任务之后自己结束，或者产生了异常而结束。

### 使用线程
- Java 中有三种使用线程的方法： 实现 `Runnable` 接口；实现 `Callable` 接口；继承 `Thread` 类。

	> 实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程。因此最后还需要通过 Thread 来调用，可以说任务是通过线程驱动从而执行的。

#### 实现 Runnable 接口

- 需要实现 run() 方法；通过 Thread 调用 start() 方法来启动线程。

	```java
	public class MyRunnable implements Runnable {
	    public void run() {
	        // ...
	    }
	}
	
	public static void main(String[] args) {
	    MyRunnable instance = new MyRunnable();
	    Thread thread = new Thread(instance);
	    thread.start();
	}
	```

#### 实现 Callable 接口
- 与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。

	```java
	public class MyCallable implements Callable<Integer> {
	    public Integer call() {
	        return 123;
	    }
	}

	public static void main(String[] args) 
	    throws ExecutionException, InterruptedException {
	    MyCallable mc = new MyCallable();
	    FutureTask<Integer> ft = new FutureTask<>(mc);
	    Thread thread = new Thread(ft);
	    thread.start();
	    System.out.println(ft.get());
	}
	```

#### 继承 Thread 类
- 同样也是需要实现 run() 方法，因为 Thread 类也实现了 Runable 接口。

	当调用 start() 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法。

	```java	
	public class MyThread extends Thread {
	    public void run() {
	        // ...
	    }
	}
	
	public static void main(String[] args) {
	    MyThread mt = new MyThread();
	    mt.start();
	}
	```

#### 实现接口 / 继承 Thread
- Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
- 类可能只要求可执行就行，继承整个 Thread 类开销过大。

	> 为此实现接口会更好一些。

### 基础线程机制
#### Executor
- Executor 管理 `多个异步任务` 的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。
- 主要有三种 Executor：
	- CachedThreadPool：一个任务创建一个线程；
	- FixedThreadPool：所有任务只能使用固定大小的线程；
	- SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。

		```java
		public static void main(String[] args) {
		    ExecutorService executorService =
                Executors.newCachedThreadPool();
		    for (int i = 0; i < 5; i++) {
		        executorService.execute(new MyRunnable());
		    }
		    executorService.shutdown();
		}
		```

#### Daemon
- `守护线程` 是程序运行时在 `后台` 提供服务的线程，不属于程序中不可或缺的部分。

- 当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。

	> 其中 main() 属于非守护线程。

- 在线程启动之前使用 setDaemon() 方法可以将一个线程设置为守护线程。

	```java
	public static void main(String[] args) {
	    Thread thread = new Thread(new MyRunnable());
	    thread.setDaemon(true);
	}
	```

#### sleep()
- `Thread.sleep(millisec)` 方法会 `休眠` 当前正在执行的线程，`millisec` 单位为毫秒。
- sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。

	```java
	public void run() {
	    try {
	        Thread.sleep(3000);
	    } catch (InterruptedException e) {
	        e.printStackTrace();
	    }
	}
	```

#### yield()
- 对静态方法 `Thread.yield()` 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个 `建议`，而且也只是建议具有相同优先级的其它线程可以运行。

	```java
	public void run() {
	    Thread.yield();
	}
	```

### 线程中断机制
- 一个线程 `执行完毕` 之后会 `自动结束`，如果在运行过程中 `发生异常` 也会 `提前结束`。

#### InterruptedException
- 通过调用一个线程的 `interrupt()` 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。
- 对于以下代码，在 main() 中启动一个线程之后再中断它，由于线程中调用了 Thread.sleep() 方法，因此会抛出一个 InterruptedException，从而提前结束线程，不执行之后的语句。

    ```java
    public class InterruptExample {
        private static class MyThread extends Thread {
            @Override
            public void run() {
                try {
                    Thread.sleep(2000);
                    System.out.println("Thread run");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
	    Thread thread = new MyThread();
	    thread.start();
	    thread.interrupt();
	    System.out.println("Main run");
	}
    ```
    
    抛出异常：
    
    ```html
	Main run
	java.lang.InterruptedException: sleep interrupted
		at java.lang.Thread.sleep(Native Method)
		at InterruptExample.lambda$main$0(InterruptExample.java:5)
		at InterruptExample$$Lambda$1/713338599.run(Unknown Source)
		at java.lang.Thread.run(Thread.java:745)
    ```

#### interrupted()
- 如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。
- 但是调用 interrupt() 方法会设置线程的 `中断标记`，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。

    ```java
	public class InterruptExample {
	    private static class MyThread extends Thread {
	        @Override
            public void run() {
	            while (!interrupted()) {
	                // 忽略代码细节
	            }
	            System.out.println("Thread end");
	        }
	    }
	}

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new MyThread();
        thread.start();
        thread.interrupt();
    }

    // Output: Thread end
    ```

#### Executor 的中断操作
- 调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。
- 以下使用 Lambda 创建线程，相当于创建了一个匿名内部线程。

    ```java
	public static void main(String[] args) {
	    ExecutorService executorService = 
	        Executors.newCachedThreadPool();
	    executorService.execute(() -> {
	        try {
	            Thread.sleep(2000);
	            System.out.println("Thread run");
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	    });
	    executorService.shutdownNow();
	    System.out.println("Main run");
	}
    ```

    抛出异常：

    ```html
	Main run
	java.lang.InterruptedException: sleep interrupted
		at java.lang.Thread.sleep(Native Method)
		at ExecutorInterruptExample.lambda$main$0(ExecutorInterruptExample.java:9)
		at ExecutorInterruptExample$$Lambda$1/1160460865.run(Unknown Source)
		at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
		at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
		at java.lang.Thread.run(Thread.java:745)
    ```
    
- 如果只想中断 Executor 中的一个线程，可以通过使用 submit() 方法来提交一个线程，它会返回一个 Future<?> 对象，通过调用该对象的 cancel(true) 方法就可以中断线程。
  
	```java
	Future<?> future = executorService.submit(() -> {
	    // 忽略代码细节
	});
	future.cancel(true);
	```

### 线程协作
- 当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

#### join()
- 在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。
- 对于以下代码，虽然 b 线程先启动，但是因为在 b 线程中调用了 a 线程的 join() 方法，b 线程会等待 a 线程结束才继续执行，因此最后能够保证 a 线程的输出先于 b 线程的输出。

	```java
	public class JoinExample {
	    private class A extends Thread {
	        @Override
	        public void run() {
	            System.out.println("A");
	        }
	    }

	    private class B extends Thread {
	        private A a;

	        B(A a) {
	            this.a = a;
	        }

	        @Override
	        public void run() {
	            try {
	                a.join();
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	            System.out.println("B");
	        }
	    }

	    public void test() {
	        A a = new A();
	        B b = new B(a);
	        b.start();
	        a.start();
	    }
	}

	public static void main(String[] args) {
	    JoinExample example = new JoinExample();
	    example.test();
	}

	// Output:
	// A
	// B
	```

#### wait() notify() notifyAll()
- 调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。
- 它们都属于 Object 的一部分，而不属于 Thread。
- 只能用在 `同步方法` 或者 `同步控制块` 中使用，否则会在运行时抛出 `IllegalMonitorStateException`。
- 使用 wait() 挂起期间，线程会释放锁。这是因为没有释放锁，其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

	```java
	public class WaitNotifyExample {
	    public synchronized void before() {
	        System.out.println("before");
	        notifyAll();
	    }

	    public synchronized void after() {
	        try {
	            wait();
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	        System.out.println("after");
	    }
	}

	public static void main(String[] args) {
	    ExecutorService executorService = 
	        Executors.newCachedThreadPool();
	    WaitNotifyExample example = new WaitNotifyExample();
	    executorService.execute(() -> example.after());
	    executorService.execute(() -> example.before());
	}

    // Output:
    // before
    // after
  ```

- wait() 和 sleep() 的区别：
	- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
	- wait() 会释放锁，sleep() 不会。

#### await() signal() signalAll()
- `java.util.concurrent` 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。
- 相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。
- 使用 Lock 来获取一个 Condition 对象。

	```java
	public class AwaitSignalExample {
	    private Lock lock = new ReentrantLock();
	    private Condition condition = lock.newCondition();

	    public void before() {
	        lock.lock();
	        try {
	            System.out.println("before");
	            condition.signalAll();
	        } finally {
	            lock.unlock();
	        }
	    }

	    public void after() {
	        lock.lock();
	        try {
	            condition.await();
	            System.out.println("after");
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        } finally {
	            lock.unlock();
	        }
	    }
	}

	public static void main(String[] args) {
	    ExecutorService executorService = 
	        Executors.newCachedThreadPool();
	    AwaitSignalExample example = new AwaitSignalExample();
	    executorService.execute(() -> example.after());
	    executorService.execute(() -> example.before());
	}

	// Output:
	// before
	// after
	```

### 同步互斥
- Java 提供了两种锁机制来控制 `多个线程` 对共享资源的 `互斥访问`，第一个是 JVM 实现的 `synchronized`，而另一个是 JDK 实现的 `ReentrantLock`。

#### 参考资料
- [Aoho. 并发编程的锁机制 synchronized 和 lock. juejin.im](https://juejin.im/post/5a43ad786fb9a0450909cb5f)
- [Matrix海子. Java并发编程 Lock. cnblogs.com](https://www.cnblogs.com/dolphin0520/p/3923167.html)
- [Cyc2018. Java并发-互斥同步. cyc2018.github.io](https://cyc2018.github.io/CS-Notes/#/notes/Java%20并发?id=五、互斥同步)

####  锁的分类
- 锁的分类有很多种，比如自旋锁、自旋锁的其他种类、阻塞锁、可重入锁、读写锁、互斥锁、悲观锁、乐观锁、公平锁、可重入锁等。

	我们这边重点看如下几种：`可重入锁`、`读写锁`、`可中断锁`、`公平锁`。
	
	|  | Synchronized | ReentranLock | ReentrantReadWirteLock |
	| :-: | :-: | :-: | :-: |
	| 可重入锁 | ✓ | ✓ | - |
	| 读写锁 | - | - | ✓ |
	| 可中断锁 | ✕ | ✓ | - |
	| 公平锁 | ✕ | ✕ or ✓ | ✕ or ✓ |
	
	> ✓ 表示支持；✕ 表示不支持；- 表示无关

- `可重入锁`：可重入性表明了锁的分配机制，即基于线程的分配，而不是基于方法调用的分配。若锁具备可重入性则称其为可重入锁。

	`synchronized` 和 `ReentrantLock` 都是可重入锁。

	> 例如：一个线程执行到 method1 的 synchronized 方法时，而在 method1中会调用另外一个 synchronized 方法 method2，此时该线程不必重新去申请锁，而是可以直接执行方法 method2。

- `读写锁`：读写锁将对一个资源的访问分成了 2 个锁。比如文件，一个读锁 (共享锁) 和一个写锁 (排它锁)。正因为有了读写锁，才使得多个线程之间的读操作不会发生冲突。

	`ReadWriteLock` 接口就是读写锁，`ReentrantReadWriteLock` 实现了这个接口。可以通过 readLock() 获取读锁，通过 writeLock() 获取写锁。

- `可中断锁`：即可以中断的锁。`synchronized` 就不是 `可中断锁`，而 `Lock` 是 `可中断锁`。Lock 接口中的 `lockInterruptibly()` 方法就体现了 Lock 的可中断性。

	> 例如：某一线程 A 正在执行锁中的代码，另一线程 B 正在等待获取该锁，可能由于等待时间过长，线程 B 不想等待了，想先处理其他事情，我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁。

- `公平锁`：尽量以 `请求锁的顺序` 来获取锁。同时若有多个线程在等待一个锁，当这个锁被释放时，等待时间最久的线程或 `最先请求的线程` 会获得该锁，这种就是公平锁。

	非公平锁即无法保证锁的获取是按照请求锁的顺序进行的，这样就可能导致某个或者一些线程永远获取不到锁。
	
	- synchronized 就不是 `公平锁`，它无法保证等待的线程获取锁的顺序。
	- Lock 也不是 `公平锁`，它无法保证等待的线程获取锁的顺序。对于 `ReentrantLock` 和 `ReentrantReadWriteLock`，默认是非公平锁，可设置为公平锁。

#### Lock 和 synchronized
##### synchronized
- synchronized 是 Java 的关键字，当它用来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码。简单总结如下四种用法：
	- `代码块`：对某一代码块使用，synchronized 后跟括号，括号里是变量。

		```java
		public int func(int m){
			synchronized(m) {
				//...
			}
		}
		```
		
	- `方法声明`：它和同步代码块一样，作用于同一个对象。即一次只能一个线程进入该方法，其他线程想在此时调用该方法只能排队等候。

		```java
		// 放在范围操作符之后，返回类型声明之前
		public synchronized void func() {
			// ...
		}
		```

	- `同步一个对象`：synchronized 后面括号里是对象，此时线程获得的是对象锁。由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步。当一个线程进入同步语句块时，另一个线程就必须等待。

		> 同一对象需同步，不同对象不同步。

		```java
		public class SynchronizedExample {
			public void func1() {
				synchronized (this) {
					for (int i = 0; i < 10; i++) {
						System.out.print(i + " ");
					}
				}
			}
		}
		
		// Output: 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
		public static void main(String[] args) {
			SynchronizedExample e1 =  new SynchronizedExample();
			ExecutorService executorService =
				Executors.newCachedThreadPool();
			executorService.execute(() -> e1.func1());
			executorService.execute(() -> e1.func1());
		}
		
		// Output: 0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9
		public static void main(String[] args) {
			SynchronizedExample e1 = new SynchronizedExample();
			SynchronizedExample e2 = new SynchronizedExample();
			ExecutorService executorService = 
				Executors.newCachedThreadPool();
			executorService.execute(() -> e1.func1());
			executorService.execute(() -> e2.func1());
		}
		```

	- `同步一个类`：作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。

		> 同一个类，不同一对象需同步。

		```java
		public class SynchronizedExample {
			public void func2() {
				synchronized (SynchronizedExample.class) {
					for (int i = 0; i < 10; i++) {
						System.out.print(i + " ");
					}
				}
			}
		}
		
		// Output: 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
		public static void main(String[] args) {
		SynchronizedExample e1 = new SynchronizedExample();
		SynchronizedExample e2 = new SynchronizedExample();
		ExecutorService executorService =
			Executors.newCachedThreadPool();
		executorService.execute(() -> e1.func2());
		executorService.execute(() -> e2.func2());
		}
		```

	-  同步一个 `静态方法`：

		```java
		public synchronized static void fun() {
			// ...
		}
		```
	
##### Lock
> Lock 是锁接口，其实现类为 ReetrantLock。  
> ReadWriteLock 是读写锁接口，其实现类为 ReetrantReadWriteLock。  

- Lock：

	```java
	public interface Lock {
		/**
		 * 获取锁，如果锁被其他线程获取，处于等待状态
		 * 必须主动去释放锁，并且在发生异常时不会自动释放锁。
		 * 因此一般来说，使用 Lock 必须在 try{...}catch{...} 块中进行，
		 * 并且将释放锁的操作放在 `finally` 块中进行，
		 * 以保证锁一定被被释放，防止死锁的发生。
		 */
		void lock();
		
		// 尝试获取锁
		// 若失败，等待的过程中可响应中断 threadWait.interrupt()
		void lockInterruptibly() throws InterruptedException;  
		
		// 尝试获取锁，若获取成功，就马上返回 true
		// 否则马上返回 false (锁已经被其他线程获取)
		boolean tryLock();  
		
		// 尝试获取锁，若获取失败，会等待 unit 时间
		// 等待期间还拿不到锁就马上返回 false
		boolean tryLock(long time, TimeUnit unit) 
			throws InterruptedException;  
			
		// 释放锁，一定要在 finally 块中释放
		void unlock();  
		Condition newCondition();
	}
	```

- ReetrantLock：可重入锁，Lock 接口的实现类，且内部定义了公平锁与非公平锁 (默认为非公平锁)：

	```java
	// 默认情况
	public ReentrantLock() {  
		sync = new NonfairSync();  
	}
	
	// 可以手动设置为公平锁：
	public ReentrantLock(boolean fair) {  
		sync = fair ? new FairSync() : new NonfairSync();  
	}  
	```

- ReadWriteLock：一个用来获取读锁 (共享锁)，一个用来获取写锁 (排它锁)。也就是说将文件的读写操作分开，分成 2 个锁来分配给线程，从而使得多个线程可以同时进行读操作。

	```java
	public interface ReadWriteLock {  
		Lock readLock();	// 获取读锁  
		Lock writeLock();	// 获取写锁  
	}
	```
	
- ReentrantReadWirteLock 实现了 ReadWirteLock 接口，并未实现 Lock 接口。
	
	```java
	private ReadWriteLock rwl = new ReentrantReadWriteLock();
	
	public static void main(String[] args) {
		final Main main = new Main();
	
		new Thread(
			() -> main.testRWL(Thread.currentThread()) ).start();
		new Thread(
			() -> main.testRWL(Thread.currentThread()) ).start();
	
	    // 输出的结果是两个 thread 交替输出“正在读”
	}
	
	public void testRWL(Thread thread) {
		rwl.readLock().lock();
		try {
			long finish = System.currentTimeMillis() + 1;
			while (System.currentTimeMillis() <= finish) {
				System.out.println(thread.getName() + "正在读");
			}
			System.out.println(thread.getName() + "读结束");
		} finally {
			rwl.readLock().unlock();
		}
	}
	```
	
	- 若一个线程已经占用了读锁，其他线程可以马上获得读锁，但需要等待才能获取写锁，则申请写锁的线程会一直等待释放读锁。

	- 若一个线程已经占用了写锁，其他线程要获取读锁或写锁都需要等待，则申请的线程会一直等待释放写锁。

####  锁的比较
- `锁的实现`：synchronized 是 Java 关键字，Lock 是接口。synchronized 是 JVM 实现的，而 Lock / ReentrantLock 是 JDK 实现的。
- `性能比较`：新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等。synchronized 与 ReentrantLock 大致相同。
- `异常处理方式`：
	- synchronized：会自动释放线程占有的锁，因此不会导致死锁现象发生；
	- Lock：若没有主动通过 unLock() 去释放锁，则很可能造成死锁现象，因此使用 Lock 时需要在 finally 块中释放锁。
- `等待可中断`：当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

	> Lock / ReentrantLock 可中断，而 synchronized 不行。

#### 使用选择
- 除非需要使用 Lock / ReentrantLock 的 `高级功能`，否则优先使用 `synchronized`。

	> Lock 是基于在语言层面实现的锁，Lock 锁可以被中断，支持定时锁等。

- 因为 synchronized 是 `JVM` 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。
- 并且使用 synchronized 不用担心没有释放锁而导致 `死锁问题`，因为 JVM 会确保锁的释放。

### 线程不安全示例
- 若多个线程对同一个共享数据进行访问而不采取同步操作的话，那么操作的结果是不一致的。

	以下代码演示了 1000 个线程同时对 cnt 执行自增操作，操作结束之后它的值有可能小于 1000。
	
	```java
    public class ThreadUnsafeExample {
	    private int cnt = 0;
	    
	    public void add() {
	        cnt++;
        }

	    public int get() {
	        return cnt;
	    }
    }
    
	public static void main(String[] args) throws InterruptedException {
	    final int threadSize = 1000;
	    
	    ThreadUnsafeExample example = 
	    	new ThreadUnsafeExample();
	    	
	    final CountDownLatch countDownLatch = 
	    	new CountDownLatch(threadSize);
	    	
	    ExecutorService executorService = 
	    	Executors.newCachedThreadPool();
	    	
	    for (int i = 0; i < threadSize; i++) {
	        executorService.execute(() -> {
	            example.add();
	            countDownLatch.countDown();
	        });
	    }
	    countDownLatch.await();
	    executorService.shutdown();
	    System.out.println(example.get());
	}

	// Output: 997
	```

### Java 内存模型
- Java 内存模型试图屏蔽各种 `硬件` 和 `操作系统` 的内存访问差异，以实现让 Java 程序在各种平台下都能达到一致的内存访问效果。

#### 主内存与工作内存
- 处理器上的寄存器的读写的速度比内存快几个数量级，为了解决这种 `速度矛盾`，在它们之间加入了 `高速缓存`。

	加入高速缓存带来了一个新的问题：`缓存一致性`。如果多个缓存共享同一块主内存区域，那么多个缓存的数据可能会不一致，需要一些 `协议` 来解决这个问题。

	| ![P、C、M的交互关系](img/Cys2018-CS-Notes-Java_3-7.png) |
	| :-: |
	| 图 3-7 处理器、高速缓存、主内存间的交互关系 |
	
- 所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中，保存了该线程使用的变量的主内存副本拷贝。

	线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。

	| ![P、C、M的交互关系](img/Cys2018-CS-Notes-Java_3-7-1.png) |
	| :-: |
	| 图 3-7-1 线程、工作内存、主内存间的交互关系 (对比图 3-7) |

- 主内存、工作内存与 JVM 内存模型中的 Java 堆、栈、方法区等并不是同一层次的内存划分。若两者的关系一定要对应起来：
	- 从变量、主内存、工作内存的定义来看，主内存主要对应于 Java 堆中的对象实例数据部分，工作内存则对应于虚拟机栈中的部分区域。
	- 从更低层次上说，主内存直接对应于物理硬件的内存，工作内存优先存储于寄存器和高速缓存中。

#### 两内存间交互操作
| ![主内存和工作内存的交互操作](img/Cys2018-CS-Notes-Java_3-7-2.png) |
| :-: |
| 图 3-7-2 主内存和工作内存的交互操作 |

- Java 内存模型定义了 8 个操作来完成主内存和工作内存的交互操作，如图 3-7-2 所示：
	- lock：作用于主内存的变量，把一个变量标识别为一条线程独占的状态。
	- unlock：作用于主内存的变量，释放出于锁定状态的变量。
	- read：作用于主内存的变量，把一个变量的值从主内存传输到工作内存中。
	- load：作用于工作内存的变量，在 read 之后执行，把 read 得到的值放入工作内存的变量副本中。
	- use：作用于工作内存的变量，把工作内存中一个变量的值传递给执行引擎。
	- assign：作用于工作内存的变量，把一个从执行引擎接收到的值赋给工作内存的变量。
	- store：作用于工作内存的变量，把工作内存的一个变量的值传送到主内存中。
	- write：作用于主内存的变量，在 store 之后执行，把 store 得到的值放入主内存的变量中。

#### 内存模型三大特征
##### 原子性
- Java 内存模型保证了 read、load、assign、use、store、write、lock 和 unlock 操作具有原子性，例如对一个 int 类型的变量执行 assign 赋值操作，这个操作就是原子性的。

	> long 和 double 的非原子性协定：Java 内存模型允许虚拟机将没有被 `volatile` 修饰的 64 位数据 (long，double) 的读写操作划分为两次 32 位的操作来进行，即 load、store、read 和 write 操作可以不具备原子性。
	
- 有一个错误认识就是，int 等原子性的类型在多线程环境中不会出现线程安全问题。前面的 [线程不安全示例](线程不安全示例) 代码中，cnt 属于 int 类型变量，1000 个线程对它进行自增操作之后，得到的值为 997 而不是 1000。

	下图演示了两个线程同时对 cnt 进行操作，load、assign、store 这一系列操作整体上看不具备原子性，那么在 T1 修改 cnt 并且还没有将修改后的值写入主内存，T2 依然可以读入旧值。可以看出，这两个线程虽然执行了两次自增运算，但是主内存中 cnt 的值最后为 1 而不是 2。因此对 int 类型读写操作满足原子性只是说明 load、assign、store 这些单个操作具备原子性。
	
	> 为了方便讨论，将内存间的交互操作简化为 3 个：load、assign、store。

	| ![两个线程同时对cnt进行操作示意](img/Cys2018-CS-Notes-Java_3-7-3.png) | ![AtomicInteger对cnt进行操作示意](img/Cys2018-CS-Notes-Java_3-7-4.png) |
	| :-: | :-: |
	| 图 3-7-3 两个线程同时对 cnt 进行操作示意 | 图 3-7-4 AtomicInteger 对 cnt 进行操作示意 |

- `AtomicInteger` 能保证多个线程修改的原子性，如图 3-7-4 所示。

- 使用 AtomicInteger 重写之前线程不安全的代码之后得到以下线程安全实现：

	```java
	public class AtomicExample {
	    private AtomicInteger cnt = new AtomicInteger();

	    public void add() {
		    cnt.incrementAndGet();
	    }

	    public int get() {
		    return cnt.get();
	    }
	}
	
	public static void main(String[] args) throws InterruptedException {
	    final int threadSize = 1000;
	    AtomicExample example = new AtomicExample(); // 只修改这条语句
	    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
	    ExecutorService executorService = Executors.newCachedThreadPool();
	    for (int i = 0; i < threadSize; i++) {
	        executorService.execute(() -> {
	            example.add();
	            countDownLatch.countDown();
	        });
	    }
	    countDownLatch.await();
	    executorService.shutdown();
	    System.out.println(example.get());
	}
	
	// Output: 1000
	```

- 除了使用原子类之外，也可以使用 `synchronized` 互斥锁来保证操作的原子性。它对应的内存间交互操作为 lock 和 unlock，在虚拟机实现上对应的字节码指令为 monitorenter 和 monitorexit。

	```java
	public class AtomicSyncExample {
	    private int cnt = 0;
	
	    public synchronized void add() {
	        cnt++;
	    }
	
	    public synchronized int get() {
	        return cnt;
	    }
	}
	
	public static void main(String[] args) throws InterruptedException {
	    final int threadSize = 1000;
	    AtomicSyncExample example = new AtomicSyncExample();
	    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
	    ExecutorService executorService = Executors.newCachedThreadPool();
	    for (int i = 0; i < threadSize; i++) {
	        executorService.execute(() -> {
	            example.add();
	            countDownLatch.countDown();
	        });
	    }
	    countDownLatch.await();
	    executorService.shutdown();
	    System.out.println(example.get());
	}
	
	// Output: 1000
	```

##### 可见性
- 可见性：指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。
- 主要有三种实现可见性的方式：
	- `volatile`：volatile 的特殊规则保证了新值能立即同步到主内存，以及每次使用前立即从主内存刷新。
	- `synchronized`：对一个变量执行 unlock 操作之前，必须把变量值同步回主内存。
	- `final`：被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 `this引用逃逸`，那么其它线程就能看见 final 字段的值。

		> `this引用逃逸`：其它线程通过 this 引用访问到初始化了一半的对象。

- 对前面的线程不安全示例中的 cnt 变量使用 volatile 修饰，不能解决线程不安全问题，因为 volatile 并不能保证操作的原子性。

##### 有序性
- 有序性：指在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序。在 Java 内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

- `volatile` 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。

- 也可以通过 `synchronized` 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。

#### 先行发生原则
- 上面提到了可以用 volatile 和 synchronized 来保证有序性。除此之外，JVM 还规定了先行发生原则，让一个操作无需控制就能先于另一个操作完成。
- 若两个操作之间的关系不在下列规则，并且无法从下列规则推导出来的话，则它们没有顺序性保障，JVM 可以对它们随意地进行重排序。

##### 程序次序原则
- 程序次序原则 (Program Order Rule)：在一个线程内，在程序前面的操作先行发生于后面的操作。更准确说,应该是代码中控制流的顺序，因为要考虑分支、循环结构。如图 3-7-5 所示。

	| ![程序次序原则](img/Cys2018-CS-Notes-Java_3-7-5.png) |
	| :-: |
	| 图 3-7-5 程序次序原则 |

##### 管程锁定规则
- 管程锁定规则 (Monitor Lock Rulu)：一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。如图 3-7-6 所示。

	> 必须是同一个锁；“后面” 指时间上的先后顺序。

	| ![程序次序原则](img/Cys2018-CS-Notes-Java_3-7-6.png) |
	| :-: |
	| 图 3-7-6 管程锁定规则 |

##### Volatile 变量规则
- volatile 变量规则 (Volatile Variable Rule)：对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。如图 3-7-7 所示。

	> “后面” 指时间上的先后顺序。

	| ![程序次序原则](img/Cys2018-CS-Notes-Java_3-7-7.png) |
	| :-: |
	| 图 3-7-7 Volatile 变量规则 |

##### 线程启动规则
- 线程启动规则 (Thread Start Rule)：Thread 对象的 start() 方法调用先行发生于此线程的每一个动作。

	| ![线程启动规则](img/Cys2018-CS-Notes-Java_3-7-8.png) |
	| :-: |
	| 图 3-7-8 线程启动规则 |

##### 线程终止规则
- 线程终止规则 (Thread Termination Rule)：线程中的所有操作都先行发生于对此线程的终止检测。可通过 Thread.join() 方法结束、Thread.isAlive() 的返回值等手段检测到线程已经终止执行。

	| ![线程终止规则](img/Cys2018-CS-Notes-Java_3-7-9.png) |
	| :-: |
	| 图 3-7-9 线程终止规则 |

##### 线程中断规则
- 线程中断规则 (Thread Interruption Rule)：对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过 Thread.interrupted() 方法检测到是否有中断发生。

##### 对象终结规则
- 对象终结规则 (Finalizer Rule)：一个对象的初始化完成 (构造函数执行结束) 先行发生于它的 finalize() 方法的开始。

##### 传递性
- 传递性 (Transitivity)：如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。

### 线程安全
- 线程安全：当多个线程访问一个对象时，如果不考虑这些线程在运行时环境下的调度和交替执行，也不需要进行额外的同步，或者在调用方法进行任何其他的协调操作，调用这个对象的行为都可以获得正确的结果，那么这个对象是线程安全的  $^{[1, 2]}$。

	> 该定义较严谨，它要求线程安全的代码都必须具备一个特征：代码本身封装了所有必要的正确性保障手段 (同步互斥等)，令调用者无须关系多线程的问题，无须自己采取措施来保证多线程的正确调用。

#### 参考资料
- [1] [周志明. 深入理解 Java 虚拟机 [M]. 第二版. 机械工业出版社, 2013](https://book.douban.com/subject/24722612/)
- [2] [ Brian Goetz. Java Concurrency in Practice [M]. Addison-Wesley Professional, 2016](https://book.douban.com/subject/1888733/)

#### Java 语言的线程安全
- 不可变：
	- 不可变 (Immutable) 的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。只要一个不可变的对象被正确地构建出来，永远也不会看到它在多个线程之中处于不一致的状态。
	- 多线程环境下，应当尽量使对象成为不可变，来满足线程安全。

- 不可变的类型：
	- final 关键字修饰的基本数据类型；
	- String 类型；
	- 枚举类型；
	- Number 部分子类，如 Long 和 Double 等数值包装类型，BigInteger 和 BigDecimal 等大数据类型。但同为 Number 的原子类 AtomicInteger 和 AtomicLong 则是可变的。

- 对于集合类型：可以使用 `Collections.unmodifiableXXX()` 方法来获取一个不可变的集合。

	```java
	public class ImmutableExample {
	    public static void main(String[] args) {
	        Map<String, Integer> map = new HashMap<>();
	        Map<String, Integer> unmodifiableMap = 
	            Collections.unmodifiableMap(map);
	        unmodifiableMap.put("a", 1);
	    }
	}
	```
	
	- 执行 unmodifiableMap.put() 则会抛出以下异常：
	
		```java
		Exception in thread "main" java.lang.UnsupportedOperationException
		    at java.util.Collections$UnmodifiableMap.put(Collections.java:1457)
		    at ImmutableExample.main(ImmutableExample.java:9)
		```
	
	- Collections.unmodifiableXXX() 先对原始的集合进行拷贝，需要对集合进行修改的方法都直接抛出异常。
	
		```java
		public V put(K key, V value) {
		    throw new UnsupportedOperationException();
		}
		```

#### 线程安全的实现方法
##### 互斥同步
- 互斥同步 (Synchronization) 是常见的一种并发正确性保障手段。
	- 同步是指多个线程并发访问共享数据时，保证共享数据在同一时刻只被一个线程使用 (使用信号量时可以是一些线程)。
	- 互斥是实现同步的一种手段，临界区、互斥量和信号量都是主要的互斥实现方式。

		> 互斥是因，同步是果；互斥是方法，同步是目的。
	
- synchronized 关键字和 ReentrantLock 接口。详细见上述章节 [同步互斥](#同步互斥)。

##### 非阻塞同步
- 互斥同步最主要的问题就是 `线程阻塞和唤醒` 所带来的 `性能问题`，因此这种同步也称为阻塞同步 (Blocking Synchronization)。
- `互斥同步` 属于一种 `悲观并发策略`，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁 (这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁)、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。
- 随着硬件指令集的发展，我们可以使用基于 `冲突检测` 的 `乐观并发策略`：先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施，即 `不断地重试`，直到成功为止。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为 `非阻塞同步`。

###### CAS
- 乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：`比较并交换` (Compare-and-Swap，CAS)。
- CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。

###### AtomicInteger
- J.U.C 包 (java.util.concurrent) 里面的整数原子类 AtomicInteger 的方法调用了 Unsafe 类的 CAS 操作。

	- 以下代码使用了 AtomicInteger 执行了自增的操作。
	
		```java
		private AtomicInteger cnt = new AtomicInteger();
	
		public void add() {
		    cnt.incrementAndGet();
		}
		```
		
	- 以下代码是 incrementAndGet() 的源码，它调用了 Unsafe 的 getAndAddInt() 。

		```java
		public final int incrementAndGet() {
		    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
		}
		```
		
	- 以下代码是 getAndAddInt() 源码，var1 指示对象内存地址，var2 指示该字段相对对象内存地址的偏移，var4 指示操作需要加的数值，这里为 1。

		通过 getIntVolatile(var1, var2) 得到旧的预期值，通过调用 compareAndSwapInt() 来进行 CAS 比较，如果该字段内存地址中的值等于 var5，那么就更新内存地址为 var1+var2 的变量为 var5+var4。

		可以看到 getAndAddInt() 在一个循环中进行，发生冲突的做法是不断的进行重试。
	
		```java
		public final int getAndAddInt(Object var1, long var2, int var4) {
		    int var5;
		    do {
		        var5 = this.getIntVolatile(var1, var2);
		    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
		
		    return var5;
		}
		```

###### ABA
- 如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。
- J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它可以通过控制变量值的版本来保证 CAS 的正确性。大部分情况下 ABA 问题不会影响程序并发的正确性，如果需要解决 ABA 问题，改用传统的 `互斥同步` 可能会比原子类更高效。

##### 无同步方案
- 要保证线程安全，同步并不是必要步骤。毕竟同步只是保证共享数据争用时的正确性手段，若一个方法本身就不设计不涉及共享数据，那么它无须任何同步措施去确保正确性。

###### 栈封闭
- 多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为 `局部变量` 存储在 `虚拟机栈` 中，属于线程私有的。

	```java
	public class StackClosedExample {
	    public void add() {
	        int cnt = 0;
	        for (int i = 0; i < 100; i++) {
	            cnt++;
	        }
	        System.out.println(cnt);
	    }
	}
	
	public static void main(String[] args) {
	    StackClosedExample example = new StackClosedExample();
	    ExecutorService executorService = Executors.newCachedThreadPool();
	    executorService.execute(() -> example.add());
	    executorService.execute(() -> example.add());
	    executorService.shutdown();
	}
	```

######  线程本地存储
- 线程本地存储 (Thread Local Storage)：如果一段代码中所需要的数据必须与其他代码共享，判断这些共享数据的代码是否能保证在同一个线程中执行。若能保证就可以把共享数据的可见范围限制在同一个线程之内，这样无须同步也能保证线程之间不出现数据争用的问题。

- 符合这种特点的应用并不少见，大部分使用 `消费队列架构模式` (如 `生产者-消费者` 模式) 都会将产品的消费过程尽量在一个线程中消费完。

	> 一个经典应用实例就是 Web 交互模型中的 “一个请求对应一个服务器线程” (Thread-per-Request)，这种处理方式的广泛应用使得很多 Web 服务端应用都可以使用线程本地存储来解决线程安全问题。

- 可以使用 `java.lang.ThreadLocal` 类来实现线程本地存储功能。
	- 对于以下代码，thread1 中设置 threadLocal 为 1，而 thread2 设置 threadLocal 为 2。过了一段时间之后，thread1 读取 threadLocal 依然是 1，不受 thread2 的影响。

		```java
		public class ThreadLocalExample {
		    public static void main(String[] args) {
		        ThreadLocal threadLocal = new ThreadLocal();
		        Thread thread1 = new Thread(() -> {
		            threadLocal.set(1);
		            try {
		                Thread.sleep(1000);
		            } catch (InterruptedException e) {
		                e.printStackTrace();
		            }
		            System.out.println(threadLocal.get());
		            threadLocal.remove();
		        });
		        Thread thread2 = new Thread(() -> {
		            threadLocal.set(2);
		            threadLocal.remove();
		        });
		        thread1.start();
		        thread2.start();
		    }
		}
		```
		
	- 为了理解 ThreadLocal，先看以下代码：

		```java
		public class ThreadLocalExample1 {
		    public static void main(String[] args) {
		        ThreadLocal threadLocal1 = new ThreadLocal();
		        ThreadLocal threadLocal2 = new ThreadLocal();
		        Thread thread1 = new Thread(() -> {
		            threadLocal1.set(1);
		            threadLocal2.set(1);
		        });
		        Thread thread2 = new Thread(() -> {
		            threadLocal1.set(2);
		            threadLocal2.set(2);
		        });
		        thread1.start();
		        thread2.start();
		    }
		}
		```
		
		它所对应的底层结构图为：
		
		| ![ThreadLocal底层结构图](img/Cys2018-CS-Notes-Java_3-8.png) |
		| :-: |
		| 图 3-8 ThreadLocal底层结构图 |
	
	- 每个 Thread 都有一个 ThreadLocal.ThreadLocalMap 对象。

		```java
		/**
		 * ThreadLocal values pertaining to this thread. 
		 * This map is maintained by the ThreadLocal class.
		 */
		ThreadLocal.ThreadLocalMap threadLocals = null;
		```
		
	- 当调用一个 ThreadLocal 的 set(T value) 方法时，先得到当前线程的 ThreadLocalMap 对象，然后将 ThreadLocal->value 键值对插入到该 Map 中。

		> get() 方法类似。

		```java
		public void set(T value) {
		    Thread t = Thread.currentThread();
		    ThreadLocalMap map = getMap(t);
		    if (map != null)
		        map.set(this, value);
		    else
		        createMap(t, value);
		}
		```

- ThreadLocal 从理论上讲并不是用来解决多线程并发问题的，因为根本不存在多线程竞争。

	在一些场景下，尤其是使用线程池，由于 ThreadLocal.ThreadLocalMap 的底层数据结构导致 ThreadLocal 有内存泄漏的情况，应该尽可能在每次使用 ThreadLocal 后手动调用 remove()，以避免出现 ThreadLocal 经典的内存泄漏甚至是造成自身业务混乱的风险。

######  可重入代码
-  可重入代码 (Reentrant Code)：也称纯代码 (Pure Code)，可以在代码执行的任何时刻中断它，转而去执行另外一段代码 (包括递归调用它本身)，而在控制权返回后，原来的程序不会出现任何错误。
- 可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源、用到的状态量都由参数中传入、不调用非可重入的方法等。
- 通过简单原则来判断代码是否具备可重入性：如果一个方法，它的返回结果是可以预测的，只要输入相同的数据就能返回相同的结果，那么它满足可重入性的要求，当然也是线程安全的。

### 锁优化
- 这里的锁优化主要是指 JVM 对 `synchronized` 的优化。

#### 自旋锁
- 互斥同步进入阻塞状态的开销都很大，应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短的一段时间。自旋锁的思想是让一个线程在请求一个共享数据的锁时执行 `忙循环 (自旋)` 一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。
- 自旋锁虽然能避免进入阻塞状态从而减少开销，但是它需要进行忙循环操作占用 CPU 时间，它只适用于共享数据的锁定状态很短的场景。
- 在 JDK 1.6 中引入了自适应的自旋锁。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。

#### 锁消除
- 锁消除是指对于被检测出不可能存在竞争的共享数据的锁进行消除。
- 锁消除主要是通过 `逃逸分析` 来支持，如果堆上的共享数据不可能逃逸出去被其它线程访问到，那么就可以把它们当成私有数据对待，也就可以将它们的锁进行消除。
- 对于一些看起来没有加锁的代码，其实 `隐式` 的加了很多锁。例如下面的字符串拼接代码就隐式加了锁：

	```java
	public static String concatString(String s1, String s2, String s3) {
	    return s1 + s2 + s3;
	}
	```
	
- String 是一个不可变的类，编译器会对 String 的拼接自动优化。在 JDK 1.5 之前，会转化为 StringBuffer 对象的连续 append() 操作：

	每个 append() 方法中都有一个同步块。虚拟机观察变量 sb，很快就会发现它的动态作用域被限制在 concatString() 方法内部。也就是说，sb 的所有引用永远不会逃逸到 concatString() 方法之外，其他线程无法访问到它，因此可以进行消除。

	```java
	public static String concatString(String s1, String s2, String s3) {
	    StringBuffer sb = new StringBuffer();
	    sb.append(s1);
	    sb.append(s2);
	    sb.append(s3);
	    return sb.toString();
	}
	```

#### 锁粗化
- 如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。

- 上一节的示例代码中连续的 append() 方法就属于这类情况。如果虚拟机探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展 (粗化) 到整个操作序列的外部。

	对于上一节的示例代码就是扩展到第一个 append() 操作之前直至最后一个 append() 操作之后，这样只需要加锁一次就可以了。

#### 轻量级锁
- JDK 1.6 引入了偏向锁和轻量级锁，从而让锁拥有了四个状态：无锁状态 (unlocked)、偏向锁状态 (biasble)、轻量级锁状态 (lightweight locked) 和重量级锁状态 (inflated)。

	> 轻量级锁是较于传统的 “重量级” 锁的说法，它并不是用来代替重量级锁的，而是在没有多线程的前提下，减少传统的重量级锁定使用操作系统互斥量产生的 `性能消耗`。

#### 偏向锁
- 偏向锁的思想是偏向于让第一个获取锁对象的线程，这个线程在之后获取该锁就不再需要进行同步操作，甚至连 CAS 操作也不再需要。
- 当有另外一个线程去尝试获取这个锁对象时，偏向状态就宣告结束，此时撤销偏向 (Revoke Bias) 后恢复到未锁定状态或者轻量级锁状态。

### 多线程开发良好实践
> 摘录自原文：[CyC2018. Java 并发-多线程开发良好的实践. cyc2018.github.io](https://cyc2018.github.io/CS-Notes/#/notes/Java%20并发?id=十三、多线程开发良好的实践)

- 定义线程，起个有意义的名字，便于识别，且方便找 BUG。

- 缩小同步范围，从而减少锁争用。例如对于 synchronized，应该尽量使用同步块而不是同步方法。

- 多用同步工具，少用 wait() 和 notify()。
	- 例如，CountDownLatch, CyclicBarrier, Semaphore 和 Exchanger 这些同步类简化了编码操作，而用 wait() 和 notify() 很难实现复杂控制流；
	- 其次，这些同步类是由最好的企业编写和维护，在后续的 JDK 中还会不断优化和完善。

- 使用 BlockingQueue 实现 `生产者-消费者` 架构模式。
- 多用并发集合，少用同步集合。 例如应该使用 ConcurrentHashMap 而不是 HashMap。
- 使用本地变量和不可变类来保证线程安全。
- 使用线程池，而不是直接创建线程，这是因为创建线程代价很高，线程池可以有效地利用有限的线程来启动任务。

## Java 虚拟机

### 参考资料 
- [周志明. 深入理解 Java 虚拟机 [M]. 第二版. 机械工业出版社, 2013](https://book.douban.com/subject/24722612/)
- [归去来兮. HotSpot的安全区和安全点. cnblogs.com](https://www.cnblogs.com/mazhimazhi/p/11337660.html)

### JVM 运行时数据区域
- Java 虚拟机 (Java Virtual Machine, JVM) 在执行 Java 程序的过程中会把它所管理的内存划分为若干不同的数据区域。
- 这些区域各有用途，以及各自创建和销毁的时间。比如有的区域随着虚拟机进程的启动而存在，有些区域以来用户线程的启动而创建，结束而销毁。

	| ![运行时数据区域](img/Cys2018-CS-Notes-Java_4-1.png) |
	| :-: |
	| 图 4-1 Java 虚拟机运行时数据区域 / JVM 内存模型 (来自 [CyC2018.CS-Notes](https://cyc2018.github.io/CS-Notes/#/notes/Java%20虚拟机?id=一、运行时数据区域) ) |

##### 程序计数器
- 程序计数器：记录正在执行的虚拟机字节码指令的地址 (如果正在执行的是本地方法则为空)。

- Java 虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式实现的，在任何时刻一个处理器 (多核心处理器是内核) 都只会执行一条线程中的指令。 

	为此，确保线程切换后能恢复正确的执行位置，每条线程都需要拥有一个独立的程序计数器。我们称这类内存区域为 `线程私有的内存`。

##### Java 虚拟机栈
- Java 虚拟机栈：`线程私有的内存`。每个 Java 方法在执行的同时会创建一个 `栈帧` 用于存储 `局部变量表`、`操作数栈`、`常量池引用` 等信息。从方法调用直至执行完成的过程，对应着一个栈帧在 Java 虚拟机栈中入栈和出栈的过程。

	| ![栈帧](img/Cys2018-CS-Notes-Java_4-1-1.png) |
	| :-: |
	| 图 4-1-1 栈帧 |

- 该区域可能抛出以下异常：
	- 当线程请求的栈深度超过最大值，会抛出 StackOverflowError 异常；
	- 栈进行动态扩展时如果无法申请到足够内存，会抛出 OutOfMemoryError 异常。
- 可以通过 `-Xss` 这个虚拟机参数来指定每个线程的 Java 虚拟机栈内存大小，在 JDK 1.4 中默认为 256K，而在 JDK 1.5+ 默认为 1M：

	```shell
	java -Xss2M HackTheJava
	```

##### 本地方法栈
- 本地方法栈与 Java 虚拟机栈类似，它们之间的区别只不过是本地方法栈为本地方法服务。
- 本地方法一般是用其它语言 (C、C++ 或汇编语言等) 编写的，并且被编译为基于本机硬件和操作系统的程序，对待这些方法需要特别处理。

##### 堆
- `所有线程共享` 的内存区域，虚拟机启动是创建。
- `所有对象` 都在这里 `分配内存`，是垃圾收集的主要区域 (GC 堆，Garbage Collected Heap)。
- 现代的垃圾收集器基本都是采用 `分代收集算法`，其主要的思想是针对不同类型的对象采取不同的垃圾回收算法。可以将堆分成两块：
	- 新生代 (Young Generation)
	- 老年代 (Old Generation)
- 堆不需要连续内存 (不要求物理上连续的内存空间，逻辑连续即可)，并且可以动态增加其内存，增加失败会抛出 OutOfMemoryError 异常。

	可以通过 `-Xms` 和 `-Xmx` 这两个虚拟机参数来指定一个程序的堆内存大小，第一个参数设置初始值，第二个参数设置最大值。
	
	```shell
	java -Xms1M -Xmx2M HackTheJava
	```

##### 方法区
- `所有线程共享` 的内存区域。
- 用于存放已被加载的 `类信息`、`常量`、`静态变量`、即时编译器编译后的代码等数据。
- 和堆一样不需要连续的内存，并且可以动态扩展，动态扩展失败一样会抛出 OutOfMemoryError 异常。
- 对这块区域进行垃圾回收的主要目标是对常量池的回收和对类的卸载，但是一般比较难实现。
- HotSpot 虚拟机把它当成 `永久代` 来进行垃圾回收。但很难确定永久代的大小，因为它受到很多因素影响，并且每次 Full GC 之后永久代的大小都会改变，所以经常会抛出 OutOfMemoryError 异常。

	> 为了更容易管理方法区，从 JDK 1.8 开始，移除永久代，并把方法区移至元空间，它位于本地内存中，而不是虚拟机内存中。

- 方法区是一个 JVM 规范，永久代与元空间都是其一种实现方式。在 JDK 1.8 之后，原来永久代的数据被分到了堆和元空间中。元空间存储类的元信息，静态变量和常量池等放入堆中。

##### 运行时常量池
- 运行时常量池是方法区的一部分。
- Class 文件中的常量池 (编译器生成的字面量和符号引用) 会在类加载后被放入这个区域。
- 除了在编译期生成的常量，还允许动态生成，例如 String 类的 intern()。

	```java
	// String.intern() -- JDK 1.8
	if 判断这个常量是否存在于常量池 { // 存在
	    if 判断存在内容是引用还是常量 {
	        如果是引用，返回引用地址指向堆空间对象
	    } else {
	        如果是常量，直接返回常量池常量
	    }
	} else { // 不存在
	    将当前对象引用复制到常量池,并且返回的是当前对象的引用
	}
	```

##### 直接内存
- 直接内存并不是虚拟机运行时数据区的一部分，也不是 JVM 规范中定义的内存区域。
- 在 JDK 1.4 中新引入了 `NIO` (New Input/Output) 类，它可以使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆里的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在堆内存和堆外内存来回拷贝数据。

### 垃圾收集算法与工具
- 垃圾收集 (Garbage Collection, GC) 主要是针对 `堆` 和 `方法区` 进行。
- 程序计数器、虚拟机栈和本地方法栈这三个区域属于线程私有的，只存在于线程的生命周期内，随线程结束就会消失，因此不需要对这三个区域进行垃圾回收。

#### 对象存活判定算法
##### 引用计数算法
- 为对象添加一个引用计数器，当对象增加一个引用时计数器加 1，引用失效时计数器减 1。引用计数为 0 的对象可被回收。

- 在两个对象出现循环引用的情况下，此时引用计数器永远不为 0，导致无法对它们进行回收。正是因为循环引用的存在，因此 Java 虚拟机不使用引用计数算法。

	```java
	public class Test {

		public Object instance = null;

		public static void main(String[] args) {
			Test a = new Test();
			Test b = new Test();
			a.instance = b;
			b.instance = a;
			a = null;
			b = null;
			
			// 假设在此时发生 GC，a 和 b 能否被回收？
			System.gc();
		}
	}
	```
	
	答案是否定的。在上述代码中，a 与 b 引用的对象实例互相持有了对象的引用，因此当我们把对 a 对象与 b 对象的引用去除之后，由于两个对象还存在互相之间的引用，导致两个 Test 对象无法被回收。

##### 可达性分析算法
- 以 GC Roots 为起始点进行搜索，可达的对象都是存活的，不可达的对象可被回收。

	> 以图论角度解释，即没有一条路径可以让 GC Roots 达到这个对象。

	| ![可达性分析算法判断对象是否可回收](img/Cys2018-CS-Notes-Java_4-2.png) |
	| :-: |
	| 图 4-2 可达性分析算法判断对象是否可回收 |

- Java 虚拟机使用该算法来判断对象是否可被回收，GC Roots 一般包含以下内容：
  - 虚拟机栈中局部变量表 (栈帧的局部变量表)  中引用的对象。
  - 本地方法栈中 JNI 中引用的对象。
  - 方法区中类静态属性引用的对象。
  - 方法区中的常量引用的对象。

##### 方法区的回收
- 因为方法区主要存放永久代对象，而永久代对象的回收率比新生代低很多，所以在方法区上进行回收性价比不高。
- 主要是对常量池的回收和对类的卸载。

	> 为了避免内存溢出，在大量使用反射和动态代理的场景都需要虚拟机具备类卸载功能。

- 类的卸载条件很多，需要满足以下三个条件，并且满足了条件也不一定会被卸载：
	- 该类所有的实例都已经被回收，此时堆中不存在该类的任何实例。
	- 加载该类的 ClassLoader 已经被回收。
	- 该类对应的 Class 对象没有在任何地方被引用，也就无法在任何地方通过反射访问该类方法。

##### finalize()
- 类似 C++ 的析构函数，用于关闭外部资源。但是 try-finally 等方式可以做得更好，并且该方法运行代价很高，不确定性大，无法保证各个对象的调用顺序，因此最好不要使用。
- 当一个对象可被回收时，如果需要执行该对象的 finalize() 方法，那么就有可能在该方法中让对象重新被引用，从而实现自救。

	> 自救只能进行一次，若回收的对象之前调用了 finalize() 方法自救，后面回收时不会再调用该方法。

#### 再谈引用类型
- 无论是通过引用计数算法判断对象的引用数量，还是通过可达性分析算法判断对象引用链是否可达，判定对象是否存活都与 `引用` 有关。
- 四种 `引用强度` 依次逐渐减弱：
	- 强引用：被强引用关联的对象不会被回收。
		
		```java
		// 使用 new 一个新对象的方式来创建强引用
		Object obj = new Object();
		```
		
	- 软引用：被软引用关联的对象只有在内存不够的情况下才会被回收。
	
		```java
		// 使用 SoftReference 类来创建软引用
		Object obj = new Object();
		SoftReference<Object> sf = new SoftReference<Object>(obj);
		obj = null;  // 使对象只被软引用关联
		```
	
	- 弱引用：被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前。
	
		```java
		// 使用 WeakReference 类来创建弱引用
		Object obj = new Object();
		WeakReference<Object> wf = new WeakReference<Object>(obj);
		obj = null;
		```
		
	- 虚引用：又称为幽灵引用或者幻影引用，一个对象是否有虚引用的存在，不会对其生存时间造成影响，也无法通过虚引用得到一个对象。
	
		为一个对象设置虚引用的唯一目的是能在这个对象被回收时收到一个系统通知。
		
		```java
		// 使用 PhantomReference 来创建虚引用
		Object obj = new Object();
		PhantomReference<Object> pf = new PhantomReference<Object>(obj, null);
		obj = null;
		```

#### 垃圾收集算法
##### 标记-清除算法
| ![标记-清除算法示意](img/Cys2018-CS-Notes-Java_4-2-1.png) |
| :-: |
| 图 4-2-1 标记-清除算法示意 |

- 在标记阶段，程序会检查每个对象是否为活动对象，如果是活动对象，则程序会在对象头部打上标记。
- 在清除阶段，会进行对象回收并取消标志位。另外，还会判断回收后的分块与前一个空闲分块是否连续，若连续，会合并这两个分块。回收对象就是把对象作为分块，连接到被称为 `空闲链表` 的单向链表，之后进行分配时只需要遍历这个空闲链表，找到分块即可。
- 在分配时，程序会搜索空闲链表寻找空间大于等于新对象大小 size 的块 block。如果它找到的块等于 size，会直接返回这个分块；如果找到的块大于 size，会将块分割成大小为 size 与 (block - size) 的两部分，返回大小为 size 的分块，并把大小为 (block - size) 的块返回给空闲链表。
- 不足：
	- 标记和清除过程效率都不高；
	- 会产生大量不连续的内存碎片，导致无法给大对象分配内存。
	
##### 标记-整理算法
| ![标记-整理算法示意](img/Cys2018-CS-Notes-Java_4-2-2.png) |
| :-: |
| 图 4-2-2 标记-整理算法 |

- 让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。
- 优点:不会产生内存碎片。
- 不足:需要移动大量对象，处理效率比较低。

##### 复制算法
| ![复制算法示意](img/Cys2018-CS-Notes-Java_4-2-3.png) |
| :-: |
| 图 4-2-3 复制算法 |

- 将内存划分为大小相等的两块，每次只使用其中一块，当这一块内存用完了就将还存活的对象复制到另一块上面，然后再把使用过的内存空间进行一次清理。
- 不足：只使用了内存的一半。
- 现在的商业虚拟机都采用这种收集算法回收新生代，但是并不是划分为大小相等的两块，而是一块较大的 Eden 空间和两块较小的 Survivor 空间，每次使用 Eden 和其中一块 Survivor。在回收时，将 Eden 和 Survivor 中还存活着的对象全部复制到另一块 Survivor 上，最后清理 Eden 和使用过的那一块 Survivor。
	
	HotSpot 虚拟机的 Eden 和 Survivor 大小比例默认为 8:1，保证了内存的利用率达到 90%。我们没有办法保证每次回收都只有不多于 10% 的对象存活，那么一块 Survivor 就不够用了，此时需要依赖于老年代进行空间分配担保，也就是借用老年代的空间存储放不下的对象。

##### 分代收集算法
| ![分代收集算法示意](img/Cys2018-CS-Notes-Java_4-2-4.png) |
| :-: |
| 图 4-2-4 分代收集算法 |

- 现在的商业虚拟机采用分代收集算法，它根据对象存活周期将内存划分为几块，不同块采用适当的收集算法。

- 一般将堆分为新生代和老年代：
	- 新生代中对象存活率低，使用 `复制算法`。
	- 老年代中对象存活率高、没有额外空间对它进行分配担保，使用 `标记-清除` 算法或者 `标记-整理` 算法。

#### HotSpot 算法实现
##### 根节点枚举
- 使用可达性分析算法，从一系列 GCRoot 对象开始，向下搜索引用链，若一个对象没有与任何 GCRoot 对象关联，这个对象就会被判定为可回收对象。

	这一过程称为 `根节点枚举`，也就是垃圾回收中的 `标记过程`。当前所有的垃圾收集器，在标记阶段都必须停止所有 Java 执行线程 (Stop the wrold, STW)，以保证对象引用状态不会发生变化。

- HotSpot 虚拟机使用的是准确式 GC，当执行系统停顿下来后，并不需要一个不漏地检查完所有执行上下文和全局的引用位置，而是维护了一个专门的映射表 `OopMap` 记录哪些地方存放着对象引用，来快速完成根节点枚举过程。

	在类加载完成时，HotSpot 就会把对象内某个偏移位置是否为对象引用记录下来，JIT 编译过程中，也会在特定的位置记录下栈和局部变量表中哪些位置是引用。

##### 安全点 SafePoint
- 在 OopMap 协助下，HotSpot 可快速且准确地完成 GC Roots 枚举。但是 OopMap 内容变化的指令非常多，如果为每一条指令都生成对应的 OopMap，那么将需要大量的额外空间。为每一个操作记录 OopMap 不现实，为此 HotSpot 虚拟机引入了安全点 (SafePoint) 的概念。

- SafePoint 是程序中的某些位置，线程执行到这些位置时，线程中的某些状态是确定的，在 SafePoint 可以记录 OopMap 信息，线程在 SafePoint 停顿，虚拟机进行 GC。

- 对于一个线程来说，可以处于 SafePoint 上，也可以不处 SafePoint 上。一个线程在 SafePoint 时，它的状态可以安全地被其他 JVM 线程所操作和观测。

- SafePoint 如何在 GC 发生时让所有线程 (不包括执行 JNI 调用的线程) 能执行在最近的安全点上停顿下来，这里有两种方案可供选择：
	- 抢先式中断 (Preemptive Suspension)：JVM 需要GC时，中断所有线程，让没有到达 SafePoint 的线程继续执行至 SafePoint 并中断。
	- 主动式中断 (Voluntary Suspension)：在内存中设置标志位，各线程执行时主动式去轮询这个标志，发现中断标志为真时就自己中断挂起。

##### 安全区 SafeRegion
- SafePoint 无法解决线程未达到 SafePoint 并处于休眠或等待状态的情况，此时引入安全区域 (SafeRegion) 的概念。
- SafeRegion 是代码中的一块区域或线程的状态。在 SafeRegion 中，线程执行与否不会影响对象引用的状态。线程进入 SafeRegion 会给自己加标记，告诉虚拟机可以进行GC；线程准备离开 SafeRegion 前会询问虚拟机 GC 是否完成。

#### 垃圾收集器
> 以下垃圾收集器是基于 JDK 1.7 展开介绍的。

| ![HotSpot虚拟机的垃圾收集器](img/Cys2018-CS-Notes-Java_4-2-5.png) |
| :-: |
| 图 4-2-5 HotSpot 虚拟机的垃圾收集器 |

- 若 `垃圾收集算法` 是内存回收的 `方法论`，则 `垃圾收集器` 就是内存回收的 `具体实现`。
- 图 4-2-5 展示了 HotSpot 虚拟机中的 7 个垃圾收集器，连线表示垃圾收集器可以 `配合使用`。

	> `配合使用`：指的是在限定的使用场景，新生代和老年代各有垃圾收集器专职负责工作。

- 开始讨论垃圾收集器的语境中，我们需要了解一些名词概念：
	- 单线程与多线程：单线程指的是垃圾收集器只使用一个线程，而多线程使用多个线程。
	- 串行 (Serial)：指垃圾收集器与用户程序交替执行，这意味着在执行垃圾收集的时候需要停顿用户程序。
	- 并行 (Parallel)：指多条垃圾收集线程并行工作，但此时用户线程仍处于等待状态。
	- 并发 (Concurrent)：指的是垃圾收集器和用户程序同时执行，用户程序继续运行，垃圾收集器运行于另一个 CPU 上。但不一定是并行，可能交替执行。

		> 除了 CMS 和 G1 之外，其它垃圾收集器都是以串行的方式执行。

##### Serial 收集器
| ![Serial&SerialOld收集器](img/Cys2018-CS-Notes-Java_4-2-6.png) |
| :-: |
| 图 4-2-6 Serial / Serial Old 收集器运行示意图 |

- Serial 翻译为串行，也就是说它以串行的方式执行。
- 它是 `单线程` 的收集器，只会使用一个线程进行垃圾收集工作。
- 它是 `Client` 场景下的默认 `新生代收集器`，因为在该场景下内存一般来说不会很大。它收集一两百兆垃圾的停顿时间可以控制在一百多毫秒以内，只要不是太频繁，这点停顿时间是可以接受的。
- 优点：是简单高效，在单个 CPU 环境下，由于没有线程交互的开销，因此拥有最高的单线程收集效率。

##### ParNew 收集器
| ![ParNew&SerialOld收集器](img/Cys2018-CS-Notes-Java_4-2-7.png) |
| :-: |
| 图 4-2-7 ParNew / Serial Old 收集器运行示意图 |

- 它是 Serial 收集器的 `多线程` 版本。
- 它是 `Server` 场景下默认的 `新生代收集器`，除了性能原因外 (Serial 收集器)，它还能与 CMS 收集器配合使用。

##### Parallel Scavenge 收集器
| ![ParallelScavenge&ParallelOld收集器](img/Cys2018-CS-Notes-Java_4-2-8.png) |
| :-: |
| 图 4-2-8 Parallel Scavenge / Parallel Old 收集器运行示意图 |

- 与 ParNew 一样是 `多线程` 收集器。

- 其它收集器目标是尽可能缩短垃圾收集时用户线程的停顿时间，而它的目标是达到一个可控制的吞吐量，因此它被称为 `吞吐量优先` 收集器。

	> 吞吐量 = 运行用户代码的时间 / (运行用户代码时间 + 垃圾收集时间)，即 CPU 用于运行用户代码的时间与 CPU 总消耗时间的比值。

- 停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验。而高吞吐量则可以高效率地利用 CPU 时间，尽快完成程序的运算任务，适合在后台运算而不需要太多交互的任务。

	缩短停顿时间是以牺牲吞吐量和新生代空间来换取的：新生代空间变小，垃圾回收变得频繁，导致吞吐量下降。

- 可以通过一个开关参数打开 GC 自适应的调节策略 (GC Ergonomics)，就不需要手工指定新生代的大小 (-Xmn)、Eden 和 Survivor 区的比例、晋升老年代对象年龄等细节参数了。虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量。

##### Serial Old 收集器
- 是 Serial 收集器的 `老年代版本`，也是给 `Client` 场景下的虚拟机使用。如果用在 Server 场景下，它有两大用途：
	- 在 JDK 1.5 以及之前版本 (Parallel Old 诞生以前) 与 Parallel Scavenge 收集器搭配使用。
	- 作为 CMS 收集器的后备预案，在并发收集发生 Concurrent Mode Failure 时使用。

##### Parallel Old 收集器
- 是 Parallel Scavenge 收集器的 `老年代版本`。
- 在注重吞吐量以及 CPU 资源敏感的场合，都可以优先考虑 Parallel Scavenge 加 Parallel Old 收集器。

##### CMS 收集器
| ![CMS收集器](img/Cys2018-CS-Notes-Java_4-2-9.png) |
| :-: |
| 图 4-2-9 CMS 收集器运行示意图 |

- CMS (Concurrent Mark Sweep)，`Mark Sweep` 指的是 `标记-清除` 算法。
- CMS 运作过程可分为以下四个流程：
	- `初始标记`：仅是标记一下 GC Roots 能直接关联到的对象，速度很快，需要停顿。
	- `并发标记`：进行 GC Roots Tracing 的过程，它在整个回收过程中耗时最长，不需要停顿。
	- `重新标记`：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。
	- `并发清除`：不需要停顿。
	
		> 在整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，不需要进行停顿。

- 缺点：
	- 吞吐量低：低停顿时间是以牺牲吞吐量为代价的，导致 CPU 利用率不够高。
	- 无法处理浮动垃圾，可能出现 Concurrent Mode Failure。失败导致另一次 Full GC 的产生。

		> 浮动垃圾是指并发清除阶段由于用户线程继续运行而产生的垃圾，这部分垃圾只能到下一次 GC 时才能进行回收。由于浮动垃圾的存在，因此需要预留出一部分内存，意味着 CMS 收集不能像其它收集器那样等待老年代快满的时候再回收。如果预留的内存不够存放浮动垃圾，就会出现 Concurrent Mode Failure，这时虚拟机将临时启用 Serial Old 来替代 CMS。

	- 标记-清除算法导致的空间碎片，往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象，不得不提前触发一次 Full GC。

##### G1 收集器
| ![G1收集器](img/Cys2018-CS-Notes-Java_4-2-10.png) |
| :-: |
| 图 4-2-10 G1 收集器运行示意图 |

- G1 (Garbage-First)，它是一款面向服务端应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能。HotSpot 开发团队赋予它的使命是未来可以替换掉 CMS 收集器。

- G1 把堆划分成多个大小相等的 `独立区域` (Region)，继续保留新生代和老年代的概念，但新生代和老年代不再 `物理隔离`。
	- 通过引入 Region 的概念，从而将原来的一整块内存空间划分成多个的小空间，使得每个小空间可以单独进行垃圾回收。
	- 这种划分方法带来了很大的灵活性，使得 `可预测的停顿时间模型` 成为可能。通过记录每个 Region 垃圾回收时间以及回收所获得的空间 (这两个值是通过过去回收的经验获得)，并维护一个 `优先列表`，每次根据允许的收集时间，优先 `回收价值最大` 的 Region。
	
		> Tips：Garbage-First 名称的由来。
	
	- 每个 Region 都有一个 Remembered Set，用来记录该 Region 对象的引用对象所在的 Region。通过使用 Remembered Set，在做 `可达性分析` 的时候就可以 `避免全堆扫描`。
	
- 如果不计算维护 Remembered Set 的操作，G1 收集器的运作大致可划分为以下几个步骤：
	- `初始标记`：仅是标记一下 GC Roots 能直接关联到的对象，速度很快，需要停顿。
	- `并发标记`：从 GC Root 开始对堆中对象进行 `可达性分析`，找出存活对象，该阶段耗时较长，但可与用户程序并发执行，不需要停顿。
	- `最终标记`：为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的 Remembered Set Logs 里面，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中。这阶段需要停顿线程，但是可并行执行。
	- `筛选回收`：首先对各个 Region 中的 `回收价值和成本进行排序`，根据用户所期望的 GC 停顿时间来制定回收计划。此阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分 Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率。
	
- 具备如下特点：
	- `分代收集`：堆被分为新生代和老年代，其它收集器进行收集的范围都是整个新生代或者老年代。而 G1 可以不依赖其他收集器，直接对新生代和老年代一起回收。
	- `空间整合`：整体来看是基于 `标记-整理算法` 实现的收集器，从局部 (两个 Region 之间) 上来看是基于 `复制算法` 实现的，这意味着运行期间不会产生内存空间碎片。
	- `可预测的停顿`：能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在 GC 上的时间不得超过 N 毫秒。

##### 垃圾收集器总结
- 综上所述，7 种垃圾收集器大致的细节差异，如表 4-2-1 所示：

	| 收集器 | 架构模式 | 分代收集 | 运行方式 | 线程环境 | 适用场景 |
	| :---: | :---: | :---: | :---: | :---: | :--- |
	| Serial | Client | 新生代 | 串行 | 单线程 | -- |
	| ParNew | Server | 新生代 | 并行 | 多线程 | -- |
	| Parallel Scavenge | -- | 新生代 | 并行 | 多线程 | 吞吐量优先<br>CPU 资源敏感场合  |
	| Serial Old | Client | 老年代 | 串行 | 单线程 | -- |
	| Parallel Old | -- | 老年代 | 并行 | 多线程 | 吞吐量优先<br>CPU 资源敏感场合 |
	| CMS | Server | 老年代 | 并发 | 多线程 | 并发收集、低停顿；<br>会产生内存空间碎片 |
	| G1 | Server | 新 / 老 | 并发 | 多线程 | 并发收集、低停顿；<br>不产生内存空间碎片 |

### 内存分配与回收策略
#### 内存分配策略
##### Minor GC 和 Full GC
- `Minor GC`：回收 `新生代`，因为新生代对象存活时间很短，因此 Minor GC 会频繁执行，执行的速度也比较快。
- `Full GC`：回收 `老年代` 和 `新生代`，老年代对象其存活时间长，因此 Full GC 很少执行，执行速度会比 Minor GC 慢很多。

##### 对象优先在 Eden 分配
- 大多数情况下，对象在新生代 Eden 上分配，当 Eden 空间不够时，发起 Minor GC。

##### 大对象直接进入老年代
- 大对象是指需要连续内存空间的对象，最典型的大对象是那种很长的字符串以及数组。
- 经常出现大对象会提前触发垃圾收集以获取足够的连续空间分配给大对象。
- `-XX:PretenureSizeThreshold`：大于 此值的对象直接在老年代分配，避免在 Eden 和 Survivor 之间的大量内存复制。

##### 长期存活对象进入老年代
- 为对象定义年龄计数器，对象在 Eden 出生并经过 Minor GC 依然存活，将移动到 Survivor 中，年龄就增加 1 岁，增加到一定年龄则移动到老年代中。
- `-XX:MaxTenuringThreshold`：用来定义年龄的阈值。

##### 动态对象年龄判定
- 虚拟机并不是永远要求对象的年龄必须达到 MaxTenuringThreshold 才能晋升老年代，如果在 Survivor 中相同年龄所有对象大小的总和大于 Survivor 空间的一半，则年龄大于或等于该年龄的对象可以直接进入老年代，无需等到 MaxTenuringThreshold 中要求的年龄。

##### 空间分配担保
- 在发生 Minor GC 之前，虚拟机先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果条件成立的话，那么 Minor GC 可以确认是安全的。
- 如果不成立的话虚拟机会查看 HandlePromotionFailure 的值是否允许担保失败，若允许那么就会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小。
	- 如果大于，将尝试着进行一次 Minor GC；
	- 如果小于，或者 HandlePromotionFailure 的值不允许冒险，那么就要进行一次 Full GC。

#### 内存回收策略
- `System.gc()`：调用 System.gc() 只是 `建议` 虚拟机执行 Full GC，但虚拟机不一定真正去执行。

	> 不建议使用这种方式，而是让虚拟机管理内存。

- `老年代空间不足`：
	- 老年代空间不足的常见场景为前文所讲的大对象直接进入老年代、长期存活的对象进入老年代等。
	- 为了避免以上原因引起的 Full GC，应当尽量不要创建过大的对象以及数组。
	- 除此之外，可以通过 `-Xmn` 虚拟机参数调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。
	- 还可以通过` -XX:MaxTenuringThreshold` 调大对象进入老年代的年龄，让对象在新生代多存活一段时间。
- `空间分配担保失败`：使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC。具体内容请参考上面 [空间分配担保](#空间分配担保)。
- `永久代空间不足`：
	- 在 `JDK 1.7` 及以前，HotSpot 虚拟机中的方法区是用永久代实现的，永久代中存放的为一些 Class 的信息、常量、静态变量等数据。
	- 当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也会执行 Full GC。如果经过 Full GC 仍然回收不了，那么虚拟机会抛出 java.lang.OutOfMemoryError。
	- 为避免以上原因引起的 Full GC，可采用的方法为增大永久代空间或转为使用 CMS GC。
- `Concurrent Mode Failure`：执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足（可能是 GC 过程中浮动垃圾过多导致暂时性的空间不足），便会报 Concurrent Mode Failure 错误，并触发 Full GC。

### 虚拟机类加载机制

## Java I/O
