---
layout:     post
title:      华为交换机跨VLAN通信实验
subtitle:   单播加组播
date:       2025-03-26
author:     zcheng19
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Management
---

## 场景一：单播跨VLAN通信场景
实验拓扑图如下所示，拓扑中包含两个Vlan，分别是Vlan 10和Vlan 20，PC1、PC2和PC3直连各自交换机的链路类型为access，交换机之间的链路类型为trunk，trunk链路允许所有Vlan标签的数据通过。拓扑中，Vlan 10的PC1需要向对端Vlan 10的PC3和Vlan 20的PC2通信，由于二层不能直接跨Vlan通信，因此需要打开华为三层交换机的Vlanif逻辑接口。Vlan 10的交换机逻辑接口对应Vlanif 10，作为Vlan 10内所有主机的默认网关，IP地址为192.168.10.254/24。由于PC1的数据可以直接发送到交换机SW2，因此在SW2上设置Vlanif 10，同时作为PC1和PC3的默认网关，为了实现跨Vlan转发功能，在交换机SW2上设置Vlanif 20逻辑接口，IP地址为192.168.20.254/24，使其作为PC2的默认网关。由于在同一台交换机上各个Vlanif之间可以相互转发数据，因此可以实现在三层交换机上的跨Vlan数据转发。

![](/blog_img/RSU.png)

图1  单播跨VLAN通信场景

交换机配置命令：

**SW1：**

System-view

Vlan batch 10 20

Interface g0/0/1

Port link-type access

Port default vlan 10

Quit

Interface g0/0/2

Port link-type trunk

Port trunk allow-pass vlan all

Quit

**SW2:**

System-view

Vlan batch 10 20

Interface g0/0/1

Port link-type trunk

Port trunk allow-pass vlan all

Quit

Interface g0/0/2

Port link-type access

Port default vlan 10

Quit

Interface g0/0/3

Port link-type access

Port default vlan 20

Quit

Interface vlanif 10

Ip address 192.168.10.254 24

Quit

Interface vlanif 20

Ip address 192.168.20.254 24

Quit

测试结果如下，PC1、PC2、PC3之间可以相互ping通：

![](/blog_img/RSU.png)

图2 PC1 ping PC2、PC3

![](/blog_img/RSU.png)

图3 PC2 ping PC1、PC3

![](/blog_img/RSU.png)

图4 PC3 ping PC1、PC2
## 场景二：二层组播通信场景

图5 二层组播通信场景拓扑

实验拓扑如上图所示，组播服务器和组播组用户都在同一Vlan 10中，在同一Vlan中发送组播流属于二层组播，当没有使能华为交换机的igmp-snooping功能时，交换机会将组播数据流以广播的形式转发到每个组播组用户。如图PC1宣告加入225.1.1.1，PC2宣告加入239.50.1.1，而组播源服务器只想为225.1.1.1的组播组用户推流，交换机没有配置igmp-snooping功能，端侧配置如下图所示。

![](/blog_img/RSU.png)

图5 组播源发送UDP组播数据包地址及端口号

![](/blog_img/RSU.png)

图6 PC1加组播组

![](/blog_img/RSU.png)

图7 PC2加组播组

组播源向接收端发送组播数据后，通过在接收端抓包，发现PC1和PC2均能接收到发往225.1.1.1组播组的数据包，如下图所示，不符合组播的需求。

![](/blog_img/RSU.png)

图8 未使能igmp-snooping的PC1抓包情况

![](/blog_img/RSU.png)

图9 未使能igmp-snooping的PC2抓包情况

为了避免在PC1和PC2上都收到发往PC1的组播数据，需要在直连组播组用户侧的交换机SW2上开启igmp-snooping功能（华为仿真器不支持此功能，华为实体交换机支持此功能）。此时抓包情况如下图所示，开启igmp-snooping后只能在PC1上收到组播数据，PC2收不到发往PC1的组播数据，功能正常。

![](/blog_img/RSU.png)

图10 开启igmp-snooping后PC1抓包情况

![](/blog_img/RSU.png)

图11 开启igmp-snooping后PC2抓包情况

交换机配置命令：

**SW1：**

System-view

Vlan 10 

Quit

Interface g0/0/1

Port link-type access

Port default vlan 10

Quit

Interface g0/0/2

Port link-type trunk

Port trunk allow-pass vlan all

Quit

**SW2:**

System-view

Vlan 10

Quit

Interface g0/0/1

Port link-type trunk

Port trunk allow-pass vlan all

Quit

Interface g0/0/2

Port link-type access

Port default vlan 10

Quit

Interface g0/0/3

Port link-type access

Port default vlan 10

Quit

Igmp-snooping enable   全局使能

Vlan 10

Igmp-snooping enable   

Igmp-snooping version 3   此命令为必要配置项，交换机需要具有最高版本igmp协议
## 场景三：跨Vlan组播通信场景

![](/blog_img/RSU.png)
图12 跨Vlan组播场景

实验拓扑图如上所示，位于Vlan 10内的组播需要给Vlan 20内的用户发送组播数据流，纯二层通信是无法跨越Vlan的，因此还需要结合交换机的三层转发功能。组播连通的前提是单播能够连通，因此需要先通过设置交换机SW2的Vlanif 10和Vlanif 20使Vlan 10和Vlan 20内的终端能够互相ping通，如下图所示。

