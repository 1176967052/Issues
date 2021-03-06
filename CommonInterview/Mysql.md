#### MySql:

1. MyISAM和InnoDB的对比与总结
   + count运算的区别
   + 是否支持事务和崩溃后的安全修复
   + 是否支持外键  
   + innoDB是拷贝数据文件、备份 binlog，或者用 mysqldump，支持灾难恢复（仅需几分钟），MyISAM不支持，遇到数据崩溃，基本上很难恢复，所以要经常进行数据备份。
     
     6.锁的支持。**MyISAM只支持表锁。InnoDB支持表锁、行锁 行锁大幅度提高了多用户并发操作的新能。但是InnoDB的行锁，只是在WHERE的主键是有效的，非主键的WHERE都会锁全表的
  *总结：MyISAM适合没有事务且读密集的表，InnoDB适合有事务支持且写密集的表*
2. 数据库索引
     + 为什么使用索引
        + 创建唯一索引，保证数据库中每行数据的唯一性
        + 大大加快检索速度（大大减少检索的数据量），是主要原因
        + 帮助服务器避免排序和临时表
        + 将随机IO变为顺序IO
        + 加速表与表之间的连接
     + 索引优点这么多，为什么不对表中每列创建一个索引
        + 表中数据增，删，改的时候，要动态维护索引，降低了数据维护速度
        + 索引需要占用物理空间
        + 创建和维护索引需要耗费时间，而且时间随数据量的增加而增加
     + 索引如何提高查询速度
     
         将无序的数据变成相对有序的数据，就像查询目录一样
         
     + 使用索引注意事项
        + 在经常需要搜索的列上使用
        + 在经常使用的where子句中的列上创建索引，加快条件的判断速度
        + 在经常需要排序的列上创建索引，因为索引已经排
        + 在经常用在连接的列上，这些列主要是一些外键，可以加快连接速度
        + 对中，大型表创建索引很有效，但是特大型表维护开销很大，不适合建索引
        + 避免where子句中对字段施加函数，造成无法命中索引
        + 在使用InnoDB时使用与业务无关的自增主键昨晚主键，即使用逻辑主键，而不使用业务主键
        + 将打算加索引的列设置为not null，否则将导致引擎放弃使用索引而全表扫描
        + 删除长期未使用的索引（查询sys库的chema_unused_indexes视图来查询哪些索引从未被使用）
        + 在使用limit offset查询缓慢时，可借助索引
        + in 和 not in 也要慎用，否则会导致全表扫描,对于连续的数值，能用 between 就不要用 in 了,很多时候用 exists 代替 in 是一个好的选择
     + 索引的两种数据结构
        + 哈希索引 ：底层是哈希表，绝大多数是单条记录查询可以使用，查询性能最快，其他建议BTree索引
        + BTree索引：底层是B+树，但两种存储引擎实现方式不一样
     + MyISAM和InnoDB实现BTree索引方式的区别
      
     + 覆盖索引
     
3. 为什么索引能提高查询速度
   + mysql的基本存储结构是页（记录都存在页里）
   + 各个数据页组成一个双向链表
   + 每个数据页中的记录又可以组成一个单向链表
   
    每个数据页都会为存储在它里边儿的记录生成一个页目录，在通过主键查找某条记录的时候可以在页目录 中使用二分法快速定位到对应的槽，然后再遍历该槽对应分组中的记录即可快速找到指定的记录 以其他列(非主键)作为搜索条件:只能从最小记录开始依次遍历单链表中的每条记录。
所以说，如果我们写select * from user where indexname = 'xxx'这样没有进行任何优化的sql语句，默认会这样 做:
   1. 定位到记录所在的页:需要遍历双向链表，找到所在的页
   2. 从所在的页内中查找相应的记录:由于不是根据主键查询，只能遍历所在页的单链表了
   
   很明显，在数据量很大的情况下这样查找会很慢!这样的时间复杂度为O(n)。 使用索引之后 索引做了些什么可以让我们查询加快速度呢?其实就是将无序的数据变成有序(相对):
很明显的是:没有用索引我们是需要遍历双向链表来定位对应的页，现在通过 “目录” 就可以很快地定位到对应的页 上了!(二分查找，时间复杂度近似为O(logn))

4. 最左前缀原则

   MySQL中的索引可以以一定顺序引用多列，这种索引叫作联合索引。如User表的name和city加联合索引就是 (name,city)o而最左前缀原则指的是，如果查询的时候查询条件精确匹配索引的左边连续一列或几列，则此列就可以 被用到。如下:
   1. select * from user where name=xx and city=xx ; //可以命中索引 
   2. select * from user where name=xx ; // 可以命中索引
   3. select * from user where city=xx; // 无法命中索引

   这里需要注意的是，查询的时候如果两个条件都用上了，但是顺序不同，如 city= xx and name =xx ，那么现在 的查询引擎会自动优化为匹配联合索引的顺序，这样是能够命中索引的.
