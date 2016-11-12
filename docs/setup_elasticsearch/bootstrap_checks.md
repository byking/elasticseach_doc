# bootstrap配置

elasticsearch会自己做一些关键参数配置校验，有问题的时候不会启动成功。

## heap size check 堆大小检查

JVM在堆大小不够的时候会重新申请堆，可以在启动的时候申请最大的堆大小来避免重新申请。
另外，如果开启了`bootstrap.memory_lock`, JVM会锁住启动时申请的堆内存，如果刚开始申请的
堆内存不够如果重新申请了堆内存需要重新配置下heap size，这样新申请的内存才会也加锁。

## file descriptor check 文件描述符检查

配置了`file descriptors`会对 file descriptor进行检查。

## memory lock check 内存锁检查

JVM的内存有可能会被交换的磁盘，这样的会增加处理时间。可以通过`bootstrap.memory_lock`来锁住内存不被
交换到磁盘。但是启动elasticsearch的用户可能没有`memlock unlimited`，可以通过配置mlockall来校验是否
相关权限。

## maximum number of thread check

elasticsearch会将请求处理的过程分成很多阶段，通过不同的线程池中线程来执行。所以需要能够开启很多线程。
系统需要配置最大线程数为2048，配置`/etc/security/limits.conf`中的`nproc`。

## maximum size virtual memory check

elasticsearch和lucene通过`mmap`来提高性能，因此需要配置`/etc/security/limits.conf`中的`as`为`unlimited`。

## maximum map count check

由于elasticsearch用到了mmap，因此elasticsearch需要配置`vm.max_map_count`至少是262144。

## client jvm check

jvm有两种模式：client jvm和server jvm，前者轻量级，后者重量级在性能上会好。默认都是server jvm。

## serial collector 配置检查

JVM有多重垃圾回收方式，serial collector适合一个核或者堆内存很小的场景，默认不会启用serial collector模式，在
启动的时候需要检察下：`-XX:+UseSerialGC`这样的配置项。默认是elasticsearch使用CMS collector。

## OnError and OnOutOfMemoryError检查


