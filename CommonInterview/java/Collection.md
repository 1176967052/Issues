## Collection
1. 说说List,Set,Map三者的区别？
   
   + List(对付顺序的好帮手)： List接口存储一组不唯一（可以有多个元素引用相同的对象），有序的对象
   + Set(注重独一无二的性质): 不允许重复的集合。不会有多个元素引用相同的对象。
   + Map(用Key来搜索的专家): 使用键值对存储。Map会维护与Key有关联的值。两个Key可以引用相同的对象，但Key不能重复，典型的Key是String类型，但也可以是任何对象。

2. Arraylist 与 LinkedList 区别?
    
   1. 是否保证线程安全： ArrayList 和 LinkedList 都是不同步的，也就是不保证线程安全；
   
   2. 底层数据结构： Arraylist 底层使用的是 Object 数组；LinkedList 底层使用的是 双向链表 数据结构（JDK1.6之前为循环链表，JDK1.7取消了循环。注意双向链表和双向循环链表的区别）
   
   3. 插入和删除是否受元素位置的影响： ① ArrayList 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。 比如：执行add(E e) 方法的时候， ArrayList 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是O(1)。但是如果要在指定位置 i 插入和删除元素的话（add(int index, E element) ）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。 ② LinkedList 采用链表存储，所以插入，删除元素时间复杂度不受元素位置的影响，都是近似 O（1）而数组为近似 O（n）。
   
   4. 是否支持快速随机访问： LinkedList 不支持高效的随机元素访问，而 ArrayList 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于get(int index) 方法)。
   
   5. 内存空间占用： ArrayList的空 间浪费主要体现在在list列表的结尾会预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗比ArrayList更多的空间（因为要存放直接后继和直接前驱以及数据）。    

3. ArrayList 与 Vector 区别呢?为什么要用Arraylist取代Vector呢？ 
   
   Vector类的所有方法都是同步的。可以由两个线程安全地访问一个Vector对象、但是一个线程访问Vector的话代码要在同步操作上耗费大量的时间。
   
   Arraylist不是同步的，所以在不需要保证线程安全时时建议使用Arraylist。

4. HashMap 和 HashSet区别  
    
   HashSet 底层就是基于 HashMap 实现的。
   
   + HashMap实现了Map接口;	  HashSet实现Set接口
   + HashMap存储键值对;	HashSet仅存储对象
   + HashMap调用 put（）向map中添加元素;	HashSet调用 add（）方法向Set中添加元素
   + HashMap使用键（Key）计算Hashcode	HashSet使用成员对象来计算hashcode值，对于两个对象来说hashcode可能相同，所以equals()方法用来判断对象的相等性
   
    
   

### List

#### ArrayList(Object数组,线程不安全)
1. 简介
   
   ArrayList 的底层是数组队列，相当于动态数组。它继承于 AbstractList，实现了 List, RandomAccess, Cloneable, java.io.Serializable 这些接口。
   
   线性表的顺序存储，插入删除元素的时间复杂度为O（n）,求表长以及增加元素，取第 i 元素的时间复杂度为O（1）。
   
   + ArrayList 实现了RandomAccess 接口， RandomAccess 是一个标志接口，表明实现这个这个接口的 List 集合是支持快速随机访问的。在 ArrayList 中，我们即可以通过元素的序号快速获取元素对象，这就是快速随机访问。
   
   + ArrayList 实现了Cloneable 接口，即覆盖了函数 clone()，能被克隆。
   
   + ArrayList 实现java.io.Serializable 接口，这意味着ArrayList支持序列化，能通过序列化去传输。
   
   + 和 Vector 不同，ArrayList 中的操作不是线程安全的！所以，建议在单线程中才使用 ArrayList，而在多线程中可以选择 Vector 或者 CopyOnWriteArrayList。
   