由于最左前缀原则，在创建联合索引时，索引字段的顺序需要考虑字段值去重之后的个数，较多的放前面。 ORDERBY子句也遵循此规则。
注意避免冗余索引

   冗余索引指的是索引的功能相同，能够命中 就肯定能命中 ，那么 就是冗余索引如(name,city )和(name )这两 个索引就是冗余索引，能够命中后者的查询肯定是能够命中前者的 在大多数情况下，都应该尽量扩展已有的索引而 不是创建新索引。
MySQLS.7 版本后，可以通过查询 sys 库的 schemal_r dundant_indexes 表来查看冗余索引

5. 大表优化手段
   + 限定数据的范围（条件范围查询）
   + 读写分离，主库负责写，从库负责读
   + 垂直拆分表 
     + 优点：可以使得行数据变小，在查询时减少读取的Block数，减少I/O次数。此外，垂直分区可以简 化表的结构，易于维护。
     + 缺点：主键会出现冗余，需要管理冗余列，并会引起Join操作，可以通过 在应用层进行Join来解决。此外，垂直分区会让事务变得更加复杂
   + 水平拆分表，最好分库 
     + 优点：保持数据结构不变，进行数据分片，达到分布式目的，可以支撑非常大的数据量，应用端改造也少
     + 缺点：分片事务难以解决，跨节点join性能较差，逻辑复杂，增加逻辑，部署，运维的各种复杂度
     
     常见的数据库分片方案：
     
        + 客户端代理: 分片逻辑在应用端，封装在jar包中，通过修改或者封装JDBC层来实现。 当当网的 Sharding- JDBC 、阿里的TDDL是两种比较常用的实现。
        + 中间件代理: 在应用和数据中间加了一个代理层。分片逻辑统一维护在中间件服务中。 我们现在谈的 Mycat 、360的Atlas、网易的DDB等等都是这种架构的实现。
        
     分库分表后id主键的生成策略:
     
        + 对外提供服务的算法：使用snowflake算法：时间戳+机房号+机器号+序列号
        + 客户端生成的算法： mongodb的ObjectId生成算法：Unix时间戳+机器号+进程号+随机数。无锁，使用CAS来保证数据不重复。
           
6. 重复记录删除只保留一条？

