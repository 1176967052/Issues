1. MySQL索引怎么实现，innodb的B+索引叶子节点存什么
2. 表t(a,b,c,d)有一个联合索引(b,c,d)，查询select * from t where b>10 and c>10 and d>10可以用到索引吗
   用到了索引b，因为b是范围值，断点，阻塞了c的索引
3. 怎么解决幻读问题
   可以认为MVCC是行级锁的一个变种，但是它在很多情况下避免了加锁操作，因此开销更低。虽然实现机制有所不同，但大都实现了非阻塞的读操作，写操作也只锁定必要的行。
   
   MVCC的实现，是通过保存数据在某个时间点的快照来实现的。也就是说，不管需要执行多长时间，每个事务看到的数据都是一致的。根据事务开始的时间不同，每个事务对同一张表，同一时刻看到的数据可能是不一样的。
   
   next key 将当前行数据与上一条数据和下一条数据之间的间隙锁定，保证范围内读取是一定的
   next key 锁包含 记录锁：加载索引上的锁，
                  间隙锁：加载索引之间的锁
4. sql left/right join区别
   
   A INNER JOIN B ON……：内联操作，将符合ON条件的A表和B表结果均搜索出来，然后合并为一个结果集。
   A LEFT JOIN B ON……：左联操作，左联顾名思义是，将符合ON条件的B表结果搜索出来，
   然后左联到A表上，然后将合并后的A表输出。
   A RIGHT JOIN B ON……：右联操作，右联顾名思义是，将符合ON条件的A表结果搜索出来，
   然后右联到B表上，然后将合并后的B表输出。
   
5. mysql隔离级别，原理，解决的问题
   READ UNCOMMITTED(未提交读)：
   
   在READUNCOMMITTED级别，事务中的修改，即使没有提交，对其他事务也都是可见的。事务可以读取未提交的数据，这也被称为脏读（DirtyRead）。这个级别会导致很多问题，从性能上来说，READUNCOMMITTED不会比其他的级别好太多，但却缺乏其他级别的很多好处，除非真的有非常必要的理由，在实际应用中一般很少使用。
   
   READ COMMITTED（ 提交读）
   
   大多数数据库系统的默认隔离级别都是READCOMMITTED（但MySQL不是）。READCOMMITTED满足前面提到的隔离性的简单定义：一个事务开始时，只能“看见”已经提交的事务所做的修改。换句话说，一个事务从开始直到提交之前，所做的任何修改对其他事务都是不可见的。这个级别有时候也叫做不可重复读（nonrepeatableread），因为两次执行同样的查询，可能会得到不一样的结果。
   
   REPEATABLE READ（ 可重复读）
   
   REPEATABLEREAD解决了脏读的问题。该级别保证了在同一个事务中多次读取同样记录的结果是一致的。但是理论上，可重复读隔离级别还是无法解决另外一个幻读（PhantomRead）的问题。所谓幻读，指的是当某个事务在读取某个范围内的记录时，另外一个事务又在该范围内插入了新的记录，当之前的事务再次读取该范围的记录时，会产生幻行（PhantomRow）。InnoDB和XtraDB存储引擎通过多版本并发控制（MVCC，MultiversionConcurrencyControl）解决了幻读的问题。本章稍后会做进一步的讨论。可重复读是MySQL的默认事务隔离级别。
   
    
   SERIALIZABLE（可串行化）
   
   SERIALIZABLE是最高的隔离级别。它通过强制事务串行执行，避免了前面说的幻读的问题。简单来说，SERIALIZABLE会在读取的每一行数据上都加锁，所以可能导致大量的超时和锁争用的问题。实际应用中也很少用到这个隔离级别，只有在非常需要确保数据的一致性而且可以接受没有并发的情况下，才考虑采用该级别。
6. MVCC
   InnoDB的MVCC，是通过在每行记录后面保存两个隐藏的列来实现的。这两个列，一个保存了行的创建时间，一个保存行的过期时间（或删除时间）。当然存储的并不是实际的时间值，而是系统版本号（systemversionnumber）。每开始一个新的事务，系统版本号都会自动递增。事务开始时刻的系统版本号会作为事务的版本号，用来和查询到的每行记录的版本号进行比较。下面看一下在REPEATABLEREAD隔离级别下，MVCC具体是如何操作的。
   
   SELECT    InnoDB会根据以下两个条件检查每行记录：InnoDB只查找版本早于当前事务版本的数据行（也就是，行的系统版本号小于或等于事务的系统版本号），这样可以确保事务读取的行，要么是在事务开始前已经存在的，要么是事务自身插入或者修改过的。行的删除版本要么未定义，要么大于当前事务版本号。这可以确保事务读取到的行，在事务开始之前未被删除。只有符合上述两个条件的记录，才能返回作为查询结果。
   
   INSERT    InnoDB为新插入的每一行保存当前系统版本号作为行版本号。
   
   DELETE    InnoDB为删除的每一行保存当前系统版本号作为行删除标识。
   
   UPDATE   InnoDB为插入一行新记录，保存当前系统版本号作为行版本号，同时保存当前系统版本号到原来的行作为行删除标识。保存这两个额外系统版本号，使大多数读操作都可以不用加锁。这样设计使得读数据操作很简单，性能很好，并且也能保证只会读取到符合标准的行。不足之处是每行记录都需要额外的存储空间，需要做更多的行检查工作，以及一些额外的维护工作。MVCC只在REPEATABLEREAD和READCOMMITTED两个隔离级别下工作。其他两个隔离级别都和MVCC不兼容(4)，因为READUNCOMMITTED总是读取最新的数据行，而不是符合当前事务版本的数据行。而SERIALIZABLE则会对所有读取的行都加锁。
   
