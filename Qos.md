---
姓名：坏坏
学习时间：2020年7月1日
整理时间：2020年7月3日
---

# Qos服务模型

- Qos(Quality of Service)在带宽有限的情况下，Qos应用一个“有保证”的策略对网络流量进行管理，并实现不同的流量可以获得不同的优先服务
- 要提高通信质量就要提高带宽、减少时延和抖动、降低丢包率

| 类型 | 单位 | 说明 |
|:---:|:---|:---|
| 网络带宽 | 单位时间（一般是1s）内传输的数据量 | 最大带宽等于传输路径上的最小带宽 |
| 网络时延 | 一个报文从网络的一端传送到另一端所需要的时间 | 一个报文从一个网络的一端传送到另一端所需要的时间 |
| 抖动 | 每个报文的端到端时延不一样，会导致报文不能等间隔到达目的端 | 抖动是由于每个保温的端到端时延不相等造成的 |
| 丢包率 | 在网络传输过程中丢失报文占传输报文的百分比 | 丢包可能发生在传输过程中的每一个环节 |

> - 单个设备的时延包括：
> 	* **传输时延**：一个数据从发送方到接收方所需要的时间。该时延取决于传输距离和传输介质，与带宽无关
> 	* **串行化时延**：指发送节点在传输链路上开始发送报文的第一个比特至发完该报文的最后一个比特所需要的时间。该时延取决于链路带宽及报文的大小
> 	* **处理时延**：指路由器把报文从入接口放到出接口所需的时间。该时延与路由器的处理性能有关
> 	* **队列时延**：指报文在队列中等待的时间。该时延与队列中报文的大小、数量、带宽以及队列机制有关
> - 抖动的大小跟时延的大小直接相关，时延小，抖动小；时延大，抖动大
> - 丢包可能发生的环节：
> 	* 处理过程
> 	* 排队过程
> 	* 传输过程

**各类业务对服务质量的要求**

| 流量类型 | 带宽 | 时延 | 抖动 | 丢包率 |
|:---:|:---:|:---:|:---:|:---:|
| 语音 | 低 | 高 | 高 | 低 |
| 视频 | 高 | 高 |高 | 低 |
| FTP | 中，高 | 低 | 低 | 高 |
| 电子邮件、HTTP网页浏览 | 低 | 低 | 低 | 中，高 |

## 尽力而为服务模型

- 先进先出转发即Best-Effort（尽力而为）服务模型
	* 一个单一（最简单）的服务模型，应用程序可以在任意时候，发出任意数量的报文，不需要事先获得批准，不需要通知网络
	* 尽最大的可能性来发送报文，对时延、可靠性等性能不提供保证，适用于绝大多数网络应用
	* 是现在Internet的缺省服务类型，通过先入先出（FIFO）队列来实现
- 在尽力而为的服务模型下，可以通过两种方式来提高端到端的通信质量：
	* 增大网络带宽：可以增大单位时间内传输的数据量
		+ 优点：可以改善带宽瓶颈、串行优化延迟、丢包等问题
		+ 缺点：网络建设成本高
	* 升级网络设备：可以增大数据处理能力
		+ 优点：可以改善处理延迟、队列延迟、丢包等问题
		+ 缺点：成本较高，替换设备增大业务中断风险

## 综合服务模型

- 综合服务模型（Integrated Services Model），使设备运行一些协议（RSVP）来保障关键业务的通信质量
- RSVP协议工作过程：在应用程序发送报文前，需要向网络申请特定的带宽和所需要的特定服务质量的请求，等收到确认信息后才发送报文
- 优点：可以为某些特定业务提供贷款、延迟保障
- 缺点：
	* 实现较为复杂
	* 当没有流量发送时，仍然独占带宽，使用效率较低
	* 该方案要求端到端所有节点设备都支持并运行RSVP协议

> 综合服务模型在现实网络中并不多见

## 区分服务模型

- 区分服务模型（DiffServ），首先将网络中的流量分成多个类，然后每个类定义相应的处理行为，使其拥有不同的优先转发、丢包率、时延等
- 目前应用最广的就是区分服务模型

# 三种服务模型对比

| 服务模型 | 优点 | 缺点 |
|:---:|:---|:---|
| 尽力而为服务模型 | 实现机制简单 | 对不同的业务流不能进行区分对待 |
| 综合服务模型 | 可提供端到端的Qos服务，并保证带宽延迟 | 需要跟踪和记录每个数据流的状态，实现较为复杂，且拓展性较差，带宽利用率较低 |
| 区分服务模型 | 不需要跟踪每个数据流状态，资源占用少，拓展性较强；且能实现对不同业务流提供不同的服务质量 | 需要在端到端每个节点都进行手工部署，对人员能力要求较高 |

