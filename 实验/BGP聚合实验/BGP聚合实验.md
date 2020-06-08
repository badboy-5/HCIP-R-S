---
姓名：坏坏
实验时间：2020年6月8日
整理时间：2020年6月8日
---

# BGP聚合实验

如下拓扑：

![BGP聚合拓扑](F:%5CGitHub%5CHCIP%20R&S%5C%E5%AE%9E%E9%AA%8C%5CBGP%E8%81%9A%E5%90%88%E5%AE%9E%E9%AA%8C%5CBGP%E8%81%9A%E5%90%88%E5%AE%9E%E9%AA%8C.assets%5CBGP%E8%81%9A%E5%90%88%E6%8B%93%E6%89%91.png)

## 基本配置

```bash
# AR2基础配置
 sysname AR2
#
interface GigabitEthernet0/0/0
 ip address 12.1.1.2 255.255.255.0 
#
interface LoopBack0
 ip address 2.2.2.1 255.255.255.255 
#
interface LoopBack1
 ip address 2.2.2.2 255.255.255.255 
#
interface LoopBack2
 ip address 2.2.2.3 255.255.255.255
```

```bash
# AR1基础配置
 sysname AR1
#
interface GigabitEthernet0/0/0
 ip address 12.1.1.1 255.255.255.0 
```

## 配置BGP

```bash
[AR1] bgp 100
[AR1-bgp]peer 12.1.1.2 as-number 100
```

```bash
[AR2]bgp 100
[AR2-bgp]peer 12.1.1.1 as-number 100
```

> 查看BGP邻居状态`display bgp peer`

## 发布环回口

```bash
[AR2-bgp]net 2.2.2.1 32
[AR2-bgp]net 2.2.2.2 32
[AR2-bgp]net 2.2.2.3 32
````

> - 查看BGP路由表`display bgp routing-table`
> - 在R1查看路由表是否有2.2.2.1/2/3的明细路由

## 自动聚合

- 取消network的路由，BGP引入直连路由

```bash
[AR2-bgp]undo net 2.2.2.1 32
[AR2-bgp]undo net 2.2.2.2 32
[AR2-bgp]undo net 2.2.2.3 32
[AR2-bgp]import-route direct  //引入直连路由
```

```bash
[AR2-bgp]summary automatic  //自动聚合
```

> - 自动聚合对network的路由没有用，只对引入的路由有效
> - 在R1上查看BGP路由表，是否有聚合的2.0.0.0路由

## 手动聚合

- 使用network获得环回口的路由

```bash
[AR2-bgp]undo import-route direct 
[AR2-bgp]net 2.2.2.1 32
[AR2-bgp]net 2.2.2.2 32
[AR2-bgp]net 2.2.2.3 32
```

- 配置路由策略

```bash

