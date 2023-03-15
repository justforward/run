
ES的底层设计理念：分布式搜索引擎

1）index ---具有一个类别的表
2）type---表
3）mapping ---字段定义
4）document ---行
5）field--字段



ES 中存储数据的**基本单位是索引，**


ES的存储单位是index，index可以理解为一个mysql的表。


ES存储的基本单元是shard（分片），每个segment实际上是一些倒排索引的集合，

每次创建一个新的Document， 创建一个新的Segment， 而不会去修改原来的Segment； 且每次的文档删除操作，会仅仅标记Segment中该文档为删除状态， 而不会真正的立马物理删除。当文件过多时 es 会自动进行 segment merge（合并文件），合并时会同时将已经标注删除的文档物理删除。

每次创建一个新的document，创建一个新的segment，而不是去修改原来的，且每次的文档删除操作，会仅仅标记为segment种改文档为删除状态，而不是真正的立马删除，

倒排索引

在搜索引擎中，每个文档都有一个对应的文档 ID，**文档内容被表示为一系列关键词的集合**。例如，文档 1 经过分词，提取了 20 个关键词，每个**关键词都会记录它在文档中出现的次数和出现位置**。

那么，倒排索引就是**关键词到文档** ID 的映射，每个关键词都对应着一系列的文件，这些文件中都出现了关键词。


segment 存储文档的基本单元

![[Pasted image 20230315192726.png]]


index(documents)

shard 

index

segment


**写的整个流程：**

数据先写入内存 buffer，然后每隔 1s，将数据 refresh 到 os cache，到了 os cache 数据就能被搜索到（所以我们才说 es 从写入到能被搜索到，中间有 1s 的延迟）。每隔 5s，将数据写入 translog 文件（这样如果机器宕机，内存数据全没，最多会有 5s 的数据丢失），translog 大到一定程度，或者默认每隔 30mins，会触发 commit 操作，将缓冲区的数据都 flush 到 segment file 磁盘文件中。

**es 丢数据的问题：**

1）其实 es 第一是准实时的，数据写入 1 秒后可以搜索到；可能会丢失数据的。

2）有 5 秒的数据，停留在 buffer、translog os cache、segment file os cache 中，而不在磁盘上，此时如果宕机，会导致 5 秒的**数据丢失**


## 2.2 删除和更新操作

如果是删除操作，commit 的时候会生成一个 .del 文件，里面将某个 doc 标识为 deleted 状态，那么搜索的时候根据 .del 文件就知道这个 doc 是否被删除了。

如果是更新操作，就是将原来的 doc 标识为 deleted 状态，然后新写入一条数据。

buffer 每 refresh 一次，就会产生一个 segment file ，所以默认情况下是 1 秒钟一个 segment file ，这样下来 segment file 会越来越多，此时会定期执行 merge。每次 merge 的时候，会将多个 segment file 合并成一个，同时这里会将标识为 deleted 的 doc 给**物理删除掉**，然后将新的 segment file 写入磁盘，这里会写一个 commit point ，标识所有新的 segment file ，然后打开 segment file 供搜索使用，同时删除旧的 segment file 。


