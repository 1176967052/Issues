1. 分类
   + 按照流的流向分，可以分为输入流和输出流；
   + 按照操作单元划分，可以划分为字节流和字符流；
   + 按照流的角色划分为节点流和处理流。
   
   Java Io流共涉及40多个类，这些类看上去很杂乱，但实际上很有规则，而且彼此之间存在非常紧密的联系， Java I0流的40多个类都是从如下4个抽象类基类中派生出来的。
   
   InputStream/Reader: 所有的输入流的基类，前者是字节输入流，后者是字符输入流。
   OutputStream/Writer: 所有输出流的基类，前者是字节输出流，后者是字符输出流。
   ![IO](https://camo.githubusercontent.com/639ec442b39898de071c3e4fd098215fb48f11e9/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f494f2d2545362539332538442545342542442539432545362539362542392545352542432538462545352538382538362545372542312542422e706e67)
   
   按操作对象分类结构图：
   ![IO](https://camo.githubusercontent.com/4a44e49ab13eacac26cbb0e481db73d6d11181b7/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d362f494f2d2545362539332538442545342542442539432545352541462542392545382542312541312545352538382538362545372542312542422e706e67)
   
2. BIO,NIO,AIO 有什么区别?
   
   同步与异步
   
   + 同步： 同步就是发起一个调用后，被调用者未处理完请求之前，调用不返回。
   + 异步： 异步就是发起一个调用后，立刻得到被调用者的回应表示已接收到请求，但是被调用者并没有返回结果，此时我们可以处理其他的请求，被调用者通常依靠事件，回调等机制来通知调用者其返回结果。
   同步和异步的区别最大在于异步的话调用者不需要等待处理结果，被调用者会通过回调等机制来通知调用者其返回结果。
   
   阻塞和非阻塞
   
   + 阻塞： 阻塞就是发起一个请求，调用者一直等待请求结果返回，也就是当前线程会被挂起，无法从事其他任务，只有当条件就绪才能继续。
   + 非阻塞： 非阻塞就是发起一个请求，调用者不用一直等着结果返回，可以先去干其他事情。
   
   举个生活中简单的例子，你妈妈让你烧水，小时候你比较笨啊，在那里傻等着水开（同步阻塞）。等你稍微再长大一点，你知道每次烧水的空隙可以去干点其他事，然后只需要时不时来看看水开了没有（同步非阻塞）。后来，你们家用上了水开了会发出声音的壶，这样你就只需要听到响声后就知道水开了，在这期间你可以随便干自己的事情，你需要去倒水了（异步非阻塞）。
      
   + BIO (Blocking I/O): 同步阻塞I/O模式，数据的读取写入必须阻塞在一个线程内等待其完成。在活动连接数不是特别高（小于单机1000）的情况下，这种模型是比较不错的，可以让每一个连接专注于自己的 I/O 并且编程模型简单，也不用过多考虑系统的过载、限流等问题。线程池本身就是一个天然的漏斗，可以缓冲一些系统处理不了的连接或请求。但是，当面对十万甚至百万级连接的时候，传统的 BIO 模型是无能为力的。因此，我们需要一种更高效的 I/O 处理模型来应对更高的并发量。
   + NIO (New I/O): NIO是一种同步非阻塞的I/O模型，在Java 1.4 中引入了NIO框架，对应 java.nio 包，提供了 Channel , Selector，Buffer等抽象。NIO中的N可以理解为Non-blocking，不单纯是New。它支持面向缓冲的，基于通道的I/O操作方法。 NIO提供了与传统BIO模型中的 Socket 和 ServerSocket 相对应的 SocketChannel 和 ServerSocketChannel 两种不同的套接字通道实现,两种通道都支持阻塞和非阻塞两种模式。阻塞模式使用就像传统中的支持一样，比较简单，但是性能和可靠性都不好；非阻塞模式正好与之相反。对于低负载、低并发的应用程序，可以使用同步阻塞I/O来提升开发速率和更好的维护性；对于高负载、高并发的（网络）应用，应使用 NIO 的非阻塞模式来开发
   + AIO (Asynchronous I/O): AIO 也就是 NIO 2。在 Java 7 中引入了 NIO 的改进版 NIO 2,它是异步非阻塞的IO模型。异步 IO 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作。AIO 是异步IO的缩写，虽然 NIO 在网络操作中，提供了非阻塞的方法，但是 NIO 的 IO 行为还是同步的。对于 NIO 来说，我们的业务线程是在 IO 操作准备好时，得到通知，接着就由这个线程自行进行 IO 操作，IO操作本身是同步的。查阅网上相关资料，我发现就目前来说 AIO 的应用还不是很广泛，Netty 之前也尝试使用过 AIO，不过又放弃了。   
 
3. NIO
   
   包含下面几个核心的组件：
   
   + Channel(通道)
     + Scatter / Gather：Scatter: 从一个Channel读取的信息分散到N个缓冲区中(Buufer)；Gather: 将N个Buffer里面内容按照顺序发送到一个Channel.
     + 通道之间的数据传输：在Java NIO中如果一个channel是FileChannel类型的，那么他可以直接把数据传输到另一个channel；transferFrom() :transferFrom方法把数据从通道源传输到FileChannel；transferTo() :transferTo方法把FileChannel数据传输到另一个channel
   + Buffer(缓冲区)：
     + Java NIO Buffers用于和NIO Channel交互。 我们从Channel中读取数据到buffers里，从Buffer把数据写入到Channels；
     + Buffer本质上就是一块内存区；
     + 一个Buffer有三个属性是必须掌握的，分别是：capacity容量、position位置、limit限制。
   + Selector(选择器)   
     + 多路复用器 。它是Java NIO核心组件中的一个，用于检查一个或多个NIO Channel（通道）的状态是否处于可读、可写。如此可以实现单线程管理多个channels,也就是可以管理多个网络链接。
     + 使用Selector的好处在于： 使用更少的线程来就可以来处理通道了， 相比使用多个线程，避免了线程上下文切换带来的开销。

   
   NIO的特性/NIO与IO区别:
   
   + IO是面向流的，NIO是面向缓冲区的；
   + IO流是阻塞的，NIO流是不阻塞的;
   + NIO有选择器，而IO没有。
   
4. Linux IO模型 [IO模型](https://mp.weixin.qq.com/s?__biz=Mzg3MjA4MTExMw==&mid=2247484746&idx=1&sn=c0a7f9129d780786cabfcac0a8aa6bb7&source=41#wechat_redirect)
   
   Java中的BIO、NIO和AIO理解为是Java语言对操作系统的各种IO模型的封装，IO可以理解为文件从硬盘中转移到用户空间的过程。
   
   在Linux(UNIX)操作系统中，共有五种IO模型，分别是：阻塞IO模型、非阻塞IO模型、IO复用模型、信号驱动IO模型以及异步IO模型。
   
   + 阻塞IO模型
     
     我们钓鱼的时候，有一种方式比较惬意，比较轻松，那就是我们坐在鱼竿面前，这个过程中我们什么也不做，双手一直把着鱼竿，就静静的等着鱼儿咬钩。一旦手上感受到鱼的力道，就把鱼钓起来放入鱼篓中。然后再钓下一条鱼。
     
     映射到Linux操作系统中，这就是一种最简单的IO模型，即阻塞IO。 阻塞 I/O 是最简单的 I/O 模型，一般表现为进程或线程等待某个条件，如果条件不满足，则一直等下去。条件满足，则进行下一步操作。
      
     应用进程通过系统调用 recvfrom 接收数据，但由于内核还未准备好数据报，应用进程就会阻塞住，直到内核准备好数据报，recvfrom 完成数据报复制工作，应用进程才能结束阻塞状态。
     
     这种钓鱼方式相对来说比较简单，对于钓鱼的人来说，不需要什么特制的鱼竿，拿一根够长的木棍就可以悠闲的开始钓鱼了（实现简单）。缺点就是比较耗费时间，比较适合那种对鱼的需求量小的情况（并发低，时效性要求低）。
   
   + 非阻塞IO模型
     
     我们钓鱼的时候，在等待鱼儿咬钩的过程中，我们可以做点别的事情，比如玩一把王者荣耀、看一集《延禧攻略》等等。但是，我们要时不时的去看一下鱼竿，一旦发现有鱼儿上钩了，就把鱼钓上来。
     
     映射到Linux操作系统中，这就是非阻塞的IO模型。应用进程与内核交互，目的未达到之前，不再一味的等着，而是直接返回。然后通过轮询的方式，不停的去问内核数据准备有没有准备好。如果某一次轮询发现数据已经准备好了，那就把数据拷贝到用户空间中。
     
     应用进程通过 recvfrom 调用不停的去和内核交互，直到内核准备好数据。如果没有准备好，内核会返回error，应用进程在得到error后，过一段时间再发送recvfrom请求。在两次发送请求的时间段，进程可以先做别的事情。
     
     这种方式钓鱼，和阻塞IO比，所使用的工具没有什么变化，但是钓鱼的时候可以做些其他事情，增加时间的利用率。
     
   + 信号驱动IO模型
   
     我们钓鱼的时候，为了避免自己一遍一遍的去查看鱼竿，我们可以给鱼竿安装一个报警器。当有鱼儿咬钩的时候立刻报警。然后我们再收到报警后，去把鱼钓起来。
     
     映射到Linux操作系统中，这就是信号驱动IO。应用进程在读取文件时通知内核，如果某个 socket 的某个事件发生时，请向我发一个信号。在收到信号后，信号对应的处理函数会进行后续处理。
     
     应用进程预先向内核注册一个信号处理函数，然后用户进程返回，并且不阻塞，当内核数据准备就绪时会发送一个信号给进程，用户进程便在信号处理函数中开始把数据拷贝的用户空间中。
     
     这种方式钓鱼，和前几种相比，所使用的工具有了一些变化，需要有一些定制（实现复杂）。但是钓鱼的人就可以在鱼儿咬钩之前彻底做别的事儿去了。等着报警器响就行了
     
   + IO复用模型
     
     我们钓鱼的时候，为了保证可以最短的时间钓到最多的鱼，我们同一时间摆放多个鱼竿，同时钓鱼。然后哪个鱼竿有鱼儿咬钩了，我们就把哪个鱼竿上面的鱼钓起来。
     
     映射到Linux操作系统中，这就是IO复用模型。多个进程的IO可以注册到同一个管道上，这个管道会统一和内核进行交互。当管道中的某一个请求需要的数据准备好之后，进程再把对应的数据拷贝到用户空间中。    
   
     IO多路转接是多了一个select函数，多个进程的IO可以注册到同一个select上，当用户进程调用该select，select会监听所有注册好的IO，如果所有被监听的IO需要的数据都没有准备好时，select调用进程会阻塞。当任意一个IO所需的数据准备好之后，select调用就会返回，然后进程在通过recvfrom来进行数据拷贝。
     
     这里的IO复用模型，并没有向内核注册信号处理函数，所以，他并不是非阻塞的。进程在发出select后，要等到select监听的所有IO操作中至少有一个需要的数据准备好，才会有返回，并且也需要再次发送请求去进行文件的拷贝。
     
   + 异步IO模型
   
     我们钓鱼的时候，采用一种高科技钓鱼竿，即全自动钓鱼竿。可以自动感应鱼上钩，自动收竿，更厉害的可以自动把鱼放进鱼篓里。然后，通知我们鱼已经钓到了，他就继续去钓下一条鱼去了。
     
     映射到Linux操作系统中，这就是异步IO模型。应用进程把IO请求传给内核后，完全由内核去操作文件拷贝。内核完成相关操作后，会发信号告诉应用进程本次IO已经完成。
     
     用户进程发起aio_read操作之后，给内核传递描述符、缓冲区指针、缓冲区大小等，告诉内核当整个操作完成时，如何通知进程，然后就立刻去做其他事情了。当内核收到aio_read后，会立刻返回，然后内核开始等待数据准备，数据准备好以后，直接把数据拷贝到用户控件，然后再通知进程本次IO已经完成。
   
   
   
   
      
   