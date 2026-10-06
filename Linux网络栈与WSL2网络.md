# Linux 网络栈与 WSL2 网络

在上一篇中，我们已经把 ROS2 的通信链拆到了：

```text
Node
↓
rclcpp / rclpy
↓
rcl
↓
RMW
↓
DDS
↓
Discovery
↓
QoS
↓
Data Transport
```

但是到这里还有一个非常重要的问题：

```text
DDS 最后怎么把数据真正发送出去？
```

例如：

```text
DDS DataWriter
```

已经准备好一条消息。

这条消息最终必须经过：

```text
操作系统
↓
网络协议
↓
网卡
↓
真实网络
```

才能到达另一台机器。

而当前环境又不是普通 Ubuntu 主机，而是：

```text
Windows 11
↓
WSL2
↓
Ubuntu 22.04
↓
ROS2 Humble
```

因此实际网络链路比：

```text
Ubuntu
↓
网卡
```

更加复杂。

本篇重点就是搞清楚：

```text
DDS
↓
Socket
↓
UDP / TCP
↓
IP
↓
Linux Network Stack
↓
WSL2 Network
↓
Windows Network
↓
Physical Network
```

这一整条链到底是什么逻辑。

---

# 1. 什么是网络栈

网络栈可以理解为：

```text
操作系统处理网络通信的一整套协议和机制
```

例如：

```text
Application
↓
Socket
↓
Transport Layer
↓
IP
↓
Network Interface
```

在 Linux 中，这整套机制通常可以称为：

```text
Linux Network Stack
```

---

# 2. ROS2 为什么最终一定会进入网络栈

ROS2 上层看到的是：

```text
Node
Topic
Publisher
Subscriber
```

DDS 看到的是：

```text
Participant
DataWriter
DataReader
```

但是网卡并不认识：

```text
ROS2 Topic
```

也不认识：

```text
DDS DataWriter
```

网卡最终处理的是：

```text
网络数据包
```

所以中间必须经过：

```text
DDS
↓
操作系统网络接口
↓
Linux 网络栈
```

完成真正的数据发送。

---

# 3. 完整通信链继续向下

目前可以把 ROS2 通信链继续补完整：

```text
ROS2 Application
↓
rclcpp / rclpy
↓
rcl
↓
RMW
↓
DDS
↓
Socket
↓
UDP / TCP
↓
IP
↓
Linux Network Stack
↓
Network Interface
↓
Physical Network
```

在当前 WSL2 环境里，还需要再增加：

```text
WSL2 Virtual Network
↓
Windows Network Stack
```

---

# 4. 什么是 Socket

Socket：

```text
套接字
```

可以简单理解为：

```text
应用程序访问操作系统网络功能的接口
```

也就是说：

```text
DDS
```

通常不会直接：

```text
控制物理网卡
```

而是通过操作系统提供的：

```text
Socket API
```

发送和接收网络数据。

---

# 5. Socket 在哪里

可以把它放到架构中：

```text
DDS
↓
Socket
↓
Linux Network Stack
```

Socket 是：

```text
用户空间程序
```

和：

```text
操作系统网络栈
```

之间的重要接口。

---

# 6. 一个简单类比

可以把：

```text
Socket
```

理解成：

```text
网络通信的“插座”
```

应用程序把数据交给 Socket：

```text
DDS
↓
Socket
```

然后操作系统负责：

```text
协议处理
路由
分包
网卡发送
```

---

# 7. UDP 和 TCP 在哪里

Socket 下面通常会选择：

```text
UDP
```

或者：

```text
TCP
```

进行通信。

它们都属于：

```text
Transport Layer
```

也就是：

```text
传输层
```

---

# 8. TCP 是什么

TCP 全称：

```text
Transmission Control Protocol
```

它的特点包括：

```text
面向连接
可靠传输
有序传输
丢包重传
流量控制
拥塞控制
```

可以简单理解成：

```text
TCP 更强调可靠
```

---

# 9. UDP 是什么

