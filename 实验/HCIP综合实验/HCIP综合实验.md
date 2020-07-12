---
姓名：坏坏
学习时间：2020年7月3日
整理时间：2020年7月3日
---

![HCIP综合实验拓扑](F:\GitHub\HCIP R&S\实验\HCIP综合实验\HCIP综合实验.assets\HCIP综合实验拓扑.png)


1. 全网依照拓扑图配置vlan和IP地址
2. 总公司：
	* SW3和SW4的互连接口启用eth-trunk，使用静态LACP模式，允许最大带宽为2G
	* SW1、SW2、SW3和SW4运行MSTP，SW3为VLAN100的Root,SW4为VLAN200的Root
	* PC1-PC4 需要提供网关冗余，为了提高安全性，需要做认证，并使用BFD动态检测上行链路状态，实现自动切换
3. AS100需求：
	* 每台设备都需要配置Loopback接口，地址为X.X.X.X (X为设备编号)
	* AS100底层IGP协议为IS-IS，设备级别为level-2，确保各路由器的loopback接口互通
	* R1与R4建立IBGP邻居(尽量确保邻居关系稳定)
	* MPLS-VPN需求：
		+ 总公司的PC能访问分公司1/2的PC，分公司之间不能互访
		+ R1和SW3、SW4之间运行0SPF协议
		+ R4和R5之间运行BGP协议
		+ R4和R6之间运行0SPF协议
		+ R1和R4建立MP-BGP邻居
4. 分公司1需求：
	* SW5为二层交换机，PC5与PC6配置不同VLAN(属于不同网段)，确保两台PC能互访
5. 分公司2需求：
	* PC7与PC8属于不同VLAN(相同网段)，通过VLANIF技术让两台PC正常访问总公司，但是不能互访
	* 内部IGP运行ISIS协议，为了加快收敛速度，每网段不允许存在DIS



2. （1）改一下模式，最大链路（2）配置MSTP，一个实例100，一个实例200，强行指定根（3）配置VRRP，实现VRRP与MSTP联动，在SW3、4上检测上行链路
3. MPLS VPN（1）RT（2）路由引入（4单点双向引入）


4.配置单臂路由
5.（1）super vlan（2）网络类型P2P























