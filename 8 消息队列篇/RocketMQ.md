



https://cloud.tencent.com/developer/article/1581368



DDMQ，底层消息中间件的基础上加了一层代理，独立部署延迟服务模块，使用rocksdb进行临时存储。rocksdb是一个高性能的KV存储，并支持排序。


![[Pasted image 20230327200123.png]]