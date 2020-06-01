---
姓名：坏坏
学习时间：2020年5月28日
整理时间：2020年5月28日
---

# 复习

**IGP协议**（内部网关协议）
- RIP（已淘汰）
- OSPF
- IS-IS

**EGP协议**（外部网关协议）
- BGP

> - IGP用来计算和发现路由，BGP路由控制和路由优选（13条选路原则）
> - IGP是在自治系统（AS）内部使用，EGP是在自治系统间使用

# BGP基本概述

- BGP可以跨越多跳路由建立邻居关系
- 通过单播发送报文
- 基于TCP，端口号179，为应用层协议

# BGP邻居关系建立与配置

- BGP基于TCP，所以建立邻居前会先建立三次握手
- 先启动BGP的一端先发起TCP连接

## BGP邻居类型

**EBGP**

- 外部边界网关协议
- 运行在不同的AS之间的BGP路由器建立的邻居关系叫EBGP（External BGP）邻居关系

**IBGP**

- 内部边界网关协议
- 运行在相同AS内部的BGP路由器建立的邻居关系为IBGP（Internal BGP）邻居关系
- 在AS内部，不使单独用BGP协议，通常是IGP+BGP

## BGP配置

- `router-id 1.1.1.1`指定router-id
- `bgp 300`进入到自身的AS区域
- `peer 10.1.1.3 as-number 100`对方的IP和AS区域
- `peer 2.2.2.2 connect-interface loopback 0`指定更新源，更稳定

> - BGP是在多个站点之间传递路由，并不是为了在AS内部打通路由
> - EBGP之间建邻居一般使用直连接口
> - IBGP之间建邻居一般使用环回口

[【BGP练习实验】]()

# BGP路由生成方式


# BGP通告原则和路由处理


# BGP常用属性介绍


# BGP选路原则


# BGP路由聚合























