# 数据持久化

  Redis中所有数据都是放在内存当中的，如果突然宕机，那么数据就会丢失。这时我们就需要将数据持久化。

  两种数据持久化的方法：

  - 快照：一次全量备份，它是内存数据的二进制序列化形式，在存储上十分的紧凑。
  - AOF日志：连续增量备份，它是内存数据修改的记录文本，AOF日志在长期运行过程中会变得无比庞大，所以会定时的去重写AOF，从而加快数据库重启的速度。

### 快照原理

  首先Redis是单线程的。

  在服务线上请求的同时，Redis还需要进行内存快照，内存快照要求Redis必须进行文件IO操作，可文件IO操作是不能使用多路复用API。

  这时就需要使用操作系统多进程COW(Copy On Write)机制实现数据持久化。

  Redis会fork出一个子进程，父进程继续处理客户端的请求，而子进程则去完成内存快照的处理。子进程与父进程共享代码段和数据段。

  子进程只需要遍历现有的数据结构，然后序列化写在磁盘上。但是父进程必须持续服务客户端的请求，然后对内存数据结构不断的更改。

  这时就得COW机制将数据段页面分离。数据段是由很多操作系统的页面组成的（每个页面4K）：当父进程修改数据时，会复制页面分离出去进行修改，所以子进程共享的页面在子进程产生的一瞬间就凝固了，这样子进程就可以安心的进行的数据的持久化了。

### AOF原理

  AOF日志存储的是Redis服务器的顺序指令序列，AOF日志只记录对内存修改的指令记录。

  Redis先执行指令再存储到AOF日志中(这一点不同于其它存储引擎)，这样我们通过AOF可以实现“重放”的功能来恢复Redis当前实例的内存数据结构。

  对于AOF日志我们需要定期的去瘦身。

### Redis4.0混合持久化