2. ArrayList集合实现RandomAccess接口有何作用？为何LinkedList集合却没实现这接口？
   
   RandomAccess接口是一个标志接口（Marker），只要List集合实现这个接口，就能支持快速随机访问。实现RandomAccess接口的List集合采用一般的for循环遍历，而未实现这接口则采用迭代器。
   
   ArrayList用for循环遍历比iterator迭代器遍历快（速度在同一量级），LinkedList用iterator迭代器遍历比for循环遍历快（速度在不同量级）。
   
   ArrayList底层是Object数组，遍历或者查找用下标，可以根据数组的首地址加偏移量直接查找，而用for循环遍历LinkedList,需要遍历链表获取下标，再去查询
   
   实现了 RandomAccess 接口的list，优先选择普通 for 循环 ，其次 foreach,
   
   未实现 RandomAccess接口的list，优先选择iterator遍历（foreach遍历底层也是通过iterator实现的,），大size的数据，千万不要使用普通for循环
   
   用instanceof来判断List集合子类是否实现RandomAccess接口。ArrayList 实现了 RandomAccess 接口，就表明了他具有快速随机访问功能。 RandomAccess 接口只是标识，并不是说 ArrayList 实现 RandomAccess 接口才具有快速随机访问功能的

3. 源码简介
   
   默认初始容量大小 10；空数组
   
   默认构造函数，DEFAULTCAPACITY_EMPTY_ELEMENTDATA 为0.初始化为10，也就是说初始其实是空数组 当添加第一个元素的时候数组容量才变成10
   
   当我们要 add 进第1个元素到 ArrayList 时，elementData.length 为0 （因为还是一个空的 list），因为执行了 ensureCapacityInternal() 方法 ，所以 minCapacity 此时为10。此时，minCapacity - elementData.length > 0 成立，所以会进入 grow(minCapacity) 方法。
   
   当add第2个元素时，minCapacity 为2，此时e lementData.length(容量)在添加第一个元素后扩容成 10 了。此时，minCapacity - elementData.length > 0 不成立，所以不会进入 （执行）grow(minCapacity) 方法。
   
   添加第3、4···到第10个元素时，依然不会执行grow方法，数组容量都为10。
   
   直到添加第11个元素，minCapacity(为11)比elementData.length（为10）要大。进入grow方法进行扩容。
   
   所以 ArrayList 每次扩容之后容量都会变为原来的 1.5 倍！（JDK1.6版本以后） JDk1.6版本时，扩容之后容量为 1.5 倍+1
   
   
4. System.arraycopy()和Arrays.copyOf()方法

   + arraycopy(elementData, index, elementData, index + 1, size - index)需要目标数组，将原数组拷贝到你自己定义的数组里，而且可以选择拷贝的起点和长度以及放入新数组中的位置elementData:源数组;index:源数组中的起始位置;elementData：目标数组；index + 1：目标数组中的起始位置； size - index：要复制的数组元素的数量；
                                                                                                                                 
   + copyOf(elementData, size)是系统自动在内部新建一个数组，并返回该数组。elementData：要复制的数组；size：要复制的长度

5. 注意

   + java 中的length 属性是针对数组说的,比如说你声明了一个数组,想知道这个数组的长度则用到了 length 这个属性.
   
   + java 中的length()方法是针对字 符串String说的,如果想看这个字符串的长度则用到 length()这个方法.
   
   + java 中的size()方法是针对泛型集合说的,如果想看这个泛型有多少个元素,就调用此方法来查看!
   
   


#### Vector(Object数组,线程安全)

#### LinkedList(链表，线程不安全)
1. 简介
   
   LinkedList是一个实现了List接口和Deque接口的双端链表。 LinkedList底层的链表结构使它支持高效的插入和删除操作，另外它实现了Deque接口，使得LinkedList类也具有队列的特性; LinkedList不是线程安全的，如果想使LinkedList变成线程安全的，可以调用静态类Collections类中的synchronizedList方法：
   
   List list=Collections.synchronizedList(new LinkedList(...));
   
2. addAll方法：将集合从指定位置开始插入
   
    + 检查index范围是否在size之内
    + toArray()方法把集合的数据存到对象数组中
    + 得到插入位置的前驱和后继节点
    + 遍历数据，将数据插入到指定位置  

