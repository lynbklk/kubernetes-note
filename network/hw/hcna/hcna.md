# HCNA网络技术笔记

## 1. 网络通信基础

### 1.1 一些标准机构

- ISO （International Organization for Standardization），国际标准化组织
- IETF（Internet Engineering Task Force），互联网工程任务组
- IEEE（Institute of Electrical and Electronics Engineers）,电气电子工程师学会
- ITU（International Telecommunications Union），国际电信联盟
- IEC（International Electrotechnical Commission），国际电工技术委员会

### 1.2 OSI-RM（Open Systems Interconnection Reference Model）

- 物理层： 完成逻辑"0"、"1"向物理信号的转换；物理信号的收发以及传输；
- 数据链路层：在物理链路相链接的相邻节点之间，建立逻辑意义上的数据链路，实现**点到点、点到多点的直接通信**；
- 网络层：实现数据**从任一节点到另一个节点的整个传输过程**；
- 传输层：建立、维护和取消**一次端到端的数据传输**过程；传输控制；
- 会话层：通信双方建立、管理、终止会话；
- 表述层：数据格式转换；
- 应用层：系统应用接口；

### 1.3 TCP/IP 协议簇

- TCP：Transmission Control Protocol  
- IP：Internet Protocol

TCP/IP 模型与 OSI 模型主要区别在于二者使用的具体协议的不同。

TCP/IP 协议簇：  

| 序号 | 描述 | 协议 |  数据单元 | 
| --- | --- | --- | --- | 
| 5 | 应用层 | HTTP、 FTP、 SMTP、 SNMP | HTTP / FTP Datagram | 
| 4 | 传输层 | TCP、 UDP | TCP Segment / UDP Datagram | 
| 3 | 网络层 | IP、 ICMP、 IGMP | Packet | 
| 2 | 数据链路层 | SLIP、 PPP | Frame | 
| 1 | 物理层 | ... | Bit | 

OSI 协议簇：

| 序号 | 描述 | 协议 | 
| --- | --- | --- | 
| 7 | 应用层 | FTAM、 X.400、 CMIS | 
| 6 | 表示层 | X.266、 X.236 | 
| 5 | 会话层 | X.225、 X.235 | 
| 4 | 传输层 | TP0、 TP1、 TP2 | 
| 3 | 网络层 | CLNP、 X.233 | 
| 2 | 数据链路层 | ISO/IEC 766 | 
| 1 | 物理层 | EIA/TIA-232 | 

TCP/IP 模型数据封装过程：

![](images/hcna/TCPIPPacketProgress.png)

### 1.4 网络划分

#### LAN 和 WAN 

| 网络类型 | 特点 | 技术 | 
| --- | --- | --- | 
| LAN | 覆盖几公里以内</br> 分布较近的若干终端连接起来</br> 不涉及到电信运营商线路 | Token Bus</br> Token Ring</br> FDDI(Fiber-Distributed Data Interface)</br> Ethernet</br> WLAN | 
| WAN | 覆盖几十、几百、几千公里</br> 连接若干LAN</br> 用到电信运营商线路 | T1/E1、T1/E3</br> X.25</br> HDLC </br> PPP </br> ISDN </br> FR </br> ATM </br> SDH | 

### 1.5 传输介质

#### 分类 

| 类型 | 信号 | 特点 | 
| --- | --- | --- | 
| 空间 | 电磁信号 | 真空/空气</br> 光速/近光速 | 
| 金属导线 | 电流电压信号 | 铜线（同轴电缆/双绞线） </br> 近光速 | 
| 玻璃纤维 | 光信号 | 2/3 光速 | 

| 传输介质 | 说明 | 特点 | 
| --- | --- | --- | 
| 同轴电缆 | 从得到外依次：</br> 铜导线 </br> 绝缘塑料 </br> 铜网屏蔽层 </br> 塑料护套 | 以太网演化成星型网络后不再使用同轴电缆 | 
| 双绞线 | 分为:</br> 屏蔽双绞线（STP） </br> 无屏蔽双绞线（UTP）</br> 又分为：</br> 一类 ->  超五类双绞线 | 双绞方式的导线抗电磁干扰能力强 | 
| 光纤 | | 传输距离更远</br> 抗干扰能力更强|

### 1.6 通信方式

串行/并行通信  
单工/半双工/全双工通信

## 2. VRP基础 

华为通用网络操作系统。 