UDP 全称：

```text
User Datagram Protocol
```

它的特点更接近：

```text
无连接
开销较小
不保证送达
不保证顺序
不自动重传
```

可以简单理解成：

```text
UDP 更轻量
```

---

# 10. DDS 常见为什么会用 UDP

DDS / RTPS 在很多常见配置下会大量使用：

```text
UDP
```

其中一个重要原因是：

```text
Discovery
实时数据传输
Multicast
```

这些场景非常适合 UDP。

例如：

```text
Participant Discovery
```

常常需要：

```text
向网络中的多个设备宣布自己的存在
```

这时候 UDP：

```text
Multicast
```

就很有用。

---

# 11. 但不要简单理解成 DDS = UDP

这是一个重要误区。

不要记成：

```text
DDS
=
UDP
```

更准确应该理解：

```text
DDS
=
中间件通信体系
```

而：

```text
UDP / TCP
=
操作系统传输层协议
```

DDS Implementation 可以根据：

```text
配置
实现
传输插件
场景
```

选择不同传输方式。

---

# 12. IP 在哪里

UDP / TCP 下面通常是：

```text
IP
```

例如：

```text
IPv4
IPv6
```

IP 主要负责：

```text
地址
路由
数据包从哪里到哪里
```

例如：

```text
192.168.1.10
```

就是一个典型 IPv4 地址。

---

# 13. ROS_DOMAIN_ID 和 IP 不一样

这里特别容易混淆。

例如：

```text
ROS_DOMAIN_ID=10
```

和：

```text
192.168.1.10
```

完全不是同一个概念。

前者属于：

```text
ROS2 / DDS 逻辑通信域
```

后者属于：

```text
IP 网络地址
```

所以：

```text
ROS_DOMAIN_ID
≠
IP 地址
```

---

# 14. Linux Network Interface

Linux 中的数据最终要经过：

```text
Network Interface
```

也就是：

```text
网络接口
```

例如常见：

```text
eth0
lo
wlan0
```

不过在 WSL2 中看到的接口名称和传统物理 Ubuntu 主机可能不同。

---

# 15. lo 是什么

`lo`：

```text
Loopback
```

也就是：

```text
回环接口
```

常见地址：

```text
127.0.0.1
```

可以理解成：

```text
本机和本机通信
```

---

# 16. localhost 是什么

例如：

```text
localhost
```

通常解析到：

```text
127.0.0.1
```

它只表示：

```text
当前操作系统内部
```

所以：

```text
localhost
```

不是：

```text
局域网其他电脑
```

---

# 17. eth0 是什么

在 Linux 中：

```text
eth0
```

通常表示一个以太网类型网络接口。

在 WSL2 中也经常可以看到：

```text
eth0
```

但这里的：

```text
eth0
```

通常并不是：

```text
电脑真实物理网口
```

而更可能是：

```text
WSL2 虚拟网络环境中的接口
```

---

# 18. 查看 Linux 网络接口

执行：

```bash
ip addr
```

或者简写：

```bash
ip a
```

可以看到：

```text
lo
eth0
其他接口
```

以及对应：

```text
IP 地址
```

---

# 19. 查看 eth0

可以执行：

```bash
ip addr show eth0
```

查看：

```text
eth0 IP
网络掩码
接口状态
```

---

# 20. 查看路由表

执行：

```bash
ip route
```

可能看到：

```text
default via ...
```

以及：

```text
不同网段对应哪个接口
```

路由表决定：

```text
数据包应该往哪里发送
```

---

# 21. 一个普通 Ubuntu 主机的网络链路

如果 ROS2 直接运行在实体 Ubuntu 主机：

```text
ROS2
↓
DDS
↓
Socket
↓
UDP/IP
↓
Linux Network Stack
↓
Physical NIC
↓
Router / Switch
↓
另一台机器
```

这个结构比较直接。

---

# 22. 但当前环境是 WSL2

当前结构：