3. 获取头结点
   
    getFirst(),element(),peek(),peekFirst() 这四个获取头结点方法的区别在于对链表为空时的处理，是抛出异常还是返回null，
    其中getFirst() 和element() 方法将会在链表为空时，抛出异常，element()方法的内部就是使用getFirst()实现的。它们会在链表为空时，抛出NoSuchElementException

4. 获取尾节点

   getLast() 方法在链表为空时，会抛出NoSuchElementException，而peekLast() 则不会，只是会返回 null。        
   
   

### Set

#### HashSet(基于 HashMap 实现的，底层采用 HashMap 来保存元素)
1. 如何检查重复
   
   当你把对象加入HashSet时，HashSet会先计算对象的hashcode值来判断对象加入的位置，同时也会与其他加入的对象的hashcode值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用equals（）方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让加入操作成功。
  
   hashCode（）与equals（）的相关规定：
   + 如果两个对象相等，则hashcode一定也是相同的
   + 两个对象相等,对两个equals方法返回true
   + 两个对象有相同的hashcode值，它们也不一定是相等的
   + 综上，equals方法被覆盖过，则hashCode方法也必须被覆盖
   + hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写hashCode()，则该class的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。
  
   ==与equals的区别:
  
   + ==是判断两个变量或实例是不是指向同一个内存空间 equals是判断两个变量或实例所指向的内存空间的值是不是相同
   + ==是指对内存地址进行比较 equals()是对字符串的内容进行比较
   + ==指引用是否相同 equals()指的是值是否相同
   

#### LinkedHashSet(LinkedHashSet 继承与 HashSet)

#### TreeSet(红黑树(自平衡的排序二叉树。))
 


### Map

#### HashMap
1. jdk1.8之前
   
   数组+链表 结合成的散列链表 key.hashCode()经过扰动函数得到hash值，然后通过(n-1)&hash判断当前元素存在的位置（n是数组长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。
   
   n为2幂次方，则n-1的二进制位全为1，进行&运算能减少hash()的冲突
   
   所谓扰动函数指的就是 HashMap 的 hash 方法。使用 hash 方法也就是扰动函数是为了防止一些实现比较差的 hashCode() 方法 换句话说使用扰动函数之后可以减少碰撞。

   ```
   static int hash(int h) {
       // This function ensures that hashCodes that differ only by
       // constant multiples at each bit position have a bounded
       // number of collisions (approximately 8 at default load factor).
   
       h ^= (h >>> 20) ^ (h >>> 12);
       return h ^ (h >>> 7) ^ (h >>> 4);
   }
   ```
2. jdk1.8之后
   
   ```
   static final int hash(Object key) {
            int h;
            // key.hashCode()：返回散列值也就是hashcode
            // ^ ：按位异或
            // >>>:无符号右移，忽略符号位，空位都以0补齐
            return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
   }
   ```
   + 相比于 JDK1.8 的 hash 方法 ，JDK 1.7 的 hash 方法的性能会稍差一点点，因为毕竟扰动了 4 次。
   
   + 相比于之前的版本，jdk1.8在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。
   
   所谓 “拉链法” 就是：将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。
 
3. loadFactor加载因子
    
   控制数组存放数据的疏密程度，越趋近于1，表示数组存放的数据越多，也就越密，就容易发生碰撞，会让链表的长度增加。越小，越趋近于0，数组中存放的数据越少，越稀疏，容易造成空间的浪费
   
   太大导致查找效率低，太小导致数组利用率低，默认值0.75是个很好的临界值 

4. DEFAULT_INITIAL_CAPACITY默认初始容量是16(1<<<4)
5. threshold 临界值
   
   threshold = capacity * loadFactor，当Size>=threshold的时候，那么就要考虑对数组的扩增了，也就是说，这个的意思就是 衡量数组是否需要扩增的一个标准。
   
   给定的默认容量为 16，负载因子为 0.75。Map 在使用过程中不断的往里面存放数据，当数量达到了 16 * 0.75 = 12 就需要将当前 16 的容量进行扩容，而扩容这个过程涉及到 rehash、复制数据等操作，所以非常消耗性能。