7. MVCC多版本并发控制
  
   为什么使用MVCC？
   
   锁机制可以控制并发操作,但是其系统开销较大,而MVCC可以在大多数情况下代替行级锁,使用MVCC,能降低其系统开销.
   InnoDB的MVCC,是通过在每行记录后面保存两个隐藏的列来实现的,这两个列，分别保存了这个行的创建时间，一个保存的是行的删除时间。
   这里存储的并不是实际的时间值,而是系统版本号(可以理解为事务的ID)，每开始一个新的事务，系统版本号就会自动递增，事务开始时刻的系统版本号会作为事务的ID.


   读锁：也叫共享锁、S锁，若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S 锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。
   写锁：又称排他锁、X锁。若事务T对数据对象A加上X锁，事务T可以读A也可以修改A，其他事务不能再对A加任何锁，直到T释放A上的锁。这保证了其他事务在T释放A上的锁之前不能再读取和修改A。
   表锁：操作对象是数据表。Mysql大多数锁策略都支持，是系统开销最低但并发性最低的一个锁策略。事务t对整个表加读锁，则其他事务可读不可写，若加写锁，则其他事务增删改都不行。
   行级锁：操作对象是数据表中的一行。是MVCC技术用的比较多的。行级锁对系统开销较大，但处理高并发较好。
   
   每行隐式的加两个字段：
   
   DATA_TRX_ID：用来标识最近一次对本行记录做修改(insert|update)的事务的标识符, 即最后一次修改(insert|update)本行记录的事务id。
   DATA_ROLL_PTR：指写入回滚段(rollback segment)的 undo log record (撤销日志记录记录)。如果一行记录被更新, 则 undo log record 包含 '重建该行记录被更新之前内容' 所必须的信息。
   
   InnoDB会根据以下两个条件检查每行记录: 
   a.InnoDB只会查找版本早于当前事务版本的数据行(也就是,行的系统版本号小于或等于事务的系统版本号)，这样可以确保事务读取的行，要么是在事务开始前已经存在的，要么是事务自身插入或者修改过的. 
   b.行的删除版本要么未定义,要么大于当前事务版本号,这可以确保事务读取到的行，在事务开始之前未被删除. 
   只有a,b同时满足的记录，才能返回作为查询结果.
   
   [MVCC详细](https://blog.csdn.net/whoamiyang/article/details/51901888)
   
   [MVCC详细2](https://www.jianshu.com/p/db334404d909)
   
   在insert操作时， “创建时间”=DB_TRX_ID，这时，“删除时间”是未定义的；
   在update操作时，复制新增行的“创建时间”=DB_TRX_ID，删除时间未定义，旧数据行“创建时间”不变，删除时间=该事务DB_TRX_ID；
   在delete操作时，相应数据行的“创建时间”不变，删除时间=该事务的DB_ROW_ID；
   在select操作时，对两者都不修改，只读相应的数据。
   
8. MySQL优化
   1. 超大型数据尽可能尽力不要写子查询，使用连接（JOIN）去替换它
   2. 使用联合(UNION)来代替手动创建的临时表，UNION是会把结果排序的   
      注意：
      1. UNION 结果集中的列名总是等于第一个 SELECT 语句中的列名
      2. UNION 内部的 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 SELECT 语句中的列的顺序必须相同
   
  

   

   
   
   
   
8. 如何实现数据查询时加锁
   for update

9. 事务的隔离级别
   1. READ UNCOMMITTED（未提交读）：事务中的修改，即使没有提交，对其他事务也都是可见的。事务可以读取未提交的数据，这也被称为脏读（Dirty Read）。
   2. READ COMMITTED（提交读）：一个事务从开始直到提交之前，所做的任何修改对其他事务都是不可见的。这个级别有时候叫做不可重复读（nonrepeatble read），因为两次执行同样的查询，可能会得到不一样的结果。
   3. REPEATABLE READ(可重复读)：该隔离级别保证了在同一个事务中多次读取同样记录结果是一致的。MySql默认隔离级别。Mysql的RR是由“行排它锁+MVCC”一起实现的。
   4. SERIALIZABLE（串行化）：SERIALIZABLE是最高的隔离级别。它通过强制事务串行执行，避免了前面说的幻读的问题。SERIALIZABLE会在读取每一行数据都加锁，所以可能导致大量的超时和锁争用问题。

10. ACID
    1. 原子性（Atomicity）：事务开始后所有操作，要么全部做完，要么全部不做，不可能停滞在中间环节。事务执行过程中出错，会回滚到事务开始前的状态，所有的操作就像没有发生一样。也就是说事务是一个不可分割的整体，就像化学中学过的原子，是物质构成的基本单位。
    
    2. 一致性（Consistency）：事务开始前和结束后，数据库的完整性约束没有被破坏 。比如A向B转账，不可能A扣了钱，B却没收到。
    
    3. 隔离性（Isolation）：同一时间，只允许一个事务请求同一数据，不同的事务之间彼此没有任何干扰。比如A正在从一张银行卡中取钱，在A取钱的过程结束前，B不能向这张卡转账。
    
    4. 持久性（Durability）：事务完成后，事务对数据库的所有更新将被保存到数据库，不能回滚。


11. 幻读、脏读、不可重复读
    1. 脏读：B用户并未提交事务，但是A用户却能读到未提交的数据，这就是脏读。
    2. 不可重复读：B用户读取到了A用户提交的数据。在会话B同一个事务中，读取到两次不同的结果。这就造成了不可重复读，就是两次读取的结果不同。
    3. 幻读：指的是当某个事务在读取某个范围内的记录时，另一个事务又在该范围内插入了新的记录，当之前的事务再次读取该范围的记录时，会产生幻行（Phantom Row）。
    不可重复读指的是update操作，而幻读指的是insert或delete操作。

12. sql语句的常见优化策略

13. 数据库三大范式
    
    第一范式：当关系模式R的的所有属性都不能分为更基本的数据单位时，称R是满足第一范式的
         
    + 即每一列属性都是不可再分的属性值，确保每一列的原子性
    
    + 两列的属性相近或相似或一样，尽量合并属性一样的列，确保不产生冗余数据。
    
    第二范式：如果关系模式R满足第一范式，并且R得所有非主属性都完全依赖于R的每一个候选关键属性，称R满足第二范式 
    
    + 每一行的数据只能与其中一列相关，即一行数据只做一件事。只要数据列中出现数据重复，就要把表拆分开来。
    
    第三范式：设R是一个满足第一范式条件的关系模式，X是R的任意属性集，如果X非传递依赖于R的任意一个候选关键字，称R满足第三范式
    
    + 数据不能存在传递关系，即每个属性都跟主键有直接关系而不是间接关系。像：a-->b-->c  属性之间含有这样的关系，是不符合第三范式的。        
































       