# 报文分类与标记

## 简单流分类

- 依据不同链路类型传输的不同类别的报文，且自身含有的标识Qos优先级的字段值进行分类

## 各报文中的Qos分类

- WLAN帧头中的802.1P字段（取值范围0~7）

![WLAN帧头](F:\GitHub\HCIP R&S\Qos.assets\WLAN帧头.png)

- MPLS报文中的EXP字段（取值范围0~7）

![MPLS报文](F:\GitHub\HCIP R&S\Qos.assets\MPLS报文.png)

- IPv4报文中的IP-Precedence字段标识其优先级（取值范围0~7）

![IPv4报文](F:\GitHub\HCIP R&S\Qos.assets\IPv4报文.png)

> - D代表延迟（Delay），T代表吞吐量（Throughput），R代表可靠性（Reliability）
> - 缺点：IP-Precedence字段最多只能将IP报文分为8类，而在网络部署时，这些优先级不够使用

## DSCP值

- DSCP只有6Bits
- DSCP值的两种表达方式：
	* 数字形式：DSCP取值范围为0~63
	* 关键字表达式：用关键字标识DSCP值

![DSCP值关键字表达式](F:\GitHub\HCIP R&S\Qos.assets\DSCP值关键字表达式.png)

- BE相当于尽力而为，先进先出
- AF成功转发，优先级比较中等，AFxy中
	* x代表不同的类别，由DSCP的前三位Bits组成，标识优先级的高低（4>3>2>1）
	* y代表队列被装满时丢包的概率，由DSCP前三位后的两位Bits组成，y越大越容易被丢弃
	* DSCP六位Bits中，最后一位固定为0
- 由xy组成的五位二进制数对应着数字形式中的十进制数
- EF快速转发，优先级较高

<table  style="text-align:center">
	<tr style="font-weight:bold">
        <td colspan="4">DSCP Name</td>
		<td colspan="3">DSCP Value</td>
		<td >IP-Precedence</td>
		<td>802.1P</td>
		<td>Exp</td>
	</tr>
	<tr>
	   <td>BE</td>
	   <td colspan="3">BE(CS0)</td>
	   <td colspan="3">0</td>
	   <td colspan="3">0</td>
	</tr>
	<tr>
	    <td rowspan="8">CS</td>
	    <tr>
	        <td colspan="3">CS1</td>
	        <td colspan="3">8</td>
	        <td colspan="3">1</td>
	    </tr>
	    <tr>
    	    <td colspan="3">CS2</td>
	        <td colspan="3">16</td>
	        <td colspan="3">2</td>
    	</tr>
    	<tr>
    	    <td colspan="3">CS3</td>
	        <td colspan="3">24</td>
	        <td colspan="3">3</td>
    	</tr>
    	<tr>
    	    <td colspan="3">CS4</td>
	        <td colspan="3">32</td>
	        <td colspan="3">4</td>
    	</tr>
    	<tr>
    	    <td colspan="3">CS5</td>
	        <td colspan="3">40</td>
	        <td colspan="3">5</td>
    	</tr>
    	<tr>
    	    <td colspan="3">CS6</td>
	        <td colspan="3">48</td>
	        <td colspan="3">6</td>
    	</tr>
    	<tr>  
    	    <td colspan="3">CS7</td>
	        <td colspan="3">56</td>
	        <td colspan="3">7</td>
    	</tr>
	</tr>
	<tr>
	    <td rowspan="5" >AF</td>
	    <tr>
	        <td>AF11</td>
	        <td>AF12</td>
	        <td>AF13</td>
	        <td>10</td>
	        <td>12</td>
	        <td>14</td>
	        <td colspan="3">1</td>
	    </tr>
	    <tr>
	        <td>AF21</td>
	        <td>AF22</td>
	        <td>AF23</td>
	        <td>18</td>
	        <td>20</td>
	        <td>22</td>
	        <td colspan="3">2</td>
	    </tr>
	    <tr>
	        <td>AF31</td>
	        <td>AF32</td>
	        <td>AF33</td>
	        <td>26</td>
	        <td>28</td>
	        <td>30</td>
	        <td colspan="3">3</td>
	    </tr>
	    <tr>
	        <td>AF41</td>
	        <td>AF42</td>
	        <td>AF43</td>
	        <td>34</td>
	        <td>36</td>
	        <td>38</td>
	        <td colspan="3">4</td>
	    </tr>
	</tr>
	<tr>
	    <td>EF</td>
	    <td colspan="3">EF</td>
	    <td colspan="3">46</td>
	    <td colspan="3">5</td>
	</tr>
</table>

> DSCP Name中的CS只使用了DSCP的高三位Bits，所以只有CS1~7

## 复杂流分类