```text
Windows 11
↓
WSL2
↓
Ubuntu 22.04
↓
ROS2 Humble
```

因此：

```text
Ubuntu
```

并不是直接安装在裸机上。

它运行在：

```text
WSL2 虚拟化环境
```

中。

---

# 23. WSL2 本质上是什么

WSL2 和早期 WSL 的一个重要区别是：

```text
WSL2 使用真正的 Linux Kernel
```

并运行在：

```text
轻量级虚拟化环境
```

中。

所以 WSL2 Ubuntu 有：

```text
自己的 Linux Kernel
自己的 Linux Network Stack
自己的网络接口
```

---

# 24. 因此 Windows 和 WSL2 是两个网络环境

可以先粗略理解成：

```text
Windows
│
├── Windows Network Stack
│
└── Physical NIC
```

同时：

```text
WSL2 Ubuntu
│
├── Linux Network Stack
│
└── Virtual NIC
```

两者之间再通过：

```text
WSL2 网络机制
```

连接。

---

# 25. WSL2 网络结构

可以大致画成：

```text
ROS2
↓
DDS
↓
Linux Socket
↓
UDP / IP
↓
Linux Network Stack
↓
WSL2 Virtual NIC
↓
Windows / WSL2 Networking
↓
Windows Network Stack
↓
Physical NIC
↓
LAN
```

所以 WSL2 比普通 Ubuntu 多了一层：

```text
虚拟网络
```

---

# 26. 为什么这对 ROS2 很重要

普通：

```text
HTTP
SSH
```

很多时候只需要：

```text
单播连接
```

就能工作。

但是 DDS Discovery 常常涉及：

```text
UDP
Multicast
多个网络接口
自动发现
```

因此 WSL2 的网络模式会更加明显地影响 ROS2。

---

# 27. WSL2 常见网络模式

WSL2 的实际网络行为需要结合当前 Windows 和 WSL 配置判断。

常见可以遇到：

```text
NAT 网络模式
```

以及较新的：

```text
Mirrored Networking
```

所以不要默认：

```text
所有 WSL2 都一定使用完全相同的网络结构
```

---

# 28. NAT 模式是什么

在经典 WSL2 NAT 模式下，

可以粗略理解成：

```text
LAN
↓
Windows
↓
NAT
↓
WSL2 Virtual Network
↓
Ubuntu
```

这时 WSL2 Ubuntu 往往有：

```text
和 Windows 主机不同的虚拟 IP
```

---

# 29. NAT 对普通联网通常没有大问题

例如：

```bash
sudo apt update
```

Ubuntu 可以通过：

```text
WSL2
↓
Windows
↓
Internet
```

正常访问网络。

因为：

```text
向外主动连接
```

通常比较容易处理。

---

# 30. NAT 对 DDS Discovery 可能更复杂

DDS Discovery 不是简单：

```text
客户端主动连接固定服务器
```

而是：

```text
多个节点互相发现
```

这就可能涉及：

```text
Multicast
UDP
端口
接口选择
入站流量
```

在 NAT 环境下：

```text
这些行为可能比普通 HTTP 更复杂
```

---

# 31. Mirrored Networking 是什么

较新的 WSL2 可以支持：

```text
Mirrored Networking
```

它的目标之一是：

```text
让 WSL2 与 Windows 的网络行为更加接近
```

尤其改善：

```text
局域网访问
VPN
IPv6
Multicast
```

等一些网络场景。

---

# 32. 但不能只因为使用 Mirrored 就认为一定没问题

即使网络模式更适合：

```text
局域网通信
```

仍然可能受到：

```text
Windows Firewall
DDS 配置
RMW 配置
ROS_DOMAIN_ID
网络接口
VPN
```

等因素影响。

所以：

```text
网络模式
```

只是排障中的一个层次。

---

# 33. Windows 防火墙

在 WSL2 环境中，

还必须考虑：

```text
Windows Defender Firewall
```

因为最终网络通信可能需要经过：

```text
Windows 网络体系
```

