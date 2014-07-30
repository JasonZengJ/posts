title: memcache和redis本质区别在哪里
date: 2014-07-29 00:54:41
tags: [Memcache,Redis]
---
1.Redis中，并不是所有的数据都一直存储在内存中的，这是和Memcached相比一个最大的区别。

2.Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，hash等数据结构的存储。

3.Redis支持数据的备份，即master-slave模式的数据备份。

4.Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。

最本质的不同是Redis在很多方面具备数据库的特征，或者说就是一个数据库系统，而Memcached只是简单的K/V缓存 