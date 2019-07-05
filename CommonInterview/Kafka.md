1. 项目中怎么使用消息队列
2. 为什么使用消息队列
   解耦（发送到不确定的某个系统中），削峰（增加系统的处理能力），异步
3. 消息队列优缺点
   优点：解耦，削峰，异步
   缺点：系统可用性降低（mq挂掉，整个系统崩溃了）；
        系统的复杂性增加（消息丢失，消息重复消费，消息顺序乱了，消息消费者出现问题导致消息积压）；
        一致性问题（ABC一同执行，结果AB成功，C失败）
4. kafka,rabbitmq,rocketmq,activemq对比及适合场景
   
   + 单机吞吐量 ：activemq ，rabbitmq 吞吐量在万级，比kafka和rocketmq（10万级别）低一个数量级
   + topic数量对吞吐量的影响：topic数量达到几十上百个rocketmq吞吐量小幅度下降，kafka大幅度下降
   + 时效性：activemq毫秒级，rabbitmq微秒级（最大特点，延时最低），rocketmq毫秒级，kafka毫秒级以内
   + 可用性：activemq ，rabbitmq基于主从架构实现高可用，rocketmq和kafka非常高，是分布式架构。kafka是一个数据多个副本，不会丢失，不会导致不可用
   + 消息可靠性：activemq低概率丢失数据，rocketmq和kafka经过参数配置可以做到0丢失率
   + 核心特点：activemq功能完备，rabbitmq基于erlang开发，并发，性能极好。rocketmq功能完备，分布式，扩展性好。kafka功能简单但是大数据和日志采集的事实标准
   + 总结：activemq官方维护越来越少，rabbitmq基于erlang开发，开源提供的管理界面做的好，源码扩展和定制比较难，rocketmq简单易用，阿里的业务实践保障，kafka唯一缺点是消息可能重复消费
   
5. 如何保证消息队列的高可用
  
   activemq ，rabbitmq基于主从架构实现高可用，rocketmq和kafka非常高，是分布式架构。
   
   基于主从架构的高可用型：例子：rabbitmq
   + 单机模式
   + 普通集群模式：只有一台机器（一个集群节点上）保留queue元数据和实际的数据，其他节点只有queue的元数据
      
     缺点：在集群内部产生大量的数据传输。可用性没保障，有实际数据的机器宕机，所有数据丢了
   + 镜像集群模式：每个节点都存有所有的实际数据，写到任何一个节点，都同步到其他节点去
     
     缺点：不是分布式的，如果queue的数据量很大，大到这个机器行的容量无法容纳了，那么其他节点也保留大量数据
   
   基于分布式的高可用性：例子：kafka
   
   多个broker组成，每个broker是一个节点；创建一个topic，可以划分为多个partition。每个partition存在于不同的broker上，每个partition保留一部分数据
   
   一个topic 可以配置几个partition，produce发送的消息分发到不同的partition中，consumer接受数据的时候是按照group来接受，kafka确保每个partition只能同一个group中的同一个consumer消费，如果想要重复消费，那么需要其他的组来消费。Zookeerper中保存这每个topic下的每个partition在每个group中消费的offset
   
   如果三个partition,一个broker宕机，则1/3数据丢失，所以0.8版本之前没有高可用。
   
   通过replica副本机制，每个partition都有对应的副本。每个partition的多个副本中会选举出一个leader,其余都是follower。
   
   每个partition只有leader对外进行读写交互。leader负责将数据同步到follower上去

   
6. 如何保证消息不被重复消费，如何保证消费的幂等
   
   重复消费的可能：
   
   消费者消费完数据后准备向kafka提交offset,但是还没提交，结果消费者进程重启了，此时已经消费过的数据offset没有提交，kafka并不知道，会重复发送
   
   如何保证重复消费之后的幂等性：
   + 数据库主键(唯一键)插入，已存在则update
   + 写redis,利用set数据类型，天然幂等性
   + 加全局唯一id,操作之前先查询是否消费过 
    