如果防火墙阻止：

```text
UDP
Multicast
DDS 相关流量
```

就可能出现：

```text
WSL2 中 ROS2 本机正常
```

但是：

```text
另一台电脑发现不到它
```

---

# 34. 一个典型现象

假设：

WSL2 中运行：

```bash
ros2 run demo_nodes_cpp talker
```

同一个 WSL2 中运行：

```bash
ros2 run demo_nodes_py listener
```

正常。

但是另一台 NUC：

```text
看不到 WSL2 talker
```

这说明：

```text
ROS2 基本运行环境
```

大概率没有完全坏掉。

问题可能集中在：

```text
WSL2 网络
Windows 防火墙
DDS Discovery
Multicast
Domain
```

---

# 35. 本机通信为什么更容易成功

因为：

```text
talker
↓
listener
```

如果都在同一个 Ubuntu 中，

通信可能只需要：

```text
同一 Linux 网络环境
```

甚至某些通信可以走：

```text
localhost
```

或同机通信路径。

但是跨机器时：

```text
必须真正经过外部网络
```

所以问题会更多。

---

# 36. ROS_LOCALHOST_ONLY

前面已经遇到：

```bash
echo $ROS_LOCALHOST_ONLY
```

它会影响：

```text
ROS2 是否只关注本机通信
```

如果通信被限制在：

```text
localhost
```

那么：

```text
WSL2 内部 talker / listener
```

可能正常。

但是：

```text
另一台机器
```

无法发现。

---

# 37. ROS_DOMAIN_ID

同时检查：

```bash
echo $ROS_DOMAIN_ID
```

如果：

```text
WSL2
ROS_DOMAIN_ID=0
```

而：

```text
NUC
ROS_DOMAIN_ID=10
```

那么即使：

```text
网络完全正常
```

它们通常也不会处于同一个 ROS Graph。

---

# 38. RMW_IMPLEMENTATION

还需要检查：

```bash
echo $RMW_IMPLEMENTATION
```

不同系统可能使用：

```text
不同 RMW / DDS Implementation
```

虽然符合标准的实现可能具备互操作能力，

但是：

```text
配置
Discovery 行为
网络接口选择
```

仍可能存在差异。

---

# 39. 当前网络接口怎么检查

首先：

```bash
ip addr
```

观察：

```text
eth0
lo
IP 地址
```

然后：

```bash
ip route
```

观察：

```text
default route
```

---

# 40. Windows 端怎么查看网络

在 PowerShell 中：

```powershell
ipconfig
```

可以查看：

```text
Wi-Fi
Ethernet
WSL Virtual Adapter
其他虚拟网卡
```

等信息。

---

# 41. 为什么 Windows 可能有很多网卡

Windows 电脑上可能同时存在：

```text
Wi-Fi
Ethernet
WSL
Hyper-V
VPN
VMware
VirtualBox
Tailscale
ZeroTier
```

等接口。

DDS 进行：

```text
自动网络接口选择
```

时，

多个接口可能增加复杂性。

---

# 42. VPN 为什么可能影响 ROS2

VPN 软件可能：

```text
增加新的虚拟网卡
修改路由
修改防火墙
修改 Multicast 行为
```

因此会出现：

```text
昨天 ROS2 多机通信正常

今天打开 VPN 后失效
```

这种情况。

---

# 43. 为什么重启 WSL 有时有效

执行：

```powershell
wsl --shutdown
```

再重新进入 Ubuntu。

可能会重新初始化：

```text
WSL2 VM
虚拟网卡
IP
路由
网络状态
```

所以某些：

```text
WSL2 网络状态异常
```

可能因此恢复。

---

# 44. 但重启 WSL 不等于解决根因

如果问题来自：

```text
Firewall
DDS 配置
ROS_DOMAIN_ID
Multicast 被阻止
```

那么：

```text
wsl --shutdown
```

之后问题仍然可能重新出现。

所以仍然需要：

```text
分层排查
```