7. 分表，hash取模，一致性hash
8. mysql设置最大连接数
   查看：show variables like '%max_connections%';
   两种方法：
   1. 这种 方式在Mysql重启后就失效。
      set GLOBAL max_connections=1024;
      show variables like '%max_connections%';
   2. 修改mysql配置文件my.cnf，在[mysqld]段中添加或修改max_connections值：
      max_connections=512
      重启mysql服务即可。   
9. 设计快速同步表变化
   方法1：触发器同步表
         触发器语句中使用了两种特殊的表：old 表和 new 表
   方法2：binlog同步
         canal,kafka
   方法3：主从同步              
10. mysql like是否用索引，强制走索引操作
    mysql在使用like查询的时候只有使用后面的%时，才会使用到索引。
    强制走索引 SELECT * FROM TABLE1 FORCE INDEX (FIELD1) …
    忽略索引 SELECT * FROM TABLE1 IGNORE INDEX (FIELD1, FIELD2) …
11. 数据库常用同步操作binlog，以及除了binlog规定其他方案
12. 数据库连接池常用配置
    1. maxActive  连接池支持的最大连接数，这里取值为20，表示同时最多有20个数据库连接。一般把maxActive设置成可能的并发量就行了设 0 为没有限制。
    
    2. maxIdle 连接池中最多可空闲maxIdle个连接 ，这里取值为20，表示即使没有数据库连接时依然可以保持20空闲的连接，而不被清除，随时处于待命状态。设 0 为没有限制。
    
    3. minIdle 连接池中最小空闲连接数，当连接数少于此值时，连接池会创建连接来补充到该值的数量
    
    4. initialSize 初始化连接数目 
    
    5. maxWait 连接池中连接用完时,新的请求等待时间,毫秒，这里取值-1，表示无限等待，直到超时为止，也可取值9000，表示9秒后超时。超过时间会出错误信息
    
    6. removeAbandoned  是否清除已经超过“removeAbandonedTimout”设置的无效连接。如果值为“true”则超过“removeAbandonedTimout”设置的无效连接将会被清除。设置此属性可以从那些没有合适关闭连接的程序中恢复数据库的连接。
    
    7. removeAbandonedTimeout 活动连接的最大空闲时间,单位为秒 超过此时间的连接会被释放到连接池中,针对未被close的活动连接
    
    8. minEvictableIdleTimeMillis 连接池中连接可空闲的时间,单位为毫秒 针对连接池中的连接对象
    
    9. timeBetweenEvictionRunsMillis / minEvictableIdleTimeMillis  每timeBetweenEvictionRunsMillis毫秒秒检查一次连接池中空闲的连接,把空闲时间超过minEvictableIdleTimeMillis毫秒的连接断开,直到连接池中的连接数到minIdle为止.
    

13. B+树如何实现行级锁，innodb默认事务机制，最左原则
    行级锁是mysql中粒度最小的一种锁，他能大大减少数据库操作的冲突。但是粒度越小，实现的成本也越高。myisam引擎只支持表级锁，而innodb引擎能够支持行级锁，下面的内容也是针对INNODB行级锁展开的。
    INNODB的行级锁有共享锁（S LOCK）和排他锁（X LOCK）两种。共享锁允许事物读一行记录，不允许任何线程对该行记录进行修改。排他锁允许当前事物删除或更新一行记录，其他线程不能操作该记录。
    共享锁：
    用法： SELECT ... LOCK IN SHARE MODE;
    MySQL会对查询结果集中每行都添加共享锁。
    排他锁：
    用法： SELECT ... FOR UPDATE;
    MySQL会对查询结果集中每行都添加排他锁，在事物操作中，任何对记录的更新与删除操作会自动加上排他锁。
    
    InnoDB行锁是通过给索引项加锁来实现的，这一点mysql和oracle不同，后者是通过在数据库中对相应的数据行加锁来实现的，InnoDB这种行级锁决定，只有通过索引条件来检索数据，才能使用行级锁，否则，直接使用表级锁。特别注意:使用行级锁一定要使用索引

    在不通过索引条件查询的时候，InnoDB使用的是表锁，而不是行锁。
    由于MySQL的行锁是针对索引加的锁，不是针对记录加的锁，所以即使是访问不同行的记录，如果使用了相同的索引键，也是会出现锁冲突的。
    当表有多个索引的时候，不同的事务可以使用不同的索引锁定不同的行，另外，不论是使用主键索引、唯一索引或普通索引，InnoDB都会使用行锁来对数据加锁。
    即便在条件中使用了索引字段，但具体是否使用索引来检索数据是由MySQL通过判断不同执行计划的代价来决定的，如果MySQL认为全表扫描效率更高，比如对一些很小的表，它就不会使用索引，这种情况下InnoDB将使用表锁，而不是行锁。因此，在分析锁冲突时，别忘了检查SQL的执行计划，以确认是否真正使用了索引。

