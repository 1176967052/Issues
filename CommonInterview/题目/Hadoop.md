
1. 简介
   Hadoop是目前大数据领域最主流的一套技术体系，包含了多种技术。

   包括HDFS（分布式文件系统），YARN（分布式资源调度系统），MapReduce（分布式计算系统），等等。
2. HDFS
   HDFS全称是Hadoop Distributed File System，是Hadoop的分布式文件系统。
   hadoop中的HDFS，就是大数据技术体系中的核心基石，负责分布式存储数据。
   它由很多机器组成，每台机器上运行一个DataNode进程，负责管理一部分数据。然后有一台机器上运行了NameNode进程，NameNode大致可以认为是负责管理整个HDFS集群的这么一个进程，他里面存储了HDFS集群的所有元数据。
   
   NameNode有一个很核心的功能：管理整个HDFS集群的元数据，比如说文件目录树、权限的设置、副本数的设置，等等。
   文件目录树是HDFS非常核心的一块元数据，维护了HDFS这个分布式文件系统中，有哪些目录，有哪些文件。这个文件目录树是在NameNode的内存里
   万一NameNode不小心宕机了可咋整？元数据不就全部丢失了？可你要是每次都频繁的修改磁盘文件里的元数据，性能肯定是极低的啊！毕竟这是大量的磁盘随机读写！
   
   HDFS优雅的解决方案：
   每次内存里改完了，写一条edits log，元数据修改的操作日志到磁盘文件里，不修改磁盘文件内容，就是顺序追加，这个性能就高多了。
   每次NameNode重启的时候，把edits log里的操作日志读到内存里回放一下，不就可以恢复元数据了？
   但是问题又来了，那edits log如果越来越大的话，岂不是每次重启都会很慢？因为要读取大量的edits log回放恢复元数据！
   引入一个新的磁盘文件叫做fsimage，然后呢，再引入一个JournalNodes集群，以及一个Standby NameNode（备节点）。
   每次Active NameNode（主节点）修改一次元数据都会生成一条edits log，除了写入本地磁盘文件，还会写入JournalNodes集群。
   然后Standby NameNode就可以从JournalNodes集群拉取edits log，应用到自己内存的文件目录树里，跟Active NameNode保持一致。
   然后每隔一段时间，Standby NameNode都把自己内存里的文件目录树写一份到磁盘上的fsimage，这可不是日志，这是完整的一份元数据。这个操作就是所谓的checkpoint检查点操作。
   然后把这个fsimage上传到到Active NameNode，接着清空掉Active NameNode的旧的edits log文件，这里可能都有100万行修改日志了！
   然后Active NameNode继续接收修改元数据的请求，再写入edits log，写了一小会儿，这里可能就几十行修改日志而已！
   如果说此时，Active NameNode重启了，bingo！没关系，只要把Standby NameNode传过来的fsimage直接读到内存里，这个fsimage直接就是元数据，不需要做任何额外操作，纯读取，效率很高！
   然后把新的edits log里少量的几十行的修改日志回放到内存里就ok了！
   这个过程的启动速度就快的多了！因为不需要回放大量上百万行的edits log来恢复元数据了！
   
   现在咱们有俩NameNode。
   
   一个是主节点对外提供服务接收请求
   另外一个纯就是接收和同步主节点的edits log以及执行定期checkpoint的备节点。
   如果Active NameNode挂了，是不是可以立马切换成Standby NameNode对外提供服务？
   这不就是所谓的NameNode主备高可用故障转移机制么！
   
   文件上传原理：
   HDFS客户端会给拆成很多block，一个block就是128MB。
   然后，HDFS客户端把一个一个的block上传到第一个DataNode
   第一个DataNode会把这个block复制一份，做一个副本发送给第二个DataNode。
   第二个DataNode发送一个block副本到第三个DataNode。
   所以你会发现，一个block有3个副本，分布在三台机器上。任何一台机器宕机，数据是不会丢失的。
   
   Hadoop如何将TB级大文件的上传性能优化上百倍？
   1. Chunk缓冲机制
   
   首先，数据会被写入一个chunk缓冲数组，这个chunk是一个512字节大小的数据片段，你可以这么来理解。
   然后这个缓冲数组可以容纳多个chunk大小的数据在里面缓冲。
   
   2. Packet数据包机制
   
   接着，当chunk缓冲数组都写满了之后，就会把这个chunk缓冲数组进行一下chunk切割，切割为一个一个的chunk，一个chunk是一个数据片段。
   然后多个chunk会直接一次性写入另外一个内存缓冲数据结构，就是Packet数据包。
   一个Packet数据包，设计为可以容纳127个chunk，大小大致为64mb。所以说大量的chunk会不断的写入Packet数据包的内存缓冲中。
   通过这个Packet数据包机制的设计，又可以在内存中容纳大量的数据，进一步避免了频繁的网络传输影响性能。
   
   3. 内存队列异步发送机制
   
   当一个Packet被塞满了chunk之后，就会将这个Packet放入一个内存队列来进行排队。
   然后有一个DataStreamer线程会不断的获取队列中的Packet数据包，通过网络传输直接写一个Packet数据包给DataNode。
   如果一个Block默认是128mb的话，那么一个Block默认会对应两个Packet数据包，每个Packet数据包是64MB。
   传送两个Packet数据包给DataNode之后，就会发一个通知说，一个Block的数据都传输完毕。
   
3. 分布式计算系统
   1. 很多公司用Hive写几百行的大SQL（底层基于MapReduce）
   
   2. 也有很多公司开始慢慢的用Spark写几百行的大SQL（底层是Spark Core引擎）。
  
   总之就是写一个大SQL，人家会拆分为很多的计算任务，放到各个机器上去，每个计算任务就负责计算一小部分数据，这就是所谓的分布式计算。   