- 根据五元组(源、目地址，源、目端口号，协议号)等报文信息对报文进行惊喜的分类

![复杂流分类](F:\GitHub\HCIP R&S\Qos.assets\复杂流分类.png)

**报文分类配置**

```bash
traffic classifier manager
 if-match source-mac 3333-3333-3333
traffic classifier voice
 if-match 8021p 3
traffic classifier video
 if-match 8021p 2
```

**报文标记配置**

```bash
traffic behavior manager
 remark 8021p 1
traffic behavior voice
 remark 8021p 5 
traffic behavior video
 remark 8021p 3
-----------------------------------------
traffic policy a1
 classifier manager behavior manager
 classifier voice behavior voice
 classifier video behavior video
```

**接口调用**

```bash
int g0/0/0
 traffic-policy a1 inbound
int g0/0/1
 traffic-policy a1 inbound
int g0/0/2
 traffic-policy a1 inbound
int g0/0/3
 traffic-policy a1 inbound
```

# 拥塞管理

- 拥塞管理通过队列机制来实现：
	* 将准备从一个接口发出的所有报文放入不同的缓存队列中
	* 根据各队列间的调度机制实现不同报文的拆分转发

- LP（本地优先级，又称内部优先级）优先级映射实现从数据原始携带的Qos优先级到内部优先级或内部优先级到Qos优先级的映射
- 对于进入设备的报文，设备将报文携带的优先级或者端口优先级映射为内部优先级，然后根据内部优先级与队列之间的映射关系确定报文进入的队列

## FIFO（First In First Out）

- FIFO先进先出队列
- FIFO队列不对报文进行分类，先进的报文将先出队，后进的报文将后出队
- FIFO队列具有处理简单、开销小的优点
- FIFO队列的缺点：不能有差别的对待优先级不同的报文

## PQ(Priority Queuing)

- PQ优先级队列
- 针对关键业务应用设计，关键业务在拥塞发生时，需要优先获得服务减少响应延迟
- PQ调度机制：分为高优先队列、中优先队列、正常优先队列、低优先队列，优先级依次降低
- 报文出队时，会优先让高优先队列的报文发送完，然后再依次按照优先级发送报文
- 优点：对高优先级的报文提供了优先转发
- 缺点：优先队列可能出现“饿死”现象

> 如果高优先队列中一直有报文等待发送，后面优先级较低的报文就迟迟不能得到发送，出现“饿死”现象

## WRR（Weight Round Robin）

- WRR加权循环调度
- 根据每个队列的权重来了轮流调度各队列中的报文流
- 优点：避免了PQ调度的“饿死”现象
- 缺点：基于报文个数来调度，容易出现包长尺寸不同的报文出现不平等调度；低时延业务得不到及时调度

## WFQ（Weighted Fair Queuing）

- WFQ加权公平队列
- 根据队列的权重（优先级）不同，划分不同的带宽，带宽大的走的多，带宽小的走的少
- 不同优先级算法：报文长度/(优先级+1)
- 优点：可保证低时延业务得到及时调度；实现按权重分配带宽
- 缺点：无法实现根据用户自定义灵活分类报文的需求

> PQ+WFQ调度:重要的协议报文以及有低时延需求的业务报文应放入PQ调度队列中，得到优先调度的机会，其他报文则可放入以WFQ方式调度的各队列中

## CBQ(Class-based Queueing)

- CBQ基于类的加权公平队列
- EF队列：满足低时延业务
	* EF队列拥有绝对优先级，仅当EF队列中的报文调度完毕后，才会调度其他队列中的报文
- AF队列：满足需要带宽保证的关键数据业务
	* 每个AF队列分别对应一类报文，用户可以设定每类报文占用的带宽。当系统调度报文出队的时候，会按用户为各类报文设定的带宽将报文进行出队发送，可实现各个类的队列的公平调度
- BE队列：满足不需要严格QoS保证的尽力发送业务
	* 当报文不匹配用户设定的所有类别时，报文会被送入系统定义的缺省BE类。BE队列使用接口剩余带宽和WFQ调度方式进行发送
- 优点：提供了自定义类的支持；可为不同的业务定义不同的调度策略
- 缺点：由于涉及到复杂的流分类，启用CBQ会耗费一定的系统资源

# 队列调度算法比较

![队列调度算法比较](F:\GitHub\HCIP R&S\Qos.assets\队列调度算法比较.png)

```bash
qos queue-profile qos-Huawei
schedule pq 5 wfq 1 to 3  //配置Qos
int g0/0/0
qos queue-profile qos-Huawei  //接口调用

dis qos queue-profile qos-Huawei  //查看Qos
```

# 拥塞避免

- 队列被装满后，传统的处理方法会将后续向该队列发送的报文全部丢弃，直至拥塞接触（**尾丢弃**）

