# key
## 名称需要有意义
业务名称:功能名称:其他
比如电商秒杀 mall:seckill:20180612

## 不能包含特殊字符
比如:空格、换行、单双引号以及其他转义字符。比如: xxxx:2017-11-10 13:20:00 +0000 UTC

## 合理设置过期时间
  随着积累增多，无效数据内存消耗增大，又不敢轻易删除

## 避免使用大key
  单key限制在20KB
  hash,list,zset,set元素个数限制在5万左右
> key过大会浪费网卡流量，影响redis性能，无法accept新的连接

## 选择合适的数据结构
背景:存储每隔用户发放的feed数量
### 实现方法：
- 使用string
  userId为key, weiboCount作为Value
- 使用hash
  hashkey只有一个, key="allUseCount", field=userId, fieldValue= count
- 对单hash进行拆分
  hashkey为多个, key=userId/100, field=userId%100, fieldValue= count

```
方法一 Memory  
used_memory:85999592  
used_memory_human:82.02M  
used_memory_rss:96043008  
used_memory_peak:85999592  
used_memory_peak_human:82.02M  
used_memory_lua:36864  
mem_fragmentation_ratio:1.12  
mem_allocator:jemalloc-3.6.0  
  
方法二 Memory  
used_memory:101665632  
used_memory_human:96.96M  
used_memory_rss:110702592  
used_memory_peak:101665632  
used_memory_peak_human:96.96M  
used_memory_lua:36864  
mem_fragmentation_ratio:1.09  
mem_allocator:jemalloc-3.6.0  
  
方法三 Memory  
used_memory:9574136  
used_memory_human:9.13M  
used_memory_rss:17285120  
used_memory_peak:101665632  
used_memory_peak_human:96.96M  
used_memory_lua:36864  
mem_fragmentation_ratio:1.81  
mem_allocator:jemalloc-3.6.0  
```


# 命令使用
## 慎用 O(N)命令
  keys, hgetall、lrange、smembers、zrange 等命令可以考虑使用对应的scan命令
## 清理命令
  线上不要轻易使用flushall、flushdb
## 事务系列命令
  尽量使用redis脚本替代事务操作。脚本更简单,快速
## 删除bigkey
  redis是同步删除，大key可能会阻塞整个server
  可以遍历部分元素慢慢删除, 新版本已支持异步删除。
  
## 合理使用monitor
功能: 实时接收master执行的所有命令(包括读)。
redis-server为每个client维护了一个input buffer和output buffer, 当访问量很大时会导致命令非常多，output就变的很大，从而额外消耗很多内存。

> 对于特殊的命令可以直接在client拦截或者redis-server执行rename


##总结: redis是基于事件驱动的单线程程序，尽量避免耗时操作阻塞主线程。

# redis分析工具 redis-analyzer
- 功能
  解析rdb, aof, 以及执行monitor, 来查找key和分析各种top-key(big key, hot-key, expiry-key, slowlog-key)
  可以协助排查线上问题，以及大key监控
  ```
    redis-analyzer
  Usage:
    redis-analyzer [command]
  
  Available Commands:
    bigkey      Find the key over the specified size
    dump        Dump rdb file from redis server
    gen-conf    Generate example json config file
    help        Help about any command
    keys        Grep key through the golang regular
    monitor     A query analyzer that parses redis monitor command
    parse-aof   Parses aof(append-only file) file from local or redis server directly
    parse-rdb   Parses rdb file from local or redis server directly
    slowlog     Collect redis server slowlog
  ```