## 3. 以太网

网卡工作在 TCP/IP 模型的物理层和数据链路层

### 3.1 以太网卡

七个功能模块：

- CU: Control Unit
	* 计算机上CU处理流程：  
		* 得到经网络层处理后的数据 Packet 并封装成 Frame 传给 OB
		* 或者从 IB 读取 Frame 并进行分析和处理，丢弃或者组成 Packet 传递给网络层；
	* 交换机上CU处理流程：
		* 直接丢弃
		* 传递给某块网卡的 CU
		* 复制 n 份，传递给所有网卡的 CU
- OB: Output Buffer，将收到的 Frame 排成队列依次传递给 LC
- LC: Line Coder， 对 Frame 进行线路编码（0/1 -> 电流电压波形)
- TX: Transmitter, 发送物理信号
- RX: Receiver， 接收物理信号
- LD: Line Decoder， 对物理信号进行解码（电流电压波形 -> 0/1) 并以 Frame 为单位传递给 IB
- IB: Input Buffer，将收到的 Frame 排成队列依次传递给 CU

### 3.2 以太网帧

#### 3.2.1 MAC 地址

48 bits

- 单播地址：前3字节为 OUI（Organizationally-Unique Identifier），第一个字节最低位为 0
- 组播地址：第一个字节最后一位为 1
- 广播地址：所有位为1

#### 3.2.2 以太帧的格式

以太帧有两种格式：

- IEEE 802.3 格式  
![](images/hcna/8023Frame.png)
- Ethernet II 格式  
![](images/hcna/ethernetFrame.png)

目前的网络设备兼容两种格式的帧，绝大部分以太帧使用 Ethernet II 格式。

### 3.3 以太网交换机

#### 3.3.1 以太网交换机

转发数据的端口都是以太网口的交换机  

#### 3.3.2 以太网交换机对帧的转发操作

分为

- 泛洪（Flooding）：转发给除进端口之外的所有端口
- 转发（Forwarding）：转发给另一个端口
- 丢弃（Discarding）：丢弃

交换机转发原理：

- 单播帧，则查询 Mac 地址表
	- 查不到：泛洪操作
	- 查到： 比较 Mac 地址表中的端口号 和 入口端口号
		- 相同：丢弃
		- 不同：转发到 Mac 地址表中的端口 并从该端口发送出去
- 广播帧，直接进行泛洪操作
- 组播帧，较为复杂，HCNA 不做介绍

在转发过程中，进行 MAC 地址学习。

#### 3.3.3 MAC 地址表与其老化机制

MAC 地址表反映 MAC 地址和端口之间的映射关系，存在于交换机内存中，交换机掉电重启则 MAC 地址表清零。

通过一个老化时间进行 MAC 地址清理。

### 3.4 ARP

ARP: Address Resolution Protocol， 地址解析协议（网络层协议），根据已知的 IP 地址获得其对应的 MAC 地址。

源设备通过某种机制（例如 DNS ）获取目的设备 IP 地址， 再通过 ARP 获得目的设备 MAC 地址。

