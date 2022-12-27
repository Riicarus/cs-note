# 计网概述

## OSI 标准模型
- 应用层: 针对特定应用(SMTP, FTP, RPC)
- 表示层: 处理数据表示和运输相关问题, 将设备的固有数据转换为网络标准格式
- 会话层: 管理通信, 负责利用传输层提供的服务建立和维持会话
- 传输层: 管理两个节点之间的数据传输(TCP, UDP)
- 网络层: 负责地址管理和路由选择(IP)
- 数据链路层: 负责在单个链路上传输数据帧
- 物理层: 提供传输媒体和互联设备

## TCP/IP 协议簇
简化为四层模型
- 应用层(包括 OSI 的应用层, 表示层, 会话层)
- 传输层
- 网络层
- 通信链路层(包括 OSI 的数据链路层, 物理层)

协议簇包含的协议:
- IP 协议: Internet Protocol. 网络层, 为运输层提供数据分发, 组装数据供运输层使用
- ICMP 协议: Internet Control Message Protocol. 网络层, 在 IP 主机, 路由器之间传递控制消息(错误侦测与回报机制).
- ARP 协议: Address Resolution Protocol. IP 地址 --> 物理地址.
- TCP 协议: Transmission Control Protocol. 传输层, 可靠数据交互(慢启动, 拥塞控制, 快速重传, 可恢复).
- UDP 协议: User Datagram Protocol. 传输层, 无连接, 不可靠, 效率高.
- FTP 协议: File Transfer Protocol. 应用层, 文件传输协议, 服务器 + 客户端.
- DNS 协议: Domain Name System. 应用层, 域名 <--> IP. 分布式系统.
- SMTP 协议: Simple Mail Transfer Protocol. 应用层, 邮件服务.
- SLIP 协议: Serial Line Internet Protocol. 链路层, 点对点.
- PPP 协议: Point to Point Protocol. 链路层, 同等单元之间传输数据包, 点对点.

