# 自建SNAT网关平滑迁移到NAT网关 {#task_ipr_2rb_zdb .task}

通过使用路由表的最长匹配原则，您可以将搭建在ECS实例的SNAT网关平滑迁移至阿里云NAT网关。

如果您已经在VPC中基于ECS搭建了SNAT网关，又想将架构切换为基于NAT网关实现的SNAT，您可以将原有自建SNAT网关拆除，再进行NAT网关的创建和配置。但该操作会导致SNAT功能中断一段时间。

本教程的迁移方法利用路由表的一些特性（主要是“最长匹配原则”），实现从自建SNAT网关到阿里云NAT网关的无缝切换。切换过程中，不会出现SNAT功能不可用，仅在切换的一瞬间发生已有TCP连接的断开，应用进行重连即可。

本操作中作为示例的VPC和ECS配置如下：

-   VPC中有两个ECS实例：

    -   i-9410jxxxx配置了自建的SNAT网关。这台ECS上绑定了一个EIP，并且开启了转发服务、配置了iptables规则以实现SNAT转发。
    -   i-94kjwxxxx为需要SNAT功能来访问互联网的服务器。
-   VPC的路由器上，添加了一条自定义路由（目标网段为0.0.0.0/0），将公网访问请求转发给i-9410jxxxx。


1.   在VPC中添加8条路由条目，对原有路由进行覆盖。 

    路由条目的目标网段分别为1.0.0.0/8、2.0.0.0/7、4.0.0.0/6、8.0.0.0/5、16.0.0.0/4、32.0.0.0/3、64.0.0.0/2、128.0.0.0/1，下一跳均为i-9410jxxxx。

    由于路由表按照最长匹配原则，会优先匹配子网掩码最长的路由条目；而去往任意IP地址的数据包，都会匹配到这8条中的一条；因此，0.0.0.0/0这条路由实际上已经不再有用了。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13997/15554147554444_zh-CN.png)

2.   删除目标网段为0.0.0.0/0的路由条目。 
3.   创建NAT网关。 

    创建NAT网关后，系统会自动添加一条0.0.0.0/0的路由，指向NAT网关。

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/13997/15554147554445_zh-CN.png)

4.   绑定弹性公网IP。 

    **说明：** 确保EIP的带宽和自建NAT的带宽一致。因为只要在NAT网关添加了SNAT规则，SNAT规则中的ECS的出公网方向的流量就会受EIP带宽的限速。

5.   配置SNAT规则。 
6.   删除VPC中添加的8条路由路由条目，使路由器把公网访问请求不再转发给自建SNAT，而是转发给NAT网关。 

    至此，已经完成了从自建SNAT网关到使用官方NAT网关的SNAT功能的全部切换流程。


