# OSPF综合实验

如下拓扑图：

![OSPF综合实验](F:%5CGitHub%5CHCIP%20R&S%5C%E5%AE%9E%E9%AA%8C%5COSPF%E7%BB%BC%E5%90%88%E5%AE%9E%E9%AA%8C%E4%B8%80%5COSPF%E7%BB%BC%E5%90%88%E5%AE%9E%E9%AA%8C.assets%5COSPF%E7%BB%BC%E5%90%88%E5%AE%9E%E9%AA%8C.png)

**实验需求**：

1. 按照图示配置IP地址，所有路由器配置环回口IP地址为X.X.X.X32作为Router-id,，X为设备编号(R6除外)
2. 按照图示分区域配置OSPF与RIP
3. R6上配置环回口模拟业务网段；并在R2 上进行单点双向路由引入，要求所有业务网段的路由聚合为一条后发布到OSPF区域
4. Area2为了减小路由表规模，配置为Totally Stub区域；要求OSPF区域内所有路由可达
5. OSPF区域内路由器不允许存在12.1.1.0/24网段的路由
6. 为了保证协议安全，Area 0配置区域验证，验证密钥bad-5

# 实验配置

## 配置IP地址

```bash
# AR1
[AR1]int lo 0
[AR1-LoopBack0]ip ad 1.1.1.1 32
[AR1-LoopBack0]int g0/0/0
[AR1-GigabitEthernet0/0/0]ip ad 12.1.1.1 24 
[AR1-GigabitEthernet0/0/0]q

# AR2
[AR2]int lo 0
[AR2-LoopBack0]ip ad 2.2.2.2 32
[AR2-LoopBack0]int g0/0/0
[AR2-GigabitEthernet0/0/0]ip ad 12.1.1.2 24
[AR2-GigabitEthernet0/0/0]int g0/0/1
[AR2-GigabitEthernet0/0/1]ip ad 23.1.1.2 24
[AR2-GigabitEthernet0/0/1]int g0/0/2
[AR2-GigabitEthernet0/0/2]ip ad 24.1.1.2 24
[AR2-GigabitEthernet0/0/2]q

# AR3
[AR3]int lo 0
[AR3-LoopBack0]ip ad 3.3.3.3 32
[AR3-LoopBack0]int g0/0/0
[AR3-GigabitEthernet0/0/0]ip ad 23.1.1.3 24
[AR3-GigabitEthernet0/0/0]int g0/0/1
[AR3-GigabitEthernet0/0/1]ip ad 35.1.1.3 24
[AR3-GigabitEthernet0/0/1]q

# AR4
[AR4]int lo 0
[AR4-LoopBack0]ip ad 4.4.4.4 32
[AR4-LoopBack0]int g0/0/0
[AR4-GigabitEthernet0/0/0]ip ad 24.1.1.4 24
[AR4-GigabitEthernet0/0/0]int g0/0/1
[AR4-GigabitEthernet0/0/1]ip ad 45.1.1.4 24
[AR4-GigabitEthernet0/0/1]q

# AR5
[AR5]int lo 0
[AR5-LoopBack0]ip ad 5.5.5.5 32
[AR5-LoopBack0]int g0/0/0
[AR5-GigabitEthernet0/0/0]ip ad 45.1.1.5 24
[AR5-GigabitEthernet0/0/0]int g0/0/1
[AR5-GigabitEthernet0/0/1]ip ad 35.1.1.5 24
[AR5-GigabitEthernet0/0/1]int g0/0/2
[AR5-GigabitEthernet0/0/2]ip ad 56.1.1.5 24
[AR5-GigabitEthernet0/0/2]q

# AR6
[AR6]int lo 0
[AR6-LoopBack0]ip ad 6.6.6.6 32
[AR6-LoopBack0]int g0/0/0
[AR6-GigabitEthernet0/0/0]ip ad 56.1.1.6 24
[AR6-GigabitEthernet0/0/0]q
```

## 分区域配置RIP和OSPF