6. put方法（HashMap只提供了put用于添加元素，putVal方法只是给put方法调用的一个方法，并没有提供给用户使用）
   
    ![put](https://camo.githubusercontent.com/725bb9953df54438fadb74a1bea180ed817b9e27/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f392f322f313635393862663735386337343765363f773d39393926683d36373926663d706e6726733d3534343836)
   
    jdk1.8:

    + 如果定位到的数组位置没有元素 就直接插入。
   
    + 如果定位到的数组位置有元素就和要插入的key比较，如果key相同就直接覆盖，如果key不相同，就判断p是否是一个树节点，如果是就调用e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value)将元素添加进入。如果不是就遍历链表插入(插入的是链表尾部)。    
   
    jdk1.7:
   
    + 如果定位到的数组位置没有元素 就直接插入。
   
    + 如果定位到的数组位置有元素，遍历以这个元素为头结点的链表，依次和插入的key比较，如果key相同就直接覆盖，不同就采用头插法插入元素。
   

7. HashMap多线程操作导致死循环的原因
   
   在多线程下，进行 put 操作会导致 HashMap 死循环，原因在于 HashMap 的扩容 resize()方法。由于扩容是新建一个数组，复制原数据到数组。由于数组下标挂有链表，所以需要复制链表，但是多线程操作有可能导致环形链表。复制链表过程如下:
   
   jdk1.7采用的是头插法，在桶0上要插入的顺序为A,B，线程一准备扩容时，线程二介入，线程二扩容后头插法变为B,A，线程一将A移入新链表，再将B插入表头,即(BA)，由于线程二的原因，b.next->A，继续插入A，然后A.next->B,即(ABA),形成死循环。
   
   jdk1.8已经解决了这个问题，jdk1.8采用的是尾插法

8. get方法
   
   先经过hash()找到数组中对应的桶，如果桶中不止一个节点，先去红黑树中找，没找到再去链表中找

9. resize扩容方法
   
   进行扩容，会伴随着一次重新hash分配，并且会遍历hash表中所有的元素，是非常耗时的。在编写程序中，要尽量避免resize。
   
   第一种：使用默认构造方法初始化HashMap。从前文可以知道HashMap在一开始初始化的时候会返回一个空的table，并且thershold为0。因此第一次扩容的容量为默认值DEFAULT_INITIAL_CAPACITY也就是16。同时threshold = DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR = 12。
   
   第二种：指定初始容量的构造方法初始化HashMap。初始容量会等于threshold，接着threshold = 当前的容量（threshold） * DEFAULT_LOAD_FACTOR。
   
   第三种：HashMap不是第一次扩容。如果HashMap已经扩容过的话，那么每次table的容量以及threshold量为原有的两倍。

   扩容会先新建一个数组，长度为原来的二倍，完成旧表到新表的转移。转移过程为遍历数组的每个桶明，再顺序遍历桶中的外接链表，找到新表的桶位，插入相应的链表
         
10. HashMap的长度为什么是2的幂次方
    
    为了能让 HashMap 存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀。我们上面也讲到了过了，Hash 值的 范围值-2147483648到2147483647，前后加起来大概40亿的映射空间，只要哈希函数映射得比较均匀松散，一般应 用是很难出现碰撞的。但问题是一个40亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之 前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算 方法是“ (n - 1) & hash ”。(n代表数组长度)。这也就解释了 HashMap 的长度为什么是2的幂次方。
    
    这个算法应该如何设计呢?
    
    我们首先可能会想到采用%取余的操作来实现。但是，重点来了:“取余(%)操作中如果除数是2的幂次则等价于与其 除数减一的与(&)操作(也就是说 hash%length==hash&(length-1)的前提是 length 是2的 n 次方;)。” 并且 采 用二进制位操作 &，相对于%能够提高运算效率，这就解释了 HashMap 的长度为什么是2的幂次方。    
   
    在HashMap通过键的哈希值进行定位桶位置的时候，调用了一个indexFor(hash, table.length);方法。
     ```
     static int indexFor(int h, int length) {
             return h & (length-1);
         }
     ```
    哈希值h与桶数组的length-1（实际上也是map的容量-1）进行了一个与操作得出了对应的桶的位置，h & (length-1)。Java的%、/操作比&慢10倍左右，因此采用&运算会提高性能。
    
    通过限制length是一个2的幂数，h & (length-1)和h % length结果是一致的。这就是为什么要限制容量必须是一个2的幂的原因。
    