ARP 工作原理：  
![](https://github.com/lynbklk/kubernetes-note/blob/master/images/hcna/ARP.png)

ARP 缓存：每台主机收到 ARP 报文时，都会通过报文里的源 IP 和源 MAC 地址更新自己的 ARP 缓存，缓存条目有 180s 生存期。 主机需要向其他设备发送单播帧时先查 APR 缓存。

ARP 报文格式：  
![](images/hcna/ARPProtocol.png)

## 4. STP 协议

STP（Spanning Tree Protocol）是交换机运行的、用来解决交换网络中环路问题的数据链路层协议。

### 4.1 环路问题

环路网络的存在会引发以下现象：

- MAC 地址表翻摆：一个广播帧会在环路网络中双向旋转，导致 MAC 地址表来回翻转
- 广播风暴：广播帧在环路网络中不停的进行广播
- 多帧复制：某个单播帧沿着环路网络不同路径到达

一些概念：

- 桥：早期交换机又被称为 桥/网桥，沿用至今
- 桥的 MAC 地址： 桥有多个转发端口， 取编号最小的端口的 MAC 地址作为桥的 MAC 地址
- 桥 BID：8个字节，优先级（2字节） + MAC 地址
- 端口ID： 16比特，优先级 + 编号（4+12 或 8+8）

### 4.2 STP 树的生成

STP 协议在环路网络中生成一个无环工作拓扑（ STP 树）。STP 树中，任何一个节点到根节点的路径唯一且最优。

STP 生成过程：

- 选举根桥：通过不断交换BPDU（Bridge Protocol Data Unit），选举出 BID 最小的网桥作为根桥  
- 确定根端口：为保证非根桥设备到根桥设备路径唯一且最优，在非根桥设备上确定一个根端口，作为与根桥设备的交换端口
	- 根路径开销RPC（Root Path Cost）：某个端口到根桥所经链路的路径开销
	- 路径开销：与端口转发速率有关，速率越大，开销越小  
		- 10Mbit/s ：2000000
		- 100Mbit/s ：200000
		- 1Gbit/s ：20000
		- 10Gbit/s ：2000
	- RPC 相同的情况下：
		- 比较上行设备 BID，选择较小
		- 上行设备 BID 相同（两个端口连接同一个上行设备的不同端口）：
			- 比较端口ID，选择较小者
- 确定指定端口

	当一个网段有两条或两条以上路径通往根桥时（一个网段连接了不同的交换机，或者连接了同一台交换机的不同端口），需要指定一个端口作为该网段到根桥的唯一端口，判断逻辑与确定根端口的逻辑类似
	
	根桥上的所有端口均为指定端口
	
- 阻塞备用端口

	根端口和指定端口之外的端口为备用端口，STP 阻塞备用端口的用户数据帧。
	
### 4.3 STP 报文格式
### 4.4 STP 端口状态
### 4.5 STP 改进

## 5. VLAN

### 5.1 VLAN 的作用

（二层）广播域： 广播帧能够到达的整个范围

VLAN 将一个大的广播域拆分成多个广播域，以减小安全隐患和垃圾流量的问题

### 5.2 VLAN 基本原理

VLAN 在交换机上进行设置，将端口进行 VLAN 划分，一个端口可以同属于两个 VLAN

仅通过二层转发，并不能跨 VLAN 通信

### 5.3 802.1Q 帧格式

IEEE802.1Q 定义了关于支持 VLAN 的交换机的标准规范

一个端口同属于两个 VLAN 时，如何判断接收到的帧所属 VLAN ？传统的 EthernetII 帧并不能识别出所属 VLAN， 802.1Q 帧则可以：

![](image/8021QFrame.png)

| 字段 | 长度 | 名称 | 解释 | 
| --- | --- | --- | --- | 
| TPID | 2bit | Tag Protocol ID | 表示该帧是否带有Tag | 
| PRI | 3bit | Priority | 优先级，交换机阻塞时，先发送高优先级帧 | 
| CFI | 1bit | Canonical Format Indicator | - | 
| VID | 12bit| VLAN Identifier | VLAN ID 有效范围：1 - 4094| 

### 5.4 VLAN 类型

计算机发送 Untagged 帧到交换机时，交换机必须通过某种原则将帧划分到特定 VLAN 中，根据划分原则的不同，有不同的VLAN 类型。

#### 基于端口的 VLAN

端口划分到 VLAN，计算机从某个端口接入，即归属于该端口所属 VLAN

无特殊说明， 通常 VLAN 是基于端口的

#### 基于 MAC 地址的 VLAN

交换机维护 MAC 地址到 VLAN 的映射表

该方案复杂但灵活，计算机可以随便更改接入交换机的端口，但 MAC 地址容易伪造，安全性不高

#### 基于协议的 VLAN

根据帧类型划分 VLAN 归属， 通常也称三层 VLAN

### 5.5 链路类型和端口类型

在一个支持 VLAN 的交互网络中：

Access Link： 计算机与交换机直接相连的链路  
Access Port： Access Link 中交换机一侧的端口  
Trunk Link： 交换机之间直接相连的链路  
Trunk Port： Trunk Link 两侧的端口  

AccessLink 上传输 Untagged 帧，只能属于某个 VLAN  
TrunkLink 上传输 tagged 帧，可以属于不同 VLAN  
注意：每个端口（Access Port 或 Trunk Port）都要设置一个 PVID（默认为 1），以界定 Untagged 帧归属  
实际使用过程中还定义一种 
不通类型端口转发规则不一样

### 5.6 VLAN 转发示例

### 5.7 VLAN 配置示例

### 5.8 GVRP

动态 VLAN

### 5.9 GVRP 配置示例

## 6 IP 基础

### 6.1 有类编址



