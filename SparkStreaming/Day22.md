## Spark提升

集群搭建:

磁盘的容量估算集群的规模,然后来进行选择软件和硬件.

高速磁盘多一点. 可以加SSD

系统盘要做镜像!

对于虚拟化/云计算的态度: 生产环境尽量不用.

### RPC原理及其他

ssh: 系统之间的.

RPC: 进程之间的.

只要是分布式, 底层一定有RPC.

发邮件的方式…..发消息

netty: java语言的

---

如果有2个网卡,不知道哪个是master?

进入spark-shell

sc.getConf.getAll —查所有的信息

---

spark-shell - -master local —>这里local不可以写成ip地址!! 源码里面写死了

