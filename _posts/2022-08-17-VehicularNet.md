---
layout:     post
title:      车联网调度文献总结
subtitle:   针对车联网中的不同通信技术存在的调度问题进行总结
date:       2022-08-17
author:     zcheng19
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Vehicular Network
---

## 车联网调度技术总结

### 参考文献
Noor-A-Rahim, Md, et al. "A survey on resource allocation in vehicular networks." IEEE transactions on intelligent transportation systems (2020).
|技术或标准|问题分类|待解决问题|解决方法或算法|性能评估|参考文献|
|:---:|:---:|:---:|:---:|:---:|:---:|
|dedicated short range communications (DSRC)|MAC层参数分配问题|不同驻留时间导致接入几率不均等|所有车辆分配相同的MAC参数|用户分配到的资源不公平|-|
|DSRC|MAC层参数分配问题|不同驻留时间导致接入几率不均等|根据驻留时间动态适配MAC参数(根据平均速度选择最小的竞争窗口)|不同驻留时间的车辆数据传输率几乎一致|[45]|
|DSRC|MAC层参数分配问题|吞吐量|提出随机模型|吞吐量提高|[47]|
|DSRC|MAC层参数分配问题|时延、数据率|根据车辆密度寻找最大的竞争窗口|传输时延降低，数据接收率提高|[48]|
|DSRC|MAC层参数分配问题|吞吐量、数据率|两种动态竞争窗口分配机制|数据传输速率和吞吐量提高|[49]|
|DSRC|信道分配问题|吞吐量、数据率|两种动态竞争窗口分配机制|数据传输速率和吞吐量提高|[49]|