---

# 45. ping 能检查什么

例如：

```bash
ping 192.168.1.100
```

成功说明：

```text
IP 层基本连通
```

至少：

```text
ICMP
```

可以正常往返。

---

# 46. ping 不能证明什么

ping 成功不能证明：

```text
DDS Discovery 正常
```

因为 DDS 还可能依赖：

```text
UDP
Multicast
特定端口
QoS
Domain
RMW 配置
```

所以：

```text
ping 成功
≠
ROS2 一定可以通信
```

---

# 47. 反过来也一样

如果：

```text
ping 不通
```

那么应该优先解决：

```text
基础网络
```

因为：

```text
ROS2
```

不可能绕过：

```text
底层网络
```

凭空完成跨机器通信。

---

# 48. 一个非常重要的排障原则

可以把通信问题分成：

```text
ROS2 层
↓
DDS 层
↓
操作系统网络层
↓
物理网络层
```

如果：

```text
IP 都不通
```

就不要优先研究：

```text
QoS
```

因为问题发生得更底层。

---

# 49. Linux Socket 状态

可以使用：

```bash
ss
```

查看 Socket。

例如：

```bash
ss -a
```

或者：

```bash
ss -u
```

查看 UDP Socket。

---

# 50. ss 和 ros2 topic list 的区别

```bash
ros2 topic list
```

观察：

```text
ROS Graph
```

而：

```bash
ss
```

观察：

```text
操作系统 Socket
```

所以它们处在完全不同层。

---

# 51. 为什么分层工具很重要

例如：

```text
ROS Graph
```

用：

```bash
ros2 node list
ros2 topic list
```

检查。

RMW 环境：

```bash
echo $RMW_IMPLEMENTATION
```

检查。

Linux Network：

```bash
ip addr
ip route
ss
ping
```

检查。

Windows Network：

```powershell
ipconfig
```

检查。

这就是：

```text
一层对应一类工具
```

---

# 52. Linux 中的 DNS

网络栈中还有：

```text
DNS
```

它负责：

```text
域名
↓
IP 地址
```

例如：

```text
packages.ros.org
```

需要先解析成：

```text
IP
```

才能访问。

---

# 53. DNS 和 DDS Discovery 不一样

不要把：

```text
DNS
```

和：

```text
DDS Discovery
```

混淆。

DNS 解决：

```text
域名是什么 IP？
```

DDS Discovery 解决：

```text
网络中有哪些 DDS Participant / Endpoint？
```

两者完全不同。

---

# 54. Multicast 是什么

Multicast：

```text
组播
```

可以理解成：

```text
一份数据
↓
发送给某个组
↓
多个接收者都可以收到
```

这与：

```text
Unicast
```

不同。

---

# 55. Unicast

Unicast：

```text
单播
```

结构：

```text
A
↓
B
```

也就是：

```text
一个发送端
对应一个目标地址
```

---

# 56. Multicast

Multicast：

```text
A
↓
Multicast Group
├── B
├── C
└── D
```

因此特别适合：

```text
Discovery
```

这种：

```text
我还不知道所有人是谁
但我要告诉同组的人我存在
```

的场景。

---

# 57. 为什么 Wi-Fi 也可能影响 Multicast

有些：

```text
路由器
企业网络
校园网络
Wi-Fi AP
```

可能限制：

```text
Multicast
设备之间直接通信
```

例如开启：

```text
AP Isolation
Client Isolation
```

后，

两台设备虽然：

```text
都能上互联网
```

但可能：

```text
不能直接互相通信
```

---

# 58. 所以上网正常不代表 ROS2 多机通信正常

这是另一个经典误区。

例如：

```text
NUC 能上网
电脑能上网
```

并不能说明：

```text
NUC ↔ 电脑
```

之间的 DDS Discovery 一定正常。

真正需要确认：

```text
它们之间能否直接通信
```

---

# 59. 一个典型多机网络结构

例如：