在此基础上，需要打开交换机SW2的全局组播路由功能，并在每个Vlanif内使能pim协议，目的是帮助组播流找到上游接口的Vlanif（本场景是Vlanif 10）和下游接口的Vlanif（本场景是Vlanif 20），为了查询各Vlanif下接入了哪个组播组的用户，需要在直连组播组用户的交换机上开启igmp协议，如果检测到Vlanif 20下接入了225.1.1.1的组播组用户，那么发往该组播地址的组播源就会向Vlan 20发送组播数据，发往其他组播地址的流量将不会到达Vlan 20。例如，PC1加入225.1.1.1组播组，PC2加入239.50.1.1组播组，此时根据igmp协议，交换机会认为Vlanif 20接口下共有两个组播组的用户，当Vlan 20没有使能igmp-snooping功能时，组播源发往225.1.1.1和239.50.1.1的组播数据都会转发到Vlan 20内并在该域内广播，即PC1和PC2都能收到这两个组的组播流。抓包情况如下图所示。

![](/blog_img/RSU.png)

图13 未使能igmp-snooping的PC1抓包情况

![](/blog_img/RSU.png)

图14 未使能igmp-snooping的PC2抓包情况

当Vlan 20使能igmp-snooping功能时，PC1和PC2只能收到各自加入组的组播流。以组播源发往225.1.1.1为例，抓包情况如下图所示。PC1能够收到其所对应的组播组信息，而另一个组播组中的PC2用户收不到组播源发往PC1组播组的数据流，功能正常。

![](/blog_img/RSU.png)

图15 开启igmp-snooping后PC1抓包情况

![](/blog_img/RSU.png)

图16 开启igmp-snooping后PC2抓包情况

在组播过程中需要注意，将组播数据流IPv4协议格式里的TTL值设置不能小于1，否则数据包将被第一跳交换机直接丢弃，无法转发。

交换机配置命令：

**SW1：**

System-view

Vlan batch 10 20

Interface g0/0/1

Port link-type access

Port default vlan 10

Quit

Interface g0/0/2

Port link-type trunk

Port trunk allow-pass vlan all

Quit

**SW2:**
System-view

Vlan batch 10 20

Interface g0/0/1

Port link-type trunk

Port trunk allow-pass vlan all

Quit

Interface g0/0/2

Port link-type access

Port default vlan 20

Quit

Interface g0/0/3

Port link-type access

Port default vlan 20

Quit

Interface vlanif 10

Ip address 192.168.10.254 24

Quit

Interface vlanif 20

Ip address 192.168.20.254 24

Quit

Multicast routing-enable    全局使能

Interface vlanif 10

Pim dm

Quit

Interface vlanif 20

Pim dm

Igmp enable

Quit

Igmp-snooping enable   全局使能

Vlan 20

Igmp-snooping enable

Igmp-snooping version 3    必写命令

Quit
## 场景四：二三层组播混跑场景
![](/blog_img/RSU.png)

图17 二三层组播混跑场景

该场景中组播组用户PC1和PC2分别在Vlan 10和 Vlan 20内，都加入225.1.1.1组播组，组播源在Vlan 10内，因此PC1与组播源之间的通信属于二层通信，而PC2与组播源之间的通信属于三层通信，如果只开启Vlanif间的组播路由功能，那么组播路由表会识别Vlanif 10为上游接口，Vlanif 20为下游接口，当Vlan 10内的用户需要接收组播源的数据时，组播路由表并不会把Vlanif 10同时当作下游接口转发到PC1，因此PC1收不到组播数据。需要进一步开启vlan 10内的igmp-snooping才能实现在同一Vlan下的二层组播。实验结果如下图所示，在开启pim组播路由且未开启Vlan 10的igmp-snooping功能时，PC1抓不到组播数据包，PC2可以抓到组播数据包。

![](/blog_img/RSU.png)

图18 Vlan 10未使能igmp-snooping的PC1抓包情况

![](/blog_img/RSU.png)

图19 Vlan 10未使能igmp-snooping的PC2抓包情况

当同时开启组播路由和igmp-snooping功能后，PC1和PC2都可以抓到组播数据包。

![](/blog_img/RSU.png)

图20 Vlan 10开启igmp-snooping后PC1抓包情况

![](/blog_img/RSU.png)

图21 Vlan 10开启igmp-snooping后PC2抓包情况

交换机配置命令：

**SW1：**

System-view

Vlan batch 10 20

Interface g0/0/1

Port link-type access

Port default vlan 10

Quit

Interface g0/0/2

Port link-type trunk

Port trunk allow-pass vlan all

Quit

**SW2:**

System-view

Vlan batch 10 20

Interface g0/0/1

Port link-type trunk

Port trunk allow-pass vlan all

Quit

Interface g0/0/2

Port link-type access

Port default vlan 10

Quit

Interface g0/0/3

Port link-type access

Port default vlan 20

Quit

Interface vlanif 10

Ip address 192.168.10.254 24

Quit

Interface vlanif 20

Ip address 192.168.20.254 24

Quit

Multicast routing-enable    全局使能

Interface vlanif 10

Pim dm

Quit

Interface vlanif 20

Pim dm

Igmp enable   直连组播组用户的下游接口需要使能igmp协议

Quit

Igmp-snooping enable   全局使能

Vlan 10

Igmp-snooping enable   二层使能Vlan 10接收组播源数据

Igmp-snooping version 3    

Quit

Vlan 20

Igmp-snooping enable   避免在Vlan 20广播组播数据

Igmp-snooping version 3   

Quit
