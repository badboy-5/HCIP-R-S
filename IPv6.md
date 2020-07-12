---
姓名：坏坏
学习时间：2020年6月29日
整理时间：2020年6月30日
---

# 回顾IPv4地址

- IPv4地址长度为32Bit，可用的地址为2^32^
- IPv6地址长度为128Bit，可用的地址为2^128^

**IPv4报文头部**

![IPv4报文头部](F:\GitHub\HCIP R&S\IPv6.assets\IPv4报文头部.png)

> - Version字段为4表示为IPv4，6为IPv6
> - TOS做QOS时使用
> - Identification、Flags、Fragment Offset分片使用
> - Protocol表示下一层的协议（TCP为6，UDP为17）

**IPv6报文头部**

![IPv6报文头部](F:\GitHub\HCIP R&S\IPv6.assets\IPv6报文头部.png)

> - IPv6报文头部长度固定为40字节，IPv4头部长度变化范围为20-60字节
> - Traffic Class相当于IPv4报文头的Type of Service（TOS），主要用于Qos
> - Flow Label用于区分实时流量
> - Payload Length表示紧跟在IPv6报头后的其他数据报文
> - Next Header相当于IPv4报文头部的Protocol，表示紧跟在IPv6报头后面的第一个拓展报头的类型
> - Hop Limit是跳数限制，相当于TTL值

# IPv6拓展报头

- IPv6拓展报头跟在IPv6基本报头后面的可选报头，可以有一个或多个
- 拓展报头的类型（也必须按照此顺序）
	* IPv6基本报头
	* 逐跳选项拓展报头
	* 目的选项拓展报头
	* 路由拓展报头
	* 分片拓展报头
	* 认证拓展报头
	* 封装安全有效载荷拓展报头
	* 目的选项拓展报头
	* 上层协议数据报文

# IPv6地址格式

- IPv6地址长度为128Bit，每16Bit划分为一段，每段由4个十六进制数表示，用冒号隔开
- IPv6地址包括
	* 网络前缀：IPv6地址前缀用来标识IPv6网络，相当于IPv4中的网络位
	* 接口标识：用接口ID标识接口，相当于IPv4中的主机位

> - IPv6没有掩码概念，只有前缀长度

# IPv6地址压缩格式

- 地址中包含连续两个或多个均为0的组，可以用双冒号来代替。<font color=FF0000>（在一个IPv6的地址中，只能使用一次双冒号）</font>
- IPv6中连续的0可以用两个冒号表示，4个0可以用一个0表示
- 每16Bit组中的前导0可以省略

# IPv6地址分类

| 地址范围 | 描述 | 对应IPv4地址类型 |
|:---:|:---:|:---|
| 2000::/3 | 全球单播地址 | 公网地址 |
| FC00::/7 | 唯一本地地址 | 私网地址 |
| FE80::/10 | 链路本地地址 |  |
| FF00::/8 | 组播地址 | 与IPv4组播相同 |
| ::/128 | 未指定地址 |  |
| ::1/128 | 环回地址 |  |

- IPv6地址分为单播地址、任播地址、组播地址
- 链路本地地址只能在连接到同一本地链路的节点时间使用（链路本地地址是IPv6新增的地址）
- IPv6的组播地址前两位固定为FF

![预定义组播地址](F:\GitHub\HCIP R&S\IPv6.assets\预定义组播地址.png)

## IPv6单播地址

- 全球单播地址中带有固定前缀，相当于IPv4中的公网地址
- 单播地址包括：
	* 全球单播地址：带有固定的前缀（前面三位固定为001）
	* 链路本地地址：固定前缀为FE80::/10，表示地址最高10位值为1111111010。后跟64位的接口标识

## IPv6组播地址

- IPv6组播地址前缀是FF00::/8（1111 1111）
- IPv6为需要使用组播发送数据的协议预留了组播组

## IPv6任播地址

- 目标地址是任播地址的数据包将发送给其中路由意义上最近的一个网络接口

## IPv6配置

```bash
[AR1]ipv6  //全局开启IPv6
[AR1]int g0/0/0
[AR1-GigabitEthernet0/0/0]ipv6 enable  //接口
[AR1-GigabitEthernet0/0/0]ipv6 address 2000::1 64
[AR1]dis ipv6 interface
GigabitEthernet0/0/0 current state : DOWN 
IPv6 protocol current state : DOWN
IPv6 is enabled, link-local address is FE80::2E0:FCFF:FE1D:284E [TENTATIVE]  //链路本地地址
  Global unicast address(es):   //GUA全球单播地址
    2000::1, subnet is 2000::/64 [TENTATIVE]  //公网地址2000：：1，网段为2000：：
  Joined group address(es):
    FF02::1:FF00:1  //由请求节点固定位+2000：：1的后24位构成
    FF02::2  //所有路由器组播地址
    FF02::1  //所有节点组播地址
    FF02::1:FF1D:284E  //Solicited-Node（请求节点）组播地址
  MTU is 1500 bytes
  ND DAD is enabled, number of DAD attempts: 1
  ND reachable time is 30000 milliseconds
  ND retransmit interval is 1000 milliseconds
  Hosts use stateless autoconfig for addresses

[AR1]
```

> - 请求节点（Solicited-Node）组播地址的前104位是：`FF02::1:FF`
> - 后24为是链路本地地址的后24位或全球单播地址的后24位

## IPv6接口标识生成方法

- 手工配置
- 系统通过软件自动生成
- IEEE EUI-64规范生成

**IEEE EUI-64规范**

1. 在MAC地址的前24位和后24位之间插入16Bit的FFFE
2. 将MAC地址的高七位从0变成1，生成64Bit的接口ID

> -　MAC地址的前24位代表厂商ID（华为的厂商标识：0x00E0FC），后24位代表制造商分配的唯一扩展标识
> -　接口下通过EUI-64自动生成接口标识`ipv6 address 2002:: 64 eui-64`

# IPv6的基本实现

![IPv6的基本实现](F:\GitHub\HCIP R&S\IPv6.assets\IPv6的基本实现.png)

## 地址解析

![地址解析](F:\GitHub\HCIP R&S\IPv6.assets\地址解析.png)

## 重复地址检测

![重复地址检测](F:\GitHub\HCIP R&S\IPv6.assets\重复地址检测.png)

## 路由器发现

![路由器发现](F:\GitHub\HCIP R&S\IPv6.assets\路由器发现.png)

## 重定向

![重定向](F:\GitHub\HCIP R&S\IPv6.assets\重定向.png)

**总结**

| 报文 | 类型 | Type字段值 |
|:---:|:---:|:---:|
| 路由器请求报文 | RS | 133 |
| 路由器通告报文 | RA | 134 |
| 邻居请求报文 | NS | 135 |
| 邻居通告报文 | NA | 136 |
| 重定向 |  | 137 |

## Path MTU

![PMTU](F:\GitHub\HCIP R&S\IPv6.assets\PMTU.png)