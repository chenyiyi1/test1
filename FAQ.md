### 1、当QPS超过预期数值，触发告警的处理方法？
正常连接配置下，QPS不会超出预期数值。可能开发人员使用了管道模式（Pipeline pipeline=jedis.pipelined();），使系统每秒处理的请求数量增大，减小了延迟时间，
但是管道模式不适用于请求之间有依赖关系、可靠性要求高的场景，会造成请求失败。请根据业务需求综合考虑是否开启管道模式，可以联系运维人员调整对应的告警值。

### 2、当已使用的内存容量超过预期数值,触发告警的处理方法？
排查一：查看Redis的内存碎片率是否已超过1.5，超过则需要重启Redis服务器，可以让额外产生的内存碎片失效并重新作为新内存来使用，使操作系统恢复高效的内存管理，在重启服务器之前，需要在Redis-cli工具上输入shutdown save命令，意思是强制让Redis数据库执行保存操作并关闭Redis服务，这样做能保证在执行Redis关闭时不丢失任何数据。 在重启后，Redis会从硬盘上加载持久化的文件，以确保数据集持续可用。

排查二：内存碎片率在1.0-1.5之间属于正常范围,内存在正常运行，建议增大集群的内存容量，提高集群的性能，否则Redis的处理请求效率会降低。

### 3、当Redis客户端的总连接数超过预期数值，触发告警的处理方法？
此时的Redis性能较低，CPU占用率很高，检查Redis客户端是否都是有效连接，（./redis-cli –h host –p port client list），由于客户端频繁的连服务器，每次连接都在很短的时间内结束，导致网络丢包。为了解决这两个问题，需要做的就是服务端和客户端定期检查，客户端通过setTestWhileIdle(Boolean.True)、setTimeBetweenEvictionRunsMillis(xxx) 来定期检查方式死链；服务端通过设置超时时间来做到检查连接的问题。
在客户端查看， netstat -ae |grep redis
发现系统存在大量TIME_WAIT状态的连接，通过调整内核参数解决，
vi /etc/sysctl.conf
编辑文件，加入以下内容：
net.ipv4.tcp_timestamps=1 
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 30
 
然后执行 /sbin/sysctl -p 让参数生效。

### 4、当一分钟内每秒的出入流量平均值超过预期数值，触发告警的处理方法？
可能是有某些较大的key一直在频繁访问导致。未防止阻塞进程，建议不在Redis客户端执行查询操作，而是将备份文件导出，使用第三方工具redis-rdb-tools对Redis备份文件dump.rdb进行分析，找出最大key，与开发人员确认具体情况，是否设置过期时间或者删除key。