## 尾丢弃的缺点：

1. 引发TCP全局同步：对于TCP报文，如果大量的报文被丢弃，将造成TCP超时，从而引发TCP慢启动，使得TCP减少报文的发送
> 解决办法：RED（Random Early Detection）随机早期检测，减缓TCP全局同步。在队列未装满时，随机丢弃一部分报文
> - RED为每个队列的长度都设定了阈值门限，并规定： 
> 	* 当队列的长度小于低门限时，不丢弃报文
> 	* 当队列的长度大于高门限时，丢弃所有收到的报文
> 	* 当队列的长度在低门限和高门限之间时，开始随机丢弃到来的报文
2. 引起TCP饿死现象：尾丢弃无法对流量进行区分丢弃，当TCP流量整体减小时，UDP流量不会减小，可能会导致UDP流量占满队列，造成TCP饿死
3. 无差别丢弃：尾丢弃无法对流量进行区分丢弃，尾丢弃可能会导致大量的非关键数据被转发，大量的关键数据被丢弃
> 缺点2、3的解决方法：WRED（Weighted Random Early Detection）加权随机早期检测，每一种优先级都能独立设置报文的丢包的高门限、低门限及丢包率，报文到达低门限时，开始丢包，到达高门限时丢弃所有的报文

## WRED配置

```bash
drop-profile manager  //创建模板-manager
wred dscp  //使用wred识别dscp的值
dscp 8 low-limit 50 high-limit 70 discard-percentage 10  //dscp的值为8时，低门限为50，高门限为70，丢弃最大概率为10%
drop-profile ftp
wred dscp
dscp 16 low-limit 70 high-limit 90 discard-percentage 10
drop-profile video
wred dscp
dscp 24 low-limit 60 high-limit 80 discard-percentage 20
# Qos队列和模板绑定
qos queue-profile qos-Huawei
queue 1 drop-profile manager
queue 2 drop-profile ftp
queue 3 drop-profile video
# 接口下应用Qos模板
interface E1
qos queue-profile qos-Huawei
```

# 流量监管与流量整形

## 令牌桶（CBS）

- TC代表令牌桶的令牌容量，即令牌桶中令牌的数量
- CBS承诺突发尺寸，即令牌桶的容量大小
- PBS峰值突发尺寸
- CIR承诺信息速率，每秒向令牌桶注入的令牌数量

> - **单数单桶**：
> 	* 收到流量后，流量与令牌桶中的TC比较，流量大于TC，无法通过，标记红色
> 	* 流量小于等于TC，TC的大小减去流量的大小，标记为绿色，通过
> - **单数双桶**：
> 	* CIR只向CBS令牌桶中注入令牌
> 	* 当一个CBS令牌桶令牌装满后，会向PBS令牌桶溢出
> 	* 收到一个流量A后，将流量A与CBS令牌桶内的TC比较，流量A小于TC，则TC减去流量A的大小，标记绿色，通过
> 	* 当流量A大于CBS中的TC，小于PBS中的TC时，PBS中的TC减去流量A的大小，标记黄色，通过
> 	* 当流量A大于PBS令牌桶中的TC时，直接标记红色，无法通过
> - **双数双桶**：
> 	* CIR会向CBS令牌桶注入令牌，PIR回向PBS令牌桶注入令牌
> 	* 收到一个流量B，先与令牌桶PBS比较，流量B大于PBS中的TC，则标记红色，无法通过
> 	* 当流量B小于PBS的TC时，在与CBS的TC比较，大于CBS的TC时，直接PBS的TC减去流量B的大小，标记黄色，通过
> 	* 当CBS的TC也大于流量B，则PBS的TC与CBS的TC同时都减去流量B的大小，标记绿色，通过

## 流量监管TP（Traffic Policing）

- 对接受或发送（进出口都可以使用）的流量进行限速控制，限制进入网络的突发流量，流量超速的部分直接丢掉
- 优点：可以实现对不同类别的报文进行限速
- 缺点：当链路空闲时，造成带宽浪费；丢弃的流量可能要进行重传

## 流量整形TS（Traffic Shaping）

- 限制流出某一网络的某一连接的正常流量与突发流量，流量超出速率的会被缓存下来，只能用在出接口
- 优点：可以实现对不同报文分别进行限速；缓冲机制可以减少带宽浪费，减少流量重传
- 缺点：可能会增加延迟

## 总结

| 限速类型 | 优点 | 缺点 |
|:---:|:---:|:---|
| 流量监管 | 可以实现对不同报文的限速及重标记 | 造成较高的丢包率；链路空闲时，带宽得不到充分利用 |
| 流量整形 | 较少丢包报文，充分利用带宽 | 引入额外的时延和抖动，需要较多的设备缓冲资源 |