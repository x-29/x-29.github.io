---
ID: 73
title: 自顶而下说 Elasticsearch
published: true
date: 2019-03-23 22:05:28
---
<!-- wp:paragraph -->
<p>试着用自顶而下的方式说道一下 Elasticsearch 的运行机制。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>高性能和高可用</strong></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>分布式系统的可用性和性能很重要，Elasticsearch 以分片和缓存保证系统的性能，Index 划分成多个 Shards，这些 Shards 分布在不同的 Node 上。以多副本的方式保证系统的可用性，每个 shard 会有一个或多个副本。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>写操作</strong></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>在一个集群环境下，收到写操作请求时，协调 Node 将请求发送到相应的 Node，Node 依据 Index 有哪些 shard 计算出新数据要写入的 shard，待主 shard 写完后，会通知副 shard 写入新的数据。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>新数据写入时，先写到内存，并在 transaction log 中记录写操作日志。这时的数据还不能搜索到，不过 Elasticsearch 会在 1s 后 refresh 数据到 page cache，此时，就能搜索到了。Elasticsearch 查询效率高，很大部分原因是数据在 page cache 中。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>数据在 page cache 中是不安全的，Elasticsearch 会每隔 30 分钟写入一次数据到磁盘，同时删除掉 transaction log。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>读操作</strong></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>读操作分为查询和获取两个阶段，查询阶段得到 document id，获取阶段根据 document id 取得真正的 document。当读请求到来时，接收到请求的 Node 变成协调 Node，它首先将请求广播到每个 Node 的 Shard 上，由 shard 处理请求并返回结果。然后，协调 Node 聚合 shard 返回的结果，确定实际需要返回的 document 之后，向含有该 document 的 Shard 发起请求，获取真正的数据。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>倒排索引</strong></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>创建好 document 后，Elasticsearch 为 document 的每个 field 建立一个倒排索引。建立倒排索引时，会创建 Posting List 、Dictionary 和 Term Index ，其中前两个存储在磁盘上，后面的 Term Index 放在内存中。搜索时的方向为 :</p>
<!-- /wp:paragraph -->

<!-- wp:preformatted -->
<pre class="wp-block-preformatted">Term Index -&gt; Term Dictionary -&gt; Posting List</pre>
<!-- /wp:preformatted -->

<!-- wp:paragraph -->
<p>先去 term index 中定位到 term dictionary 的某个 offset，然后从这个位置往后顺序查找，找到 term 对应的 posting list ， posting list 里面有document 编号。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>基本概念</strong></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Index，只是一个命名空间，指向一个或多个实际的物理分片(shard)。</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Shards，一个分片是一个底层的工作单元，它仅保存全部数据中的一部分，它是一个Lucence 实例 </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Document ，最基础的可被索引的数据单元</p>
<!-- /wp:paragraph -->
