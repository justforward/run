



https://cloud.tencent.com/developer/article/1581368



延迟队列：延迟消息是指生产者发送消息发送消息后，不能立刻被消费者消费，需要等待指定的时间后才可以被消费。


Broker端内置延迟消息处理能力，核心实现思路都是一样：将延迟消息通过一个临时存储进行暂存，到期后才投递到目标Topic中。如下图所示：

![[Pasted image 20230330190252.png]]


步骤说明如下：

1、生产者要将一个延迟消息发送到某个topic中
2、集群（broker）判断这是一个延迟消息后，将其通过临时存储进行暂存。
3、然后通过一个延迟服务（delay service）检查消息是否到期，将到期的消息投递到目标Topic中。
4、消费者消费目标topic中的延迟投递的消息



DDMQ是如何实现延迟队列的？

DDMQ，底层消息中间件的基础上加了一层代理，独立部署延迟服务模块，使用rocksdb进行临时存储。rocksdb是一个高性能的KV存储，并支持排序。


1）RocksDB 支持类似 LevelDB 的 API，允许用户在内存和磁盘之间进行高效的数据存储和检索。
2）RocksDB是一个磁盘数据库
3）仅仅支持键值对形式的数据类型
4）本身不支持分布式 需要与其他工具配合使用


![[Pasted image 20230327200123.png]]


说明如下：

1.  生产者将发送给producer proxy，proxy判断是延迟消息，将其投递到一个缓冲Topic中；
2.  delay service启动消费者，用于从缓冲topic中消费延迟消息，以时间为key，存储到rocksdb中；
3.  delay service判断消息到期后，将其投递到目标Topic中。
4.  消费者消费目标topic中的数据

这种方式的好处是，因为delay service的延迟投递能力是独立于broker实现的，不需要对broker做任何改造，对于任意MQ类型都可以提供支持延迟消息的能力。例如DDMQ对RocketMQ、Kafka都提供了秒级精度的延迟消息投递能力，但是Kafka本身并不支持延迟消息，而RocketMQ虽然支持延迟消息，但不支持秒级精度。

