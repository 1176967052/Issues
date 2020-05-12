#### 数据结构
1. SDS（simple dynamic string，简单动态字符串）
   ```
      /* 保存字符串对象的结构 */
      struct sdshdr {
          int len;       // buf 中已占用空间的长度
          int free;      // buf 中剩余可用空间的长度
          char buf[];    // 数据空间
      };
   ```
   SDS空间预分配：

   对SDS字符串进行扩展，如果free值大于扩展值则直接存储，否则重新分配空间：

   1. 如果对SDS进行修改后字符串长度（len值）小于1MB，则额外分配len大小相同的空间（free值）；
   2. 若SDS修改后len大于1MB，则额外分配1MB的空间（free值）。

   SDS惰性空间释放：

   1. 对SDS字符串进行缩短操作，并不会重新分配内存回收缩短的字节，而是使用free属性将这些字节的数量记录起来。
   2. redis保存的是SDS中的buf的二进制数据。SDS的API都是二进制安全的。
