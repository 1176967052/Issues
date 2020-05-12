1. 阻塞队列有哪些实现，怎么实现
   
   1. 阻塞队列的定义
      1. 入队时，如果队列已经满了，就阻塞等待直到队列中有位置可以插入
      
      2. 出队时，如果队列中为空，就阻塞等待直到队列中有数据可以出队列
   
   2. 继承结构
      
      继承Queue有三种：1. Deque双端队列接口  2. BlockingQueue阻塞队列接口 3. AbstractQueue 非阻塞队列类
         
      interface BlockingQueue<E> extends Queue<E>,interface Queue<E> extends Collection<E> ,interface Collection<E> extends Iterable<E>,interface Iterable<T> 
      
      BlockingQueue接口的实现类五个：
      1. ArrayBlockingQueue ：一个由数组支持的有界队列。基于数组实现的阻塞队列，初始化时需要定义数组的大小，也就是队列的大小，所以这个队列是一个有界队列。
         原理：
         实现原理：通过可重入锁ReenTrantLock+Condition 来实现多线程之间的同步效果
         
         入队过程：
         
         add方法：插入成功返回true；插入失败抛异常
         
         put方法：插入元素到尾部，如果失败则调用Condition.await()方法进行阻塞等待，直到被唤醒；
         
         offer方法：插入元素到尾部，如果失败则直接返回false，
         
         offer(timeout)：插入元素到尾部，如果失败则调用Condition.await（timeout）方法进行阻塞等待指定时间，直到被唤醒或阻塞超时，还是失败就返回false
         
         而一旦插入成功，就会唤醒出队的等待操作，执行出队的Condition的signal()方法
         
         出队过程：
         
         主要方法为：poll（）、take（）、remove（）
         
         基本上和入队过程类似，出队结束会唤醒入队的等待操作，执行入队的Condition的signal()方法
         
         而不管是入队操作还是出队操作，都会通过ReentrantLock来控制同步效果，通过两个Condition来控制线程之间的通信效果
         
         另外入队和出队操作分别通过两个索引 takeIndex 和putIndex来指定数组的位置，默认从0开始分别递增，如果达到数组的容量大小，就表示到了数组的边界了，此时再设置index=0，相当于数组是一个环形数组
         
         环形数组的好处是增删数据时不需要挪动数组中的其他数据，只需要改变入队和出队的指针即可。而如果不是环形数组而是顺序数组的话，入队和出队就需要大量移动数据，否则数组空间一下就被用完了，性能较差
           
      2. LinkedBlockingQueue ：一个由链接节点支持的可选有界队列。
         
         基于链表实现的阻塞队列，既然是链表，那么就可以看出这种阻塞队列含有链表的特性，那就是无界。但是实际上LinkedBlockingQueue是有界队列，默认大小是Integer的最大值，而也可以通过构造方法传入固定的capacity大小设置
         
         LinkedBlockingQueue有一个内部类Node，属性有：E ite和Node next，所以可以看出LinkedBlockingQueue是一个单向链表
         
         通过ReentrantLock和Condition来实现多线程之间的同步，而LinkedBlockingQueue却多了一个ReentrantLock，而不是入队和出队共用同一个锁
         
         为什么ArrayBlockingQueue只需要一个ReentrantLock而LinkedBlockingQueue需要两个ReentrantLock呢？
         
         首先，ReentrantLock肯定是越多越好，锁越多那么相同锁的竞争就越少；LinkedBlockingQueue分别有入队锁和出队锁，所以入队和出队的时候不会有竞争锁的关系；而ArrayBlockingQueue只有一个Lock，那么不管是入队还是出队，都需要竞争同一个锁，所以效率会低点。ArrayBlockingQueue是环形数组结构，入队的地址和出队的地址可能是同一个，比如数组table大小为1，那么第一次入队和出队需要操作的位置都是table[0]这个元素，所以入队和出队必须共用同一把锁；而LinkedBlockingQueue是链表形式，内存地址是散列的，入队的元素地址和出队的元素地址永远不可能会是同一个地址。所以可以采用两个锁，分别对入队进行加锁同步和对出队进行加锁同步即可。
         
      3. PriorityBlockingQueue ：一个由优先级堆支持的无界优先级队列。
         
         有优先级的阻塞队列，底层也是通过数组实现，默认初始容量为11，容量不够会自动扩容，扩容的最大值为Integer的最大值-8（有些虚拟机再实现数组头部存储内容所预留的空间），所以基本上可以认为是无界阻塞队列
         
         扩容时的线程安全通过ReentrantLock+CAS+volatile实现
         
         它是有优先级的，既然有优先级就涉及到排序,PriorityBlockingQueue默认采用Comparator，或者存储的元素有自定义的比较器。
         
         存储数据的数组也不是简单的数组，而是采用了二叉堆的数据结构，同时满足完全二叉树+堆的数据结构，PriorityBlockingQueue默认是采用的最小堆，即每次取出的元素都是优先级最小的
         
      3. DelayQueue ：一个由优先级堆支持的、基于时间的调度队列。
      
         延迟队列,顾名思义就是只有当元素达到指定的时间后才可以从队列中取出。
         
         根据这个思路可以满足下面几种需求：
         
         1.定时任务：将任务放入队列中设置时间，循环阻塞地从队列中取任务，当从队列中取出数据就表示时间到了
         
         2.缓存过期：循环从队列中取数据，一旦取出数据就表示数据过期了，直接删除即可
         
         DelayQueue主要也是通过ReentrantLock+Condition来保证线程安全，而内部还采用了ProrityQueue来保证队列的优先级，实际就是按延时的时间来进行排序，延迟时间最短的排在队列的头部，
         
         所以每次从头部获取的元素都是最先会过期的数据。      
      4. SynchronousQueue ：一个利用 BlockingQueue 接口的简单聚集（rendezvous）机制。
      
         SynchonousQueue是比较特殊的阻塞队列，特殊之处就是这个叫队列的队列没有容量，又或者说容量为0，所以一旦有元素插入此队列，由于没有容量，就必须被阻塞直到元素被取出
         
         所以SynchronousQueue更像是一个通道，一端发数据，一端消费数据，数据不可以被堆积，发送方或消费方处理不过来或者是不处理都会导致阻塞
      
      AbstractQueue接口的实现类有两个：
      1. PriorityQueue类实质上维护了一个有序列表。加入到 Queue 中的元素根据它们的天然排序（通过其 java.util.Comparable 实现）或者根据传递给构造函数的 java.util.Comparator 实现来定位。
      2. ConcurrentLinkedQueue是基于链接节点的、线程安全的队列。并发访问不需要同步。因为它在队列的尾部添加元素并从头部删除它们，所以只要不需要知道队列的大小，ConcurrentLinkedQueue 对公共集合的共享访问就可以工作得很好。收集关于队列大小的信息会很慢，需要遍历队列。
        
      阻塞队列的操作：
      1. add        增加一个元索                     如果队列已满，则抛出一个IIIegaISlabEepeplian异常
      2. remove   移除并返回队列头部的元素    如果队列为空，则抛出一个NoSuchElementException异常
      3. element  返回队列头部的元素             如果队列为空，则抛出一个NoSuchElementException异常
      4. offer       添加一个元素并返回true       如果队列已满，则返回false
      5. poll         移除并返问队列头部的元素    如果队列为空，则返回null
      6. peek       返回队列头部的元素             如果队列为空，则返回null
      7. put         添加一个元素                      如果队列满，则阻塞
      8. take        移除并返回队列头部的元素     如果队列为空，则阻塞
      
      remove、element、offer 、poll、peek 其实是属于Queue接口。 
      
      阻塞队列的操作可以根据它们的响应方式分为以下三类：
      1. aad、removee和element操作在你试图为一个已满的队列增加元素或从空队列取得元素时 抛出异常。当然，在多线程程序中，队列在任何时间都可能变成满的或空的，所以你可能想使用offer、poll、peek方法。这些方法在无法完成任务时 只是给出一个出错示而不会抛出异常。
      2. 注意：poll和peek方法出错进返回null。因此，向队列中插入null值是不合法的
      3. 最后，我们有阻塞操作put和take。put方法在队列满时阻塞，take方法在队列空时阻塞。
 
 2. JAVA调用其他语言的库
    JNI(Java Native Interface)，它允许Java代码和其他语言（尤其C/C++）写的代码进行交互，只要遵守调用约定即可。
    1. 编写java类代码 *.java
    2. 编译成字节码  *.class
    3. 产生c/c++头文件 *.h
    4. 编写JNI实现代码 *.c/cpp
    5. 编译成链接库文件 *.dll/so
    
 3. 红黑树
    二叉查找树（BST）具备什么特性呢？
    1. 左子树上所有结点的值均小于或等于它的根结点的值。
    2. 右子树上所有结点的值均大于或等于它的根结点的值。
    3. 左、右子树也分别为二叉排序树。
    
    红黑树是一种自平衡的二叉查找树，除了符合二叉查找树的特性，还有附加特性
    1. 节点是红色或黑色。
    2. 根节点是黑色。
    3. 每个叶子节点都是黑色的空节点（NIL节点）。
    4. 每个红色节点的两个子节点都是黑色。(从每个叶子到根的所有路径上不能有两个连续的红色节点)
    5. 从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。
    
    新插入的节点都是红色
    调整方式：变色和旋转
    左旋转：逆时针旋转红黑树的两个节点，使得父节点被自己的右孩子取代，而自己成为自己的左孩子。
    右旋转：顺时针旋转红黑树的两个节点，使得父节点被自己的左孩子取代，而自己成为自己的右孩子。
 4. java访问修饰词有哪些和区别
    1. private(当前类访问权限)    同一个类
    2. default（包访问权限）      同一个类，同包
    3. protected（子类访问权限）  同一个类，同包，不同包子类
    4. public（公共访问权限）     同一个类，同包，不同包子类，不同包非子类
 5. hashmap实现     
 6. hashmap底层实现及不同版本区别,扩容，冲突解决，不同版本实现
 7. arrayList,linkedList，Queue实现
 8. 装饰器模式，工厂模式,开闭原则，builder模式
 9. java装箱拆箱，原理，缓存
    装箱就是  自动将基本数据类型转换为包装器类型；拆箱就是  自动将包装器类型转换为基本数据类型。
    Integer.valueOf
    Integer.intValue
    自动装箱时编译器调用valueOf将原始类型值转换成对象，同时自动拆箱时，编译器通过调用类似intValue(),doubleValue()这类的方法将对象转换成原始类型值。
    
    Integer i1 = 40; 自动装箱，相当于调用了Integer.valueOf(40);方法。
        首先判断i值是否在-128和127之间，如果在-128和127之间则直接从IntegerCache.cache缓存中获取指定数字的包装类；不存在则new出一个新的包装类。
        IntegerCache内部实现了一个Integer的静态常量数组，在类加载的时候，执行static静态块进行初始化-128到127之间的Integer对象，存放到cache数组中。cache属于常量，存放在java的方法区中。
 10. 抽象类和接口区别
     1. 接口的方法默认是 public，所有方法在接口中不能有实现(Java 8 开始接口方法可以有默认实现)，抽象类可以 有非抽象的方法
     2. 接口中的实例变量默认是 final 类型的，而抽象类中则不一定
     3. 一个类可以实现多个接口，但最多只能实现一个抽象类
     4. 一个类实现接口的话要实现接口的所有方法，而抽象类不一定
     5. 接口不能用 new 实例化，但可以声明，但是必须引用一个实现该接口的对象 从设计层面来说，抽象是对类的抽象，是一种模板设计，接口是行为的抽象，是一种行为的规范。

 11. Rest请求权限隔离
     REST比较重要的点是资源和状态转换，所谓"资源"，就是网络上的一个实体，或者说是网络上的一个具体信息。它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的实在。而 “状态转换”，则是把对应的HTTP协议里面，四个表示操作方式的动词分别对应四种基本操作：
     GET，用来浏览(browse)资源
     POST，用来新建(create)资源
     PUT，用来更新(update)资源
     DELETE，用来删除(delete)资源
     
     RESTful 的核心思想就是，客户端发出的数据操作指令都是"动词 + 宾语"的结构。比如，GET /articles这个命令，GET是动词，/articles是宾语。
     有些客户端只能使用GET和POST这两种方法。服务器必须接受POST模拟其他三个方法（PUT、PATCH、DELETE）。
     
     这时，客户端发出的 HTTP 请求，要加上X-HTTP-Method-Override属性，告诉服务器应该使用哪一个动词，覆盖POST方法。
     
     ** 网络上的所有事物都被抽象为资源**
     
     ** 每个资源都有一个唯一的资源标识符**
     
     ** 同一个资源具有多种表现形式(xml,json等)**
     
     ** 对资源的各种操作不会改变资源标识符**
     
     ** 所有的操作都是无状态的**
     
     ** 符合REST原则的架构方式即可称为RESTful**
     
     “权限”，就是资源与操作的一套组合，例如"增加用户"是一种权限，"删除用户"是一种权限，所以对于一种资源所对应的权限有且只有四种。
     对资源的操作，无非就是分为四种：
     浏览 (browse)
     新增 (create)
     更新 (update)
     删除 (delete)
 12. hashmap的put操作什么情况下可以使用cas
     jdk1.7当我们使用put方法的时候,是对我们的key进行hash拿到一个整型,然后将整型对16取模,拿到对应的Segment,之后调用Segment的put方法,然后上锁,请注意,这里lock()的时候其实是this.lock(),也就是说,每个Segment的锁是分开的
 13. java常用集合，Map,以及feature   
     集合collection: List(ArrayList、Vector、LinkedList)，Set(HashSet、TreeSet)
     Map:HashMap,LinkedHashMap
     
     Future表示一个可能还没有完成的异步任务的结果，针对这个结果可以添加Callback以便在任务执行成功或失败后作出相应的操作。
     1. RunnableFuture
        这个接口同时继承Future接口和Runnable接口，在成功执行run（）方法后，可以通过Future访问执行结果。这个接口都实现类是FutureTask,一个可取消的异步计算，这个类提供了Future的基本实现，后面我们的demo也是用这个类实现，它实现了启动和取消一个计算，查询这个计算是否已完成，恢复计算结果。计算的结果只能在计算已经完成的情况下恢复。如果计算没有完成，get方法会阻塞，一旦计算完成，这个计算将不能被重启和取消，除非调用runAndReset方法。
        
        FutureTask能用来包装一个Callable或Runnable对象，因为它实现了Runnable接口，而且它能被传递到Executor进行执行。为了提供单例类，这个类在创建自定义的工作类时提供了protected构造函数。
     2. SchedualFuture
        这个接口表示一个延时的行为可以被取消。通常一个安排好的future是定时任务SchedualedExecutorService的结果
     3. CompleteFuture
        一个Future类是显示的完成，而且能被用作一个完成等级，通过它的完成触发支持的依赖函数和行为。当两个或多个线程要执行完成或取消操作时，只有一个能够成功。
     4. ForkJoinTask
        基于任务的抽象类，可以通过ForkJoinPool来执行。一个ForkJoinTask是类似于线程实体，但是相对于线程实体是轻量级的。大量的任务和子任务会被ForkJoinPool池中的真实线程挂起来，以某些使用限制为代价。
     
     Future接口主要包括5个方法
     1. get（）方法可以当任务结束后返回一个结果，如果调用时，工作还没有结束，则会阻塞线程，直到任务执行完毕
     2. get（long timeout,TimeUnit unit）做多等待timeout的时间就会返回结果
     3. cancel（boolean mayInterruptIfRunning）方法可以用来停止一个任务，如果任务可以停止（通过mayInterruptIfRunning来进行判断），则可以返回true,如果任务已经完成或者已经停止，或者这个任务无法停止，则会返回false.
     4. isDone（）方法判断当前方法是否完成
     5. isCancel（）方法判断当前方法是否取消   
 14. JWT (xxxxx.yyyyy.zzzzz)
     Header  header典型的由两部分组成：token的类型（“JWT”）和算法名称（比如：HMAC SHA256或者RSA等等）。
             用Base64对这个JSON编码就得到JWT的第一部分
     Payload 它包含声明（要求）。声明是关于实体(通常是用户)和其他数据的声明。声明有三种类型: registered, public 和 private。
             Registered claims : 这里有一组预定义的声明，它们不是强制的，但是推荐。比如：iss (issuer), exp (expiration time), sub (subject), aud (audience)等。
             Public claims : 可以随意定义。
             Private claims : 用于在同意使用它们的各方之间共享信息，并且不是注册的或公开的声明。
             对payload进行Base64编码就得到JWT的第二部分
     Signature  为了得到签名部分，你必须有编码过的header、编码过的payload、一个秘钥，签名算法是header中指定的那个，然对它们签名即可。      