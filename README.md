#### 1、Redis-benchmark脚本一直在跑，但通过grafana监控发现Memory Usage（内存使用量） 没有变化？
### 排除故障1：

先确认测试脚本参数中的数据库密码 -a xxx是否正确的  ，若密码填错，测试数据不会真实的写入数据库，所以监控的内存使用量不会有变化。

### 排除故障2：

登录数据库客户端，执行monitor查看Redis，发现数据一直有写入操作，所以数据是有真实写入，没有问题。

注：grafana监控的内存使用量没有变化，是由于性能测试脚本中-r参数值有限，使数据一直在写入覆盖，监控的内存没有变化，可以增大-r的取值，多次验证。


![image](https://github.com/chenyiyi1/test1/blob/fd386a3e424cdb0e86855a75c34c03ed9078bcff/picture/5d70ed19160a6e2990ad68bd811d4a5.png)
#### 2、通过grafana发现，Redis各节点的内存使用量不一致,要怎么处理？
### 解决方法：
首先判断CPU利用率最高的是主节点，检查其余从节点监控，内存使用量若与主节点保持不一致，登录此从节点客户端，查询主从关系，若主从关系与实际不符，再重新配置主从关系？
### 获取主从信息
```
config get replicaof
```
![image](https://github.com/chenyiyi1/test1/blob/master/picture/12d3e614c89a2dded5f453b846c7ce1.png)
### 修改主从信息,使其生效
```
SLAVEOF 10.2.1.21 6379
config rewrite
```