14. mysql支持的索引类型
    MySQL的存储引擎中，MyISAM不支持哈希索引，而InnoDB中的hash索引是存储引擎根据B-Tree索引自建的
15. msyql事务

16. mysql瓶颈
    1. IO瓶颈
       第一种：磁盘读IO瓶颈，热点数据太多，数据库缓存放不下，每次查询时会产生大量的IO，降低查询速度 -> 分库和垂直分表。
       
       第二种：网络IO瓶颈，请求的数据太多，网络带宽不够 -> 分库。
    2. CPU瓶颈
       第一种：SQL问题，如SQL中包含join，group by，order by，非索引字段条件查询等，增加CPU运算的操作 -> SQL优化，建立合适的索引，在业务Service层进行业务计算。
       
       第二种：单表数据量太大，查询时扫描的行太多，SQL效率低，CPU率先出现瓶颈 -> 水平分表。
   
17. 分库分表
    1. 水平分库
       1. 概念：以字段为依据，按照一定策略（hash、range等），将一个库中的数据拆分到多个库中。
       
       2. 结果：
          每个库的结构都一样;
          每个库的数据都不一样，没有交集;
          所有库的并集是全量数据;
       
       3. 场景：系统绝对并发量上来了，分表难以根本上解决问题，并且还没有明显的业务归属来垂直分库。
       
       4. 分析：库多了，io和cpu的压力自然可以成倍缓解。
    2. 水平分表
       1. 概念：以字段为依据，按照一定策略（hash、range等），将一个表中的数据拆分到多个表中。
       
       2. 结果：
          每个表的结构都一样
          每个表的数据都不一样，没有交集;
          所有表的并集是全量数据;
       3. 场景：系统绝对并发量并没有上来，只是单表的数据量太多，影响了SQL效率，加重了CPU负担，以至于成为瓶颈。
       
       4. 分析：表的数据量少了，单次SQL执行效率高，自然减轻了CPU的负担。
    3. 垂直分库
       1. 概念：以表为依据，按照业务归属不同，将不同的表拆分到不同的库中。
       
       2. 结果：
       
          每个库的结构都不一样；
          每个库的数据也不一样，没有交集；
          所有库的并集是全量数据；
       3. 场景：系统绝对并发量上来了，并且可以抽象出单独的业务模块。4.分析：到这一步，基本上就可以服务化了。
       
       例如，随着业务的发展一些公用的配置表、字典表等越来越多，这时可以将这些表拆到单独的库中，甚至可以服务化。再有，随着业务的发展孵化出了一套业务模式，这时可以将相关的表拆到单独的库中，甚至可以服务化。
    4. 垂直分表     
       1. 概念：以字段为依据，按照字段的活跃性，将表中字段拆到不同的表（主表和扩展表）中。
       
       2. 结果：
       
          每个表的结构都不一样；
          每个表的数据也不一样，一般来说，每个表的字段至少有一列交集，一般是主键，用于关联数据；
          所有表的并集是全量数据；
       
       3. 场景：
       
          系统绝对并发量并没有上来，表的记录并不多，但是字段多，并且热点数据和非热点数据在一起，单行数据所需的存储空间较大。以至于数据库缓存的数据行减少，查询时会去读磁盘数据产生大量的随机读IO，产生IO瓶颈。
       
       4. 分析：
       
          可以用列表页和详情页来帮助理解。垂直分表的拆分原则是将热点数据（可能会冗余经常一起查询的数据）放在一起作为主表，非热点数据放在一起作为扩展表。
       
          这样更多的热点数据就能被缓存下来，进而减少了随机读IO。拆了之后，要想获得全部数据就需要关联两个表来取数据。
       
          但记住，千万别用join，因为join不仅会增加CPU负担并且会讲两个表耦合在一起（必须在一个数据库实例上）。关联数据，应该在业务Service层做文章，分别获取主表和扩展表数据然后用关联字段关联得到全部数据。
    2. 分库分表步骤
       
       根据容量（当前容量和增长量）评估分库或分表个数 -> 选key（均匀）-> 分表规则（hash或range等）-> 执行（一般双写）-> 扩容问题（尽量减少数据的移动）。
    
    3. 分库分表问题
       1. 非partition key的查询问题（水平分库分表，拆分策略为常用的hash法）
       2. 非partition key跨库跨表分页查询问题
       3. 扩容问题（水平分库分表，拆分策略为常用的hash法）
          1. 水平扩容表（双写迁移法）   
             第一步：（同步双写）应用配置双写，部署；
             
             第二步：（同步双写）将老库中的老数据复制到新库中；
             
             第三步：（同步双写）以老库为准校对新库中的老数据；
             
             第四步：（同步双写）应用去掉双写，部署；