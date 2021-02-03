# HCNA网络技术笔记

## 一. 网络通信基础

### 1. 一些标准机构

- ISO （International Organization for Standardization），国际标准化组织
- IETF（Internet Engineering Task Force），互联网工程任务组
- IEEE（Institute of Electrical and Electronics Engineers）,电气电子工程师学会
- ITU（International Telecommunications Union），国际电信联盟
- IEC（International Electrotechnical Commission），国际电工技术委员会

### 2. OSI-RM（Open Systems Interconnection Reference Model）

- 物理层： 完成逻辑"0"、"1"向物理信号的转换；物理信号的收发以及传输；
- 数据链路层：在物理链路相链接的相邻节点之间，建立逻辑意义上的数据链路，实现**点到点、点到多点的直接通信**；
- 网络层：实现数据**从任一节点到另一个节点的整个传输过程**；
- 传输层：建立、维护和取消**一次端到端的数据传输**过程；传输控制；
- 会话层：通信双方建立、管理、终止会话；
- 表述层：数据格式转换；
- 应用层：系统应用接口；

### 3. TCP/IP 协议簇

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

![](images/TCPIPPacketProgress.png)

### 4. 网络划分

#### LAN 和 WAN 

| 网络类型 | 特点 | 技术 | 
| --- | --- | --- | 
| LAN | 覆盖几公里以内</br> 分布较近的若干终端连接起来</br> 不涉及到电信运营商线路 | Token Bus</br> Token Ring</br> FDDI(Fiber-Distributed Data Interface)</br> Ethernet</br> WLAN | 
| WAN | 覆盖几十、几百、几千公里</br> 连接若干LAN</br> 用到电信运营商线路 | T1/E1、T1/E3</br> X.25</br> HDLC </br> PPP </br> ISDN </br> FR </br> ATM </br> SDH | 

### 5. 传输介质

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

### 6. 通信方式

串行/并行通信  
单工/半双工/全双工通信

## 二. VRP基础 

华为通用网络操作系统。 

## 三. 以太网