[AR2]route-policy 1 permit node 1
[AR2-route-policy]apply as-path 100 200 11 additive 
[AR2-route-policy]apply local-preference 20
[AR2-route-policy]apply cost 30
[AR2-route-policy]apply community 100:1
[AR2-route-policy]apply origin egp 1
[AR2-route-policy]apply preference 100
```

```bash
[AR2]route-policy 2 permit node 1
[AR2-route-policy]apply as-path 300 3 5 additive 
[AR2-route-policy]apply local-preference  22
[AR2-route-policy]apply cost 40
[AR2-route-policy]apply community 100:2
[AR2-route-policy]apply origin incomplete 
[AR2-route-policy]apply preference 33
```

```bash
[AR2]route-policy 3 permit node 1
[AR2-route-policy]apply as-path 33 55 additive 
[AR2-route-policy]apply local-preference 89
[AR2-route-policy]apply cost 65
[AR2-route-policy]apply community 100:3
[AR2-route-policy]apply origin igp
[AR2-route-policy]apply preference 66
```

## 给发布的路由添加路由策略

```bash
[AR2]bgp 100
[AR2-bgp]network 2.2.2.1 32 route-policy 1
[AR2-bgp]network 2.2.2.2 32 route-policy 2
[AR2-bgp]network 2.2.2.3 32 route-policy 3
```

> - `route-policy`中的动作全部加入给路由后再发布
> - R1查看BGP路由表

**添加Route-Policy前**

![添加Route-Policy前](F:%5CGitHub%5CHCIP%20R&S%5C%E5%AE%9E%E9%AA%8C%5CBGP%E8%81%9A%E5%90%88%E5%AE%9E%E9%AA%8C%5CBGP%E8%81%9A%E5%90%88%E5%AE%9E%E9%AA%8C.assets%5C%E6%B7%BB%E5%8A%A0Route-Policy%E5%89%8D.png)

**添加Route-Policy后**

![添加Route-Policy后](F:%5CGitHub%5CHCIP%20R&S%5C%E5%AE%9E%E9%AA%8C%5CBGP%E8%81%9A%E5%90%88%E5%AE%9E%E9%AA%8C%5CBGP%E8%81%9A%E5%90%88%E5%AE%9E%E9%AA%8C.assets%5C%E6%B7%BB%E5%8A%A0Route-Policy%E5%90%8E.png)

## 配置聚合

```bash
[AR2]bgp 100
[AR2-bgp]aggregate 2.2.2.0 30  //聚合
[AR2-bgp]peer 12.1.1.1 advertise-community  //通告团体属性，默认不通告
```

> - 在R1查看BGP路由表，是否有聚合的路由
> - 聚合后团体属性都没有了

**通告团体属性后查看明细路由详细信息**

![明细路有详细信息](F:%5CGitHub%5CHCIP%20R&S%5C%E5%AE%9E%E9%AA%8C%5CBGP%E8%81%9A%E5%90%88%E5%AE%9E%E9%AA%8C%5CBGP%E8%81%9A%E5%90%88%E5%AE%9E%E9%AA%8C.assets%5C%E6%98%8E%E7%BB%86%E8%B7%AF%E6%9C%89%E8%AF%A6%E7%BB%86%E4%BF%A1%E6%81%AF.png)

**通告团体属性后查看聚合路由详细信息**

![聚合路由详细信息](F:%5CGitHub%5CHCIP%20R&S%5C%E5%AE%9E%E9%AA%8C%5CBGP%E8%81%9A%E5%90%88%E5%AE%9E%E9%AA%8C%5CBGP%E8%81%9A%E5%90%88%E5%AE%9E%E9%AA%8C.assets%5C%E8%81%9A%E5%90%88%E8%B7%AF%E7%94%B1%E8%AF%A6%E7%BB%86%E4%BF%A1%E6%81%AF.png)

## 抑制明细

- 明细路由不传递

```bash
[AR2]bgp 100
[AR2-bgp]aggregate 2.2.2.0 30 detail-suppressed  //抑制明细路由
```

> 在R1查看BGP路由表，不存在明细路由，只有聚合路由

## 继承明细AS-Path值

```bash
[AR2-bgp]aggregate 2.2.2.0 30 as-set  //聚合路由继承明细路由的AS-Path值
```

> 在R1查看BGP路由表，聚合路由有了Path值，括号内的Path值无顺序

## 属性策略

- 给聚合路由添加属性

- 添加路由策略

```bash
[AR2]route-policy 4 permit node 1
[AR2-route-policy]apply cost 100
[AR2-route-policy]apply local-preference 10
[AR2-route-policy]apply origin egp 1
[AR2-route-policy]apply as-path 10 20 additive 
```

- 添加属性策略

```bash
[AR2]bgp 100
[AR2-bgp]aggregate 2.2.2.0 32 attribute-policy 4
```

> - 在R1查看BGP路由表，聚合路由会增加详细的属性
> - 具体的属性在Route-Policy中调整

## 起源策略

- 给聚合设置条件

- 配置ACL

```bash
[AR2]acl 2000
[AR2-acl-basic-2000]rule permit source 2.2.2.1 0
```

- 添加Route-Policy

```bash
[AR2]route-policy 5 permit node 1 
[AR2-route-policy]if-match acl 2000  //调用ACL 2000
```

- 修改起源属性

```bash
[AR2]bgp 100
[AR2-bgp]aggregate 2.2.2.0 30 origin-policy 5
```

> 在R1查看BGP路由表，汇聚路由Path从`？`变成了`e`

- 在R2上把2.2.2.1（LoopBack 0）删除

```bash
[AR2]int lo 0
[AR2-LoopBack0]undo ip ad
```

> - 在R1查看BGP路由表，汇聚路由消失
> - 当指定的路由不存在时，将不聚合

## 抑制策略

- 抑制某些明细不传递

```bash
[AR2-bgp]aggregate 2.2.2.0 30 detail-suppressed suppress-policy 5
```

> - 抑制Route-Policy 5匹配到的2.2.2.1的路由，其他的明细不影响
> - 在R1查看BGP路由表，没有2.2.2.1的明细路由