11. 二进制运算符
    
    & 只有当两个对位数都是1时才为1，否则为0
    
    | 当两个对位数只要有一个是1则为1，否则为0
  
    ^ 只有当两个对位数字不同时为1，相同为0
    
    ~ 若出入0，则输出1；若输入1，则输入0    
    
    << 表示*2，>> 表示/2
    
12. HashMap和HashTable的区别
    
    + 是否线程安全：HashMap 是非线程安全的，HashTable 是线程安全的;HashTable 内部的方法基本都经过 synchronized 修饰。(如果你要保证线程安全的话就使用 ConcurrentHashMap 吧!)
    + 效率: 因为线程安全的问题，HashMap 要比 HashTable 效率高一点。另外，HashTable 基本被淘汰，不要在 代码中使用它;
    + 对Null key 和Null value的支持: HashMap 中，null 可以作为键，这样的键只有一个，可以有一个或多个键 所对应的值为 null。。但是在 HashTable 中 put 进的键值只要有一个 null，直接抛出 NullPointerException
    + 初始容量大小和每次扩充容量大小的不同 : 1创建时如果不指定容量初始值，Hashtable 默认的初始大小为 11，之后每次扩充，容量变为原来的2n+1。HashMap 默认的初始化大小为16。之后每次扩充，容量变为原来 的2倍。2创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而 HashMap 会将其扩充 为2的幂次方大小(HashMap 中的 tableSizeFor() 方法保证，下面给出了源代码)。
    + 底层数据结构: JDK1.8 以后的 HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阈值(默认为 8)时，将链表转化为红黑树，以减少搜索时间。Hashtable 没有这样的机制。
    
13. ConcurrentHashMap和HashTable的区别

    ConcurrentHashMap 和 Hashtable 的区别主要体现在实现线程安全的方式上不同。
    
    + 底层数据结构: JDK1.7的 ConcurrentHashMap 底层采用 分段的数组+链表 实现，JDK1.8 采用的数据结构跟 HashMap1.8的结构一样，数组+链表/红黑二叉树。Hashtable 和 JDK1.8 之前的 HashMap 的底层数据结构类 似都是采用 数组+链表 的形式，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的;
    + 实现线程安全的方式(重要): 1 在JDK1.7的时候，ConcurrentHashMap(分段锁) 对整个桶数组进行了 分割分段(Segment)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁 竞争，提高并发访问率。 
      到了 JDK1.8 的时候已经摒弃了Segment的概念，而是直接用 Node 数组+链表+红黑 树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作。synchronized只锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，效率又提升N倍(JDK1.6以后 对 synchronized锁做了很 多优化) 整个看起来就像是优化过且线程安全的 HashMap，虽然在JDK1.8中还能看到 Segment 的数据结构， 但是已经简化了属性，只是为了兼容旧版本;
      2 Hashtable(同一把锁) :使用 synchronized 来保证线程安全， 效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用 put 添加元素，另一个线程不能使用 put 添加元素，也不能使用 get，竞争会越来越激烈效率越低。
           

#### LinkedHashMap     
1. 简介

   LinkedHashMap 继承自 HashMap，所以它的底层仍然是基于拉链式散列结构即由数组和 链表或红黑树组成。另外，LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以 保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。

#### TreeMap
1. 红黑树(自平衡的排序二叉树)      