```text
Wi-Fi Router
      │
      ├──────── Windows 11 Laptop
      │             │
      │             └── WSL2
      │                   └── ROS2
      │
      └──────── NUC
                    └── Ubuntu
                          └── ROS2
```

目标是：

```text
WSL2 ROS2
↕
NUC ROS2
```

互相发现。

---

# 60. 数据实际可能经过什么

从 WSL2 发往 NUC：

```text
ROS2 Node
↓
rclcpp
↓
rcl
↓
RMW
↓
DDS
↓
Linux Socket
↓
Linux Network Stack
↓
WSL2 Virtual NIC
↓
Windows Network
↓
Wi-Fi / Ethernet
↓
Router / Switch
↓
NUC NIC
↓
NUC Linux Network Stack
↓
DDS
↓
RMW
↓
ROS2 Node
```

这是非常重要的一张总链路图。

---

# 61. 为什么任何一层都可能出问题

例如：

```text
Application
```

可能 Topic 名写错。

```text
QoS
```

可能不兼容。

```text
DDS
```

可能 Discovery 失败。

```text
Linux
```

可能路由错误。

```text
WSL2
```

可能虚拟网络异常。

```text
Windows
```

可能 Firewall 阻止。

```text
Router
```

可能阻止 Multicast。

所以排障一定需要：

```text
逐层定位
```

---

# 62. 一个推荐的基础网络检查流程

多机 ROS2 通信失败时，

第一步先看 Linux 接口：

```bash
ip addr
```

第二步看路由：

```bash
ip route
```

第三步测试 IP：

```bash
ping <对方IP>
```

---

# 63. 然后检查 ROS2 环境

```bash
echo $ROS_DISTRO
```

```bash
echo $ROS_DOMAIN_ID
```

```bash
echo $ROS_LOCALHOST_ONLY
```

```bash
echo $RMW_IMPLEMENTATION
```

---

# 64. 然后测试最简单 ROS2 通信

一台机器：

```bash
ros2 run demo_nodes_cpp talker
```

另一台机器：

```bash
ros2 run demo_nodes_py listener
```

这样可以避免：

```text
先拿 FAST-LIO
PX4
Livox
```

这种复杂系统做网络测试。

---

# 65. 为什么要先用 demo_nodes

因为：

```text
FAST-LIO 不通信
```

可能同时涉及：

```text
Livox Driver
消息类型
QoS
参数
Topic Remap
算法
DDS
网络
```

变量太多。

而：

```text
demo_nodes_cpp talker
demo_nodes_py listener
```

更适合：

```text
隔离 ROS2 基础通信问题
```

---

# 66. 如果 demo_nodes 本机正常，跨机失败

说明：

```text
ROS2 安装基本正常
```

问题更可能集中在：

```text
DDS Discovery
网络接口
Firewall
WSL2
Multicast
Domain
```

---

# 67. 如果本机 demo_nodes 都失败

那就不要先排查：

```text
路由器
Wi-Fi
```

应该优先检查：

```text
ROS2 环境
daemon
RMW
DDS
本机网络
```

---

# 68. WSL2 与 localhost

还有一个非常容易混淆的问题：

```text
Windows localhost
```

和：

```text
WSL2 localhost
```

在很多现代 WSL 使用场景中存在便捷互通机制，

但从架构理解上仍然应该知道：

```text
Windows
```

和：

```text
WSL2 Linux
```

是两个不同的网络环境。

不要简单认为：

```text
所有 localhost 行为
都完全等价于原生单系统
```

---

# 69. 为什么端口转发对普通服务更容易理解

例如：

```text
FastAPI
```

监听：

```text
0.0.0.0:8000
```

本质上是：

```text
固定 TCP / UDP 端口服务
```

比较容易通过：

```text
地址 + 端口
```

理解。

DDS 则不同。

它还涉及：

```text
自动 Discovery
动态 Endpoint
Multicast
多个端口
```

因此比普通 Web Server 更复杂。

---

# 70. ROS2 不是简单的“开一个端口”

