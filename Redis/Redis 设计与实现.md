# Redis 设计与实现



### SDS 的定义

在 Redis 当中不存在 C 字符串 [以空字符串结尾的字符数组] 这种存储的方式 ，而 C 字符串常常被用作为字符串字面量使用比如 Redis 当中的打印日志部分。

~~~ c
LOG("log_str", "No arguments follow this format string.");
~~~

对于字符串的存储 Redis 内部使用了一种数据结构 `SDS` 来实现具体的存储，举个例子。比如，在客户端有这样的一条指令。

~~~shell
redis > SET msg "Hello World"
~~~

而对于这里的 键 msg 与 值 Hello World 都分别由 SDS 来承载，同样的对于命令

~~~shell
redis > RPUSH fruits "apple" "banan" "cherry"
~~~

也是分别有 SDS 来进行存储，第一个 SDS 存储 fruits 键 ，第二个 SDS 存储 apple 值，第三个 SDS 存储 banan 以此类推。

SDS 结构除了用于存储数据库当中的字符串值，而且还被用作缓存区使用。



~~~ c
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used 已使用字节的数量 */ 
    uint64_t alloc; /* excluding the header and null terminator*/
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[]; //用于保存字符串
};
~~~

