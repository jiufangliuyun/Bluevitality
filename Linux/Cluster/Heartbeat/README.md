* haresources 资源相关
* ha.cf 心跳设置
* authkeys 认证

在备份节点上也需安装Heartbeat，安装方式与在主节点安装过程一样  
依次安装libnet和heartbeat源码包，安装完毕后在备份节点使用scp命令把主节点配置文件传输到备份节点  

[root@node2 ~]#`scp -r node1:/etc/ha.d/*  /etc/ha.d/`  
[root@node1 ~]#`/etc/init.d/heartbeat`  
Usage: /etc/init.d/heartbeat {start|stop|status|restart|reload|force-reload}   

#### 设置主备份节点时间同步
```
在双机高可用集群中，主节点和备份节点的系统时间也非常重要，因为节点之间的监控都是通过设定时间来实现的。
主备节点之间的系统时间相差在10秒以内是正常的，如果节点之间时间相差太大，就有可能造成HA环境的故障。
解决时间同步的办法有两个：
  一个办法是找一个时间服务器，两个节点通过 ntpdate命令定时与时间服务器进行时间校准；
  另一个办法是让集群中的主节点作为ntp时间服务器，让备份节点定时去主节点进行时间校验。
```