这是非常重要的理解。

不要把 ROS2 多机通信想成：

```text
打开 11311
就行
```

这更接近 ROS1 Master 的思维。

ROS2 DDS 通信涉及：

```text
Domain
Participant
Discovery
Endpoint
QoS
Transport
Network
```

所以网络逻辑更加分布式。

---

# 71. ROS1 网络与 ROS2 网络思维区别

ROS1：

```text
ROS_MASTER_URI
↓
找到 Master
↓
获取其他节点地址
↓
节点之间通信
```

ROS2：

```text
DDS Domain
↓
Discovery
↓
Participant / Endpoint
↓
QoS Matching
↓
Data Transport
```

因此 ROS2 网络排障重点也发生了变化。

---

# 72. ROS1 常见网络变量

ROS1 经常关注：

```text
ROS_MASTER_URI
ROS_IP
ROS_HOSTNAME
```

例如：

```bash
export ROS_MASTER_URI=http://192.168.1.100:11311
```

---

# 73. ROS2 更关注什么

ROS2 中更经常关注：

```text
ROS_DOMAIN_ID
ROS_LOCALHOST_ONLY
RMW_IMPLEMENTATION
DDS 配置
Network Interface
Multicast
Firewall
```

所以从 ROS1 迁移到 ROS2：

```text
网络排障思维也必须改变
```

---

# 74. 用无人机系统理解整个网络

假设：

```text
Laptop
Windows 11
↓
WSL2 ROS2
```

负责：

```text
算法开发
RViz
调试
```

无人机上的：

```text
NUC
Ubuntu
ROS2
```

负责：

```text
Livox
FAST-LIO
Planner
控制
```

那么数据：

```text
NUC FAST-LIO
↓
/Odometry
↓
DDS
↓
NUC Linux Network
↓
Wi-Fi
↓
Windows Network
↓
WSL2 Network
↓
DDS
↓
RViz
```

这就是：

```text
ROS2
+
Linux
+
WSL2
+
真实网络
```

共同完成通信。

---

# 75. 一个非常重要的结论

ROS2 通信问题不一定是：

```text
ROS2 的问题
```

可能实际上是：

```text
Linux 路由
```

或者：

```text
Windows 防火墙
```

或者：

```text
WSL2 网络模式
```

或者：

```text
Wi-Fi Multicast
```

所以：

```text
ROS2 排障必须理解网络栈
```

---

# 76. 把整个架构连接起来

现在已经可以画出：

```text
ROS2 Application
       │
       ▼
rclcpp / rclpy
       │
       ▼
      rcl
       │
       ▼
      RMW
       │
       ▼
      DDS
       │
       ├── Discovery
       ├── QoS
       └── Data Transport
               │
               ▼
             Socket
               │
               ▼
           UDP / TCP
               │
               ▼
              IP
               │
               ▼
      Linux Network Stack
               │
               ▼
       WSL2 Virtual NIC
               │
               ▼
     Windows Network Stack
               │
               ▼
         Physical NIC
               │
               ▼
          LAN / Router
```

这就是当前环境下：

```text
ROS2 从程序一路走到真实网络
```

的完整路径。

---

# 77. 每一层对应什么问题

可以直接记：

```text
Application
↓
Topic 名称、代码、参数
```

```text
rclcpp / rclpy
↓
Client Library
```

```text
rcl
↓
ROS2 Core
```

```text
RMW
↓
Middleware Adapter
```

```text
DDS
↓
Discovery / QoS / Transport
```

```text
Linux Network
↓
Socket / UDP / IP / Route
```

```text
WSL2
↓
Virtual Network
```

```text
Windows
↓
Firewall / Physical Network
```

---

# 78. Linux 常用网络检查命令

查看接口：

```bash
ip addr
```

查看路由：

```bash
ip route
```

测试 IP 连通：

```bash
ping <IP>
```

查看 Socket：

```bash
ss -a
```

查看 UDP：

```bash
ss -u
```

