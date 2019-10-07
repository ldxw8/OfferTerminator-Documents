# 技术面试必备基础知识-Java-容器

> 在线阅读：[CS-Notes-Java](https://cyc2018.github.io/CS-Notes/#/README?id=%e2%98%95%ef%b8%8f-java)

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