7. 如何保证消息的可靠性传输，消息丢失怎么办
  
   rabbitmq可能存在的消息丢失的环节：
   + 生产者写消息到mq的网络传输过程中丢了，或者是消息到mq中了但是mq内部出错了没有保存数据
     
     解决：
     + rabbitmq支持事务功能，发送数据之前开启事务(channel.txSelect),消息没有被mq收到，会报异常错误，则可以回滚事务(channel.txRollback),然后重试，成功则提交事务(channel.txCommit).但是太耗性能，吞吐量下降
     + 生产者开启confirm模式，每次写的消息分配一个唯一id,写入mq成功会回调生产者的接口通知一个ack消息;如果mq没能处理这个消息，会回调nack接口告诉消息发送失败，可以重试。
   + mq接收到消息后先暂存在内存中，结果消费者还没来得及消费，mq自己挂掉了，导致暂存在内存中的数据丢了
     
     解决：
     + rabbitmq开启持久化，保存数据导磁盘上。但是还没持久化就挂了会导致数据丢失，但是概率极小。
       
       步骤是创建queue的时候将其设置为持久化，可以持久化元数据。其次发送消息的时候将消息的deliveryMode设置为2，才会持久化数据。两个持久化必须同时开启。
       
       持久化可以和生产者confirm机制配合起来。消息持久化到磁盘，才通知生产者ack通知。则持久化之前数据丢失生产者收不到ack，可以重发
   + 消费者消费到了消息，但是没来得及处理，消费者就挂掉了，mq以为消费者已经消费完了
      
      只有一种可能是打开了消费者的autoAck机制。消费到数据后，消费者自动通知mq，通知完成消费。如果收到一条消息，还在处理中，没处理完，消费者自动autoAck。消费者系统宕机，消息丢失，没处理完，mq认为这个消息已经处理掉了
      
      解决：
      
      取消autoAck，消费者处理完后手动提交ack
   
   kafka消息丢失：
   + 消费者端丢失的唯一可能是开启了自动提交offset
     
     解决：
     
     消费者处理完消息后手动提交offset，但是刚处理完没来得及提交offset就挂了，会重新消费一次，自己保证幂等性就好了。
    
   + kafka自己弄丢数据
     
     kafka某个broker（leader）宕机，partition重新选举leader时，此时其他follower刚好有些数据没同步完，然后某个follower被选举为leader，就缺少一部分数据
     
     解决：设置四个个参数
     
     1. 给topic设置replication.factor参数，这个值必须大于1，要求每个partition至少有两个副本
     2. 在kafka服务器端设置min.insync.replicas参数，这个参数必须大于1,要求一个leader至少感知到有至少1个follower在跟自己保持联系，没掉队，保证leader挂了之后还有一个follower
     3. 在生产者端设置acks=all，要求每条数据，必须写入所有replica之后，才认为写成功了
     4. 在生产者端设置retries=MAX(很大的值),要求一旦写入失败，就无限重试，卡在这儿
     
     这样可以保证borker故障了，选举leader时数据补丢失
     
   + 生产者不会丢数据
      
     设置了acks=all一定不会丢，因为要求所有的leader接收到消息，所有follower同步到消息才算成功，如果没满足，生产者会自动不断的重试 
   
8. 如何保证消息的顺序性

   顺序错乱场景：
   
   rabbitmq: 一个queue，多个consumer
   
   解决：
   
   拆分成多个queue，每个queue一个consumer；或者就一个queue，但是对应一个consumer，consumer内部用内存队列做排序
   
   kafka：一个topic，一个partition，一个consumer，内部多线程
   
   解决：
   
   kafka一个partition中的数据一定是有序的。例如：生产者可以把订单最为key，同一个订单相关的数据写入到同一个partition中去。每个partition一个消费者，从同一个partition中取出来的数据一定是有序的
   消费者接收时，同一个订单的消息进入同一个partition中，多线程对应多个内存队列，同一个订单的消息被同一个内存队列接收，则是有序的
   
9. 消息队列的延时以及过期失效问题，消息队列满了怎么处理，消息积压怎么解决
   
   消费者挂了，导致消息不被消费，在磁盘积压大量数据
   
   快速处理积压：
   1. 先修复consumer的问题，确保其恢复消费速度，然后将现有consumer都停掉
   2. 新建topic，partition是原来的10倍，临时建立好原来10倍或20倍的queue数量
   3. 写个临时分发consumer程序，部署上去消费积压消息，消费之后不做耗时处理，直接均匀轮询写入10倍的queue
   4. 临时征用10倍的机器来部署consumer，每一批consumer消费一个临时的queue的数据
   5. 消费完成后重新部署原来的消费者
   
   rabbitmq设置过期时间，在queue中积压超过一定时间会被清理掉，导致数据丢失
   
   解决：过了高峰期写临时程序，一点一点查出丢失数据，重新灌入mq
   
   消息积压，mq都快写满了：
   
   解决：只能先快速消费数据，可以直接丢弃，走上一个问题的思路

10. 让你设计消息中间件怎么设计
    
    1. mq可伸缩，快速扩容，可以增加吞吐量和容量。怎么搞？设计分布式系统，参考kafka设计broker,topic,partition，每个partition村一部分数据，资源不够了加partition，可以提高吞吐量
    2. mq数据落地磁盘，顺序写，就没有磁盘随机读写的寻址开销，磁盘顺序读写性能很高的
    3. mq可用性，参考kafka的高可用保障机制。多副本,leader,follower，broker挂了重新选举leader
    4. 数据零丢失，参考kafka的零丢失方案
    