### 1、当大量数据在写入Redis时，，但通过grafana监控发现Memory Usage（内存使用量） 没有变化？
#### 排除故障1：

先确认数据库密码 -a xxx是否正确的  ，若密码填错，测试数据不会真实的写入数据库，所以监控的内存使用量不会有变化。

#### 排除故障2：

登录数据库客户端，执行monitor查看Redis，发现数据一直有写入操作，所以数据是有真实写入，没有问题。

注：grafana监控的内存使用量没有变化，是由于性能测试脚本中-r参数值有限，使数据一直在写入覆盖，监控的内存没有变化，可以增大-r的取值，多次验证。


![image](https://github.com/chenyiyi1/test1/blob/fd386a3e424cdb0e86855a75c34c03ed9078bcff/picture/5d70ed19160a6e2990ad68bd811d4a5.png)

### 2、通过grafana发现，Redis各节点的内存使用量不一致,要怎么处理？
#### 解决方法：
首先判断CPU利用率最高的是主节点，检查其余从节点监控，内存使用量若与主节点保持不一致，登录此从节点客户端，查询主从关系，若主从关系与实际不符，再重新配置主从关系？
#### 获取主从信息
```
config get replicaof
```
![image](https://github.com/chenyiyi1/test1/blob/master/picture/12d3e614c89a2dded5f453b846c7ce1.png)
### 修改主从信息,使其生效
```
SLAVEOF 10.2.1.21 6379
config rewrite
```
## 系统问题：
### 1、执行命令后，导致系统卡顿，甚至断开主从连接？
在Redis存储了大量数据时，一些命令可能导致系统阻塞，尽量不要执行下列命令。
config、flushall、flushdb、shutdown、eval。


### 2、Redis的cpu占比过高？
排查一：
连接数过多，关闭僵尸连接，采用redi-cli登录,采用client kill ip:port(redis远程连接的ip和端口)。修改redis timeout参数。
排查二：
慢查询堆积，因为redis是单线程，如果有慢查询的话，会阻塞住之后的操作，通过redis日志查，可以对慢查询进行持久化，比如定时存放到mysql之类。

### 3、当内存实际使用量近乎总内存使用量的一半？
建议增大Redis可用内存，以免影响缓存速率。

### 4、Redis客户端连接过多不释放？
查看redis连接信息：
/opt/redis/src/redis-cli -c -h x.x.x.x -p xxxx info clients
设置空闲清理时间：
redis-cli config set timeout 300

### 5、Redis的持久化方式?
开启了RDB内存快照和AOF日志文件。
### 6、线上尽量禁止使用 monitor 命令？
它实时打印出 Redis 服务器接收到的命令，调试可以用。在高并发条件下，会存在内存暴增和影响 Redis 性能的隐患。
### 7、