```bash
# AR1配置RIP
[AR1]rip
[AR1-rip-1]v 2 
[AR1-rip-1]network 12.0.0.0
[AR1-rip-1]net 1.0.0.0 
[AR1-rip-1]undo summary  //不汇总

# AR2配置RIP
[AR2]rip
[AR2-rip-1]v 2
[AR2-rip-1]undo summary  //不汇总
[AR2-rip-1]net 2.0.0.0
[AR2-rip-1]net 12.0.0.0 
[AR2-rip-1]q
[AR2]dis ip routing-table protocol rip  //查看RIP学到的路由
[AR2]ospf 1 router-id 2.2.2.2
[AR2-ospf-1]area 0
[AR2-ospf-1-area-0.0.0.0]net 23.1.1.0 0.0.0.255
[AR2-ospf-1-area-0.0.0.0]net 24.1.1.0 0.0.0.255
[AR2-ospf-1-area-0.0.0.0]net 2.2.2.2 0.0.0.0
[AR2-ospf-1-area-0.0.0.0]q

# AR3配置OSPF
[AR3]ospf 1 router-id 3.3.3.3
[AR3-ospf-1]area 0
[AR3-ospf-1-area-0.0.0.0]net 3.3.3.3 0.0.0.0
[AR3-ospf-1-area-0.0.0.0]net 23.1.1.0 0.0.0.255
[AR3-ospf-1-area-0.0.0.0]area 1
[AR3-ospf-1-area-0.0.0.1]net 35.1.1.0 0.0.0.255
[AR3-ospf-1-area-0.0.0.0]q
[AR3-ospf-1]q

# AR4配置OSPF
[AR4]ospf 1 router-id 4.4.4.4
[AR4-ospf-1]area 0
[AR4-ospf-1-area-0.0.0.0]net 24.1.1.0 0.0.0.255
[AR4-ospf-1-area-0.0.0.0]net 4.4.4.4 0.0.0.0 
[AR4-ospf-1-area-0.0.0.1]net 45.1.1.0 0.0.0.255
[AR4-ospf-1-area-0.0.0.1]q

# AR5配置OSPF
[AR5]ospf 1 router-id 5.5.5.5
[AR5-ospf-1]area 1
[AR5-ospf-1-area-0.0.0.1]net 5.5.5.5 0.0.0.0
[AR5-ospf-1-area-0.0.0.1]net 35.1.1.0 0.0.0.255
[AR5-ospf-1-area-0.0.0.1]net 45.1.1.0 0.0.0.255
[AR5-ospf-1-area-0.0.0.1]area 2
[AR5-ospf-1-area-0.0.0.2]net 56.1.1.0 0.0.0.255

# AR6配置OSPF
[AR6]ospf 1 router-id 6.6.6.6
[AR6-ospf-1]area 2
[AR6-ospf-1-area-0.0.0.2]net 56.1.1.0 0.0.0.255
[AR6-ospf-1-area-0.0.0.2]net 6.6.6.6 0.0.0.0   
```

## 单点双向路由引入、业务网段路由聚合

```bash
# 单点双向路由引入
[AR2-ospf-1]import-route rip 1  //在OSPF中引入RIP
[AR2-ospf-1]rip
[AR2-rip-1]import-route ospf 1  //在RIP中引入OSPF

# 路由聚合
[AR2]ospf 
[AR2-ospf-1]asbr-summary 1.1.1.0 255.255.255.0  //将1.1.1.0网段聚合
[AR2-ospf-1]q
```

## 配置Totally Stub区域、配置虚连接

```bash
# AR5配置Totally Stub区域
[AR5]ospf 
[AR5-ospf-1]area 2
[AR5-ospf-1-area-0.0.0.2]stub no-summary 
[AR5-ospf-1-area-0.0.0.2]q

# AR3配置虚连接
[AR3-ospf-1-area-0.0.0.1]vlink-peer 5.5.5.5

# AR4配置虚连接
[AR4-ospf-1-area-0.0.0.1]vlink-peer 5.5.5.5

# AR5配置虚连接
[AR5-ospf-1]area 1
[AR5-ospf-1-area-0.0.0.1]vlink-peer 3.3.3.3
[AR5-ospf-1-area-0.0.0.1]vlink-peer 4.4.4.4
[AR5-ospf-1-area-0.0.0.1]dis ospf vlink  //查看虚连接状态

# AR6配置
[AR6-ospf-1-area-0.0.0.2]stub 
```

## 汇总路由，且不通告

```bash
[AR2]ospf
[AR2-ospf-1]asbr-summary 12.1.1.0 255.255.255.0 not-advertise  //汇总并且不通告
```

## 配置认证

```bash
# AR2配置
[AR2-ospf-1]area 0
[AR2-ospf-1-area-0.0.0.0]authentication-mode  md5 1 plain bad-5  //使用MD5加密

# AR3配置
[AR3-ospf-1-area-0.0.0.0]authentication-mode md5 1 plain bad-5

#AR4配置 
[AR4-ospf-1]area 0
[AR4-ospf-1-area-0.0.0.0]authentication-mode md5 1 plain bad-5

# 配置认证，方法1
[AR5]ospf
[AR5-ospf-1]area 0
[AR5-ospf-1-area-0.0.0.0]authentication-mode md5 1 plain bad-5
[AR5-ospf-1-area-0.0.0.0]q

# 配置认证 方法2
[AR5-ospf-1]area 1
[AR5-ospf-1-area-0.0.0.1]vlink-peer 3.3.3.3 md5 1 plain bad-5
[AR5-ospf-1-area-0.0.0.1]vlink-peer 4.4.4.4 md5 1 plain bad-5
[AR5-ospf-1-area-0.0.0.1]dis ospf vlink  //查看虚连接状态 
```

> AR5上两种认证方法都可以

## AR1与AR6连通性测试

```bash
# 指定源地址ping，物理接口地址被过滤
<AR1>ping -a 1.1.1.1 6.6.6.6  //指定1.1.1.1ping6.6.6.6
```

> 原物理接口的IP被过滤了，所以不能和6.6.6.6ping通额