---

# 79. ROS2 常用网络相关检查命令

查看 Domain：

```bash
echo $ROS_DOMAIN_ID
```

查看 localhost 限制：

```bash
echo $ROS_LOCALHOST_ONLY
```

查看 RMW：

```bash
echo $RMW_IMPLEMENTATION
```

查看 ROS2：

```bash
echo $ROS_DISTRO
```

---

# 80. Windows 常用检查命令

PowerShell：

```powershell
ipconfig
```

查看 WSL：

```powershell
wsl -l -v
```

必要时重新启动 WSL：

```powershell
wsl --shutdown
```

然后重新进入：

```powershell
wsl -d Ubuntu-22.04
```

---

# 81. 推荐的跨机器 ROS2 排障顺序

不要一开始就：

```text
改 DDS XML
改 QoS
重装 ROS2
```

建议按照：

```text
第一层：
两台机器是否在网络上可达？
```

然后：

```text
第二层：
ROS_DOMAIN_ID 等环境是否一致？
```

然后：

```text
第三层：
DDS Discovery 是否成功？
```

再：

```text
第四层：
Topic / Endpoint 是否被发现？
```

最后：

```text
第五层：
QoS 和数据是否正常？
```

---

# 82. 一个完整排障树

可以整理成：

```text
跨机器 ROS2 不通信
│
├── IP 能互相访问吗？
│   │
│   ├── 不能
│   │   ↓
│   │   网络 / 路由 / 防火墙
│   │
│   └── 能
│       ↓
│
├── ROS_DOMAIN_ID 一致吗？
│
├── ROS_LOCALHOST_ONLY 是否限制？
│
├── RMW / DDS 是否正常？
│
├── Discovery 是否成功？
│
├── Node 能看到吗？
│
├── Topic 能看到吗？
│
├── QoS 兼容吗？
│
└── 数据真的在发送吗？
```

---

# 83. 本篇核心总结

第一：

```text
ROS2 数据最终必须进入操作系统网络栈
```

完整路径：

```text
DDS
↓
Socket
↓
UDP / TCP
↓
IP
↓
Linux Network Stack
```

第二：

当前不是普通 Linux 主机，而是：

```text
Windows 11
↓
WSL2
↓
Ubuntu
```

所以还需要经过：

```text
WSL2 Network
↓
Windows Network
```

第三：

```text
ping 成功
≠
DDS Discovery 一定成功
```

第四：

```text
本机 ROS2 正常
≠
跨机器 ROS2 一定正常
```

第五：

WSL2 环境中要特别关注：

```text
Network Mode
Windows Firewall
Multicast
Virtual NIC
VPN
```

---

# 84. 最重要的一条完整通信链

现在需要牢牢记住：

```text
ROS2 Node
↓
rclcpp / rclpy
↓
rcl
↓
RMW
↓
DDS
↓
Discovery / QoS
↓
Socket
↓
UDP / TCP
↓
IP
↓
Linux Network Stack
↓
WSL2 Virtual Network
↓
Windows Network Stack
↓
Physical Network
↓
另一台设备
```

这基本已经把：

```text
ROS2 软件架构
```

和：

```text
真实计算机网络
```

完整连接起来了。

---

# 85. 下一篇

到这里，我们已经从：

```text
Node
```

一路拆到了：

```text
真实网络
```

现在就可以真正开始回答：

```text
ROS2 出问题时
到底应该怎么判断是哪一层？
```

例如：

```text
ros2 node list 卡住怎么办？

ros2 topic list 卡住怎么办？

Topic 存在但 echo 没数据怎么办？

本机正常但多机无法通信怎么办？

daemon 应该什么时候重启？

什么时候应该检查 RMW？

什么时候应该检查 DDS？

什么时候应该检查 QoS？

什么时候应该检查 WSL2 网络？
```

下一篇将把前面所有知识真正用于：

```text
故障定位
```

下一篇继续拆解：

```text
11-ROS2故障排查.md
```
