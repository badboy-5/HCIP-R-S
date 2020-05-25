---
姓名：坏坏
实验时间：2020年5月25日
整理时间：2020年5月25日
---

# OSPF练习

如下拓扑：

![OSPF练习](F:%5CGitHub%5CHCIP%20R&S%5C%E5%AE%9E%E9%AA%8C%5COSPF%E7%BB%83%E4%B9%A0%5COSPF%E7%BB%83%E4%B9%A0.assets%5COSPF%E7%BB%83%E4%B9%A0.png)

**实验需求**：

1. 按照图示配置 IP 地址
2. R2，R3，R4 运行 OSPF 使内网互通，所有接口（公网接口除外）全部宣告进Area 0；要求使用环回口作为Router-id
3. 业务网段不允许出现协议报文
4. R1 模拟互联网，内网通过 R2 连接互联网，在 R2上配置默认路由并引入到 OSPF
5. R2 上配置 EASY IP，只允许业务网段访问互联网
6. 要求业务网段访问互联网流量经过 R4，R3，R2

## 基础配置

```bash
--------------------------------------------
# AR1基础配置
interface GigabitEthernet0/0/0
 ip address 12.1.1.1 255.255.255.0 
#
interface LoopBack0
 ip address 1.1.1.1 255.255.255.255 
--------------------------------------------
# AR2基础配置
interface GigabitEthernet0/0/0
 ip address 12.1.1.2 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 24.1.1.2 255.255.255.0 
#
interface GigabitEthernet0/0/2
 ip address 23.1.1.2 255.255.255.0 
#
interface LoopBack0
 ip address 2.2.2.2 255.255.255.255
--------------------------------------------
# AR3基础配置
interface GigabitEthernet0/0/0
 ip address 34.1.1.3 255.255.255.0 
#
interface GigabitEthernet0/0/1
 ip address 23.1.1.3 255.255.255.0 
#
interface LoopBack0
 ip address 3.3.3.3 255.255.255.255
--------------------------------------------
#  AR4基础配置
interface GigabitEthernet0/0/0
 ip address 192.168.1.254 255.255.255.0
#
interface GigabitEthernet0/0/1
 ip address 34.1.1.4 255.255.255.0 
#
interface GigabitEthernet0/0/2
 ip address 24.1.1.4 255.255.255.0
#
interface LoopBack0
 ip address 4.4.4.4 255.255.255.255 
```

## 配置OSPF

```bash
# AR2配置
[AR2]ospf router-id 2.2.2.2
[AR2-ospf-1]area 0
[AR2-ospf-1-area-0.0.0.0]net 2.2.2.2 0.0.0.0
[AR2-ospf-1-area-0.0.0.0]net 23.1.1.0 0.0.0.255
[AR2-ospf-1-area-0.0.0.0]net 24.1.1.0 0.0.0.255

# AR3配置
[AR3]ospf router-id 3.3.3.3
[AR3-ospf-1]area 0
[AR3-ospf-1-area-0.0.0.0]net 3.3.3.3 0.0.0.0
[AR3-ospf-1-area-0.0.0.0]net 23.1.1.0 0.0.0.255
[AR3-ospf-1-area-0.0.0.0]net 34.1.1.0 0.0.0.255

# AR4配置
[AR4]ospf router-id 4.4.4.4
[AR4-ospf-1]area 0
[AR4-ospf-1-area-0.0.0.0]net 4.4.4.4 0.0.0.0
[AR4-ospf-1-area-0.0.0.0]net 34.1.1.0 0.0.0.255
[AR4-ospf-1-area-0.0.0.0]net 24.1.1.0 0.0.0.255
[AR4-ospf-1-area-0.0.0.0]net 192.168.1.0 0.0.0.255
```

## 配置静默接口

- 业务网段不允许出现协议报文

```bash
[AR4]ospf 
[AR4-ospf-1]silent-interface g0/0/0
```

## 配置默认路由，并引入OSPF

- R1 模拟互联网，内网通过 R2 连接互联网，在 R2上配置默认路由并引入到 OSPF

```bash
[AR2]ip route-static 0.0.0.0 0 12.1.1.1  //配置默认路由
[AR2]ospf 
[AR2-ospf-1]default-route-advertise  //OSPF中引入默认路由
```

## 配置Easy IP

- R2 上配置 EASY IP，只允许业务网段访问互联网

```bash
[AR2]acl 2000
[AR2-acl-basic-2000]rule 5 permit source 192.168.1.0 0.0.0.255
[AR2-acl-basic-2000]int g0/0/0
[AR2-GigabitEthernet0/0/0]nat outbound 2000
```

	PC与AR1的环回口做连通性测试

## 修改路径开销

- 要求业务网段访问互联网流量经过 R4，R3，R2

```bash
[AR4]int g0/0/2
[AR4-GigabitEthernet0/0/2]ospf cost 3
```

	查看路由表，通往外网是否通过AR4,AR3，AR2