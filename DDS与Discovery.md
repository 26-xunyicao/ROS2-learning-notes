# DDS 与 Discovery

在上一篇中，我们已经把 ROS2 的通信链继续向下拆到了：

```text
Application
↓
rclcpp / rclpy
↓
rcl
↓
RMW
```

并且知道：

```text
RMW
=
ROS2 与底层 Middleware 之间的抽象接口层
```

RMW 再向下，就会进入真正的通信中间件。

在典型的 ROS2 DDS 架构中，这一层就是：

```text
DDS
```

这里就会出现几个非常关键的问题：

```text
DDS 到底是什么？

为什么 ROS2 不继续使用 ROS1 的 Master？

DDS Discovery 是怎么发现其他节点的？

Participant、DataWriter、DataReader 是什么？

Publisher 和 Subscriber 到底是怎么找到彼此的？

为什么 ROS2 可以没有 roscore？
```

本篇重点就是搞清楚：

```text
RMW
↓
DDS
↓
Discovery
↓
Endpoint Matching
↓
Data Transport
```

这一整套通信逻辑。

---

# 1. DDS 是什么

DDS 全称：

```text
Data Distribution Service
```

可以理解为：

```text
面向数据的分布式通信中间件
```

它的目标并不是：

```text
让程序直接调用另一个程序
```

而更强调：

```text
数据是什么
↓
谁在发布这种数据
↓
谁需要这种数据
↓
按照什么规则传输
```

所以 DDS 是一种：

```text
Data-Centric
```

也就是：

```text
以数据为中心
```

的通信体系。

---

# 2. DDS 在 ROS2 中的位置

我们目前的完整通信链：

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
Operating System
↓
Network
```

因此 DDS 已经非常接近：

```text
真正的数据传输层
```

ROS2 上层描述：

```text
Node
Topic
Publisher
Subscriber
QoS
```

RMW 将这些 ROS2 概念映射到底层 Middleware。

然后 DDS 负责：

```text
发现
匹配
通信
QoS
数据传输
```

---

# 3. 为什么 ROS2 要使用 DDS

ROS1 的设计核心之一是：

```text
ROS Master
```

所有节点先去：

```text
注册
查询
发现
```

结构：

```text
Node A
↓
ROS Master
↑
Node B
```

这种架构比较直观。

但是 ROS2 面向的场景更加复杂，例如：

```text
机器人
无人机
自动驾驶
工业控制
多机系统
实时系统
复杂网络
```

这些系统更希望拥有：

```text
分布式发现
QoS
可扩展通信
实时通信能力
减少中心节点依赖
```

DDS 本身就具备很多这样的机制。

---

# 4. ROS1 的发现方式

ROS1 中：

```text
Node A
↓
ROS Master
```

Node A 告诉 Master：

```text
我存在
```

以及：

```text
我发布哪些 Topic
```

Node B 也向：

```text
ROS Master
```

注册。

如果 Node B 想找：

```text
/chatter
```

对应 Publisher，

可以通过 Master 获取相关信息。

所以：

```text
ROS Master
```

在节点发现过程中非常重要。

---

# 5. ROS2 的发现方式

ROS2 不再依赖一个中心：

```text
ROS Master
```

而是使用分布式发现机制。

可以粗略理解为：

```text
Node A
   ↘
    DDS Discovery
   ↗
Node B
```

每个通信参与者都能够：

```text
宣布自己的存在
发现其他参与者
发现对方的通信端点
判断能否通信
```

因此：

```text
ROS2 不需要中心式 Master
```

---

# 6. DDS Discovery 是什么

Discovery：

```text
发现
```

DDS Discovery 解决的是：

```text
网络中还有谁？

对方提供什么数据？

对方需要什么数据？

我们能不能建立通信？
```

例如：

```text
Node A
```

发布：

```text
/chatter
```

Node B：

```text
订阅 /chatter
```

首先双方要能够：

```text
发现彼此
```

然后才能进一步判断：

```text
Topic 是否匹配
消息类型是否匹配
QoS 是否兼容
```

---

# 7. Discovery 不是数据本身

这里一定要区分：

```text
Discovery
```

和：

```text
Data Transport
```

Discovery 主要解决：

```text
找到对方
```

真正开始通信以后：

```text
数据传输
```

是另外一个过程。

可以理解成：

```text
Discovery
↓
我知道你存在
↓
Endpoint Matching
↓
确认可以通信
↓
Data Transport
↓
真正传输消息
```

---

# 8. 一个生活中的类比

可以把 Discovery 理解成：

```text
通讯录 / 自我介绍
```

例如一个房间里有很多人。

A 说：

```text
我是 A
我可以提供激光雷达数据
```

B 说：

```text
我是 B
我需要激光雷达数据
```

双方发现需求匹配以后：

```text
建立联系
```

然后才真正开始：

```text
发送数据
```

---

# 9. DDS Domain

DDS 中还有一个非常重要的概念：

```text
Domain
```

可以把一个 Domain 理解成：

```text
一个逻辑上的通信空间
```

只有处于合适通信范围中的 Participant，

才能正常发现和通信。

在 ROS2 中，我们经常通过：

```text
ROS_DOMAIN_ID
```

控制这一部分。

---

# 10. ROS_DOMAIN_ID

查看：

```bash
echo $ROS_DOMAIN_ID
```

例如：

```text
Machine A
ROS_DOMAIN_ID=0
```

Machine B：

```text
ROS_DOMAIN_ID=0
```

它们可能处于同一个 ROS2 DDS Domain。

如果：

```text
Machine A
ROS_DOMAIN_ID=0
```

而：

```text
Machine B
ROS_DOMAIN_ID=10
```

那么它们通常不会按照普通 ROS2 Discovery 方式互相发现。

---

# 11. 为什么需要 Domain

假设同一个实验室有：

```text
无人机 A
无人机 B
无人机 C
机器人 D
机器人 E
```

如果所有设备：

```text
全部自动发现全部设备
```

可能会造成：

```text
Topic 混乱
节点冲突
不必要的 Discovery
系统难以隔离
```

因此可以使用：

```text
ROS_DOMAIN_ID
```

进行逻辑隔离。

例如：

```text
无人机系统 A
ROS_DOMAIN_ID=1
```

```text
无人机系统 B
ROS_DOMAIN_ID=2
```

---

# 12. Domain 不等于 IP 网段

注意：

```text
DDS Domain
```

不是：

```text
IP 子网
```

它属于：

```text
DDS 的逻辑通信域
```

而：

```text
192.168.1.x
```

这种属于：

```text
IP 网络层
```

所以：

```text
DDS Domain
≠
IP Subnet
```

这是两个不同层次的概念。

---

# 13. DomainParticipant

DDS 中非常重要的实体叫：

```text
DomainParticipant
```

可以先简单理解为：

```text
一个 DDS 通信域中的参与者
```

它是 DDS 实体组织结构中的重要入口。

粗略表示：

```text
DDS Domain
│
├── DomainParticipant A
│
├── DomainParticipant B
│
└── DomainParticipant C
```

Participant 之间：

```text
互相发现
```

然后进一步发现各自的通信 Endpoint。

---

# 14. ROS2 Node 和 DomainParticipant 是不是一一对应

不要简单记成：

```text
一个 ROS2 Node
=
一个 DDS DomainParticipant
```

ROS2 上层的：

```text
Node
```

和底层 DDS：

```text
DomainParticipant
```

属于不同抽象层。

它们之间存在映射关系，

但具体组织方式取决于：

```text
RMW Implementation
Middleware Implementation
实现策略
```

因此更安全的理解方式是：

```text
ROS2 Node
↓
RMW
↓
使用 DDS 实体参与通信
```

而不是强行做：

```text
Node = Participant
```

的一一对应。

---

# 15. DDS 中还有哪些重要实体

DDS 中经常会出现：

```text
DomainParticipant
Publisher
Subscriber
Topic
DataWriter
DataReader
```

其中最值得我们先理解的是：

```text
Topic
DataWriter
DataReader
```

---

# 16. DDS Topic

DDS 也有：

```text
Topic
```

它表示：

```text
某种数据主题
```

例如 ROS2 上层：

```text
/chatter
```

经过 RMW 映射到底层 Middleware 后，

会形成对应的 DDS 通信实体和 Topic 信息。

---

# 17. DataWriter

DDS 中：

```text
DataWriter
```

可以简单理解为：

```text
真正负责写数据的发送端实体
```

ROS2 上层：

```text
Publisher
```

向下经过：

```text
rcl
↓
RMW
```

以后，

通常会对应到底层 DDS 的：

```text
DataWriter
```

一类实体。

---

# 18. DataReader

DDS 中：

```text
DataReader
```

可以简单理解为：

```text
真正负责读取数据的接收端实体
```

ROS2 上层：

```text
Subscriber
```

向下经过：

```text
rcl
↓
RMW
```

以后，

通常会对应到底层：

```text
DataReader
```

一类实体。

---

# 19. ROS2 与 DDS 的概念映射

可以粗略理解为：

```text
ROS2 Publisher
↓
RMW
↓
DDS DataWriter
```

以及：

```text
ROS2 Subscriber
↓
RMW
↓
DDS DataReader
```

因此真正底层的数据通信可以看成：

```text
DataWriter
↓
Network
↓
DataReader
```

---

# 20. DDS Publisher 和 ROS2 Publisher 不要混淆

DDS 自己也有一个实体叫：

```text
Publisher
```

ROS2 上层也有：

```text
Publisher
```

这两个名字非常容易造成混淆。

ROS2 中我们平时说：

```text
Publisher
```

通常指：

```text
ROS2 Publisher
```

而在 DDS 内部：

```text
DDS Publisher
```

是 DDS 实体层次中的另一个概念。

真正负责写数据的 Endpoint 通常是：

```text
DataWriter
```

---

# 21. DDS Subscriber 也一样

DDS 自己也有：

```text
Subscriber
```

而真正负责读取样本数据的实体是：

```text
DataReader
```

所以可以粗略记住：

```text
ROS2 Publisher
≈
DDS DataWriter 一侧
```

```text
ROS2 Subscriber
≈
DDS DataReader 一侧
```

这里用：

```text
≈
```

而不是：

```text
=
```

因为中间还有：

```text
RMW
类型支持
DDS 实体组织
具体实现
```

---

# 22. Endpoint 是什么

DDS 中：

```text
DataWriter
DataReader
```

通常可以看作：

```text
通信 Endpoint
```

也就是：

```text
通信端点
```

Discovery 不只需要发现：

```text
Participant
```

还需要进一步发现：

```text
有哪些 DataWriter

有哪些 DataReader

它们对应什么 Topic

使用什么 QoS
```

---

# 23. Discovery 可以分成两个理解阶段

为了学习，可以把 Discovery 粗略分成：

```text
第一阶段：
发现 Participant
```

然后：

```text
第二阶段：
发现通信 Endpoint
```

也就是：

```text
先发现“谁在线”
↓
再发现“谁发布什么 / 谁订阅什么”
```

---

# 24. Participant Discovery

假设网络中出现：

```text
Participant A
Participant B
```

首先需要完成：

```text
Participant Discovery
```

可以简单理解成：

```text
A：我在这里

B：我也在这里
```

于是：

```text
A 知道 B 存在

B 知道 A 存在
```

---

# 25. Endpoint Discovery

发现 Participant 之后，

还需要知道：

```text
A 有哪些 DataWriter？

A 有哪些 DataReader？

B 有哪些 DataWriter？

B 有哪些 DataReader？
```

以及：

```text
它们对应哪些 Topic？
```

这就是 Endpoint Discovery 要解决的问题之一。

---

# 26. RTPS

在 DDS 通信中还经常会看到：

```text
RTPS
```

全称：

```text
Real-Time Publish-Subscribe
```

它定义了 DDS 实现之间进行：

```text
发现
数据交换
```

所使用的一套互操作协议机制。

因此学习 ROS2 网络时经常会看到：

```text
DDS
RTPS
```

一起出现。

---

# 27. DDS 和 RTPS 的关系

可以先简单理解：

```text
DDS
=
更高层的数据分发模型和 API / QoS 体系
```

而：

```text
RTPS
=
DDS 实现之间在网络上互操作的重要协议
```

所以：

```text
DDS
```

和：

```text
RTPS
```

不是完全同一个概念。

---

# 28. SPDP

如果进一步深入 RTPS Discovery，

会遇到：

```text
SPDP
```

全称：

```text
Simple Participant Discovery Protocol
```

它主要用于：

```text
发现 Participant
```

可以简单理解：

```text
谁在线？
```

---

# 29. SEDP

还有：

```text
SEDP
```

全称：

```text
Simple Endpoint Discovery Protocol
```

它主要用于：

```text
发现 Endpoint
```

也就是：

```text
谁有 DataWriter？

谁有 DataReader？

Topic 是什么？

QoS 是什么？
```

---

# 30. Discovery 的完整粗略逻辑

可以整理成：

```text
Participant A 启动
↓
Participant B 启动
↓
SPDP
↓
互相发现 Participant
↓
SEDP
↓
交换 Endpoint 信息
↓
发现 DataWriter / DataReader
↓
检查 Topic / Type / QoS
↓
匹配
↓
建立数据通信
```

这张链路非常重要。

---

# 31. ROS2 为什么不用 Master 也能找到 Topic

现在就可以回答：

```text
ROS2 没有 ROS Master

那 ros2 topic list
怎么知道有哪些 Topic？
```

因为底层：

```text
DDS Discovery
```

会发现通信实体。

然后：

```text
DDS Discovery
↓
RMW
↓
ROS Graph
↓
daemon
↓
ros2 CLI
```

最终：

```bash
ros2 topic list
```

就可以显示当前 Graph 中的 Topic。

---

# 32. ROS2 Node Discovery 完整链路

可以粗略表示：

```text
Node A
↓
rclcpp / rclpy
↓
rcl
↓
RMW
↓
DDS Participant / Endpoint
↓
Discovery
↓
网络中的其他 DDS Participant
↓
RMW
↓
ROS Graph
```

因此：

```text
ROS Graph
```

并不是靠：

```text
中央数据库
```

生成。

而是：

```text
分布式发现
```

得到的。

---

# 33. Publisher 和 Subscriber 怎么匹配

假设 Node A：

```text
Publisher
Topic = /chatter
```

Node B：

```text
Subscriber
Topic = /chatter
```

发现之后还要检查：

```text
Topic
Type
QoS
```

等条件。

只有满足通信条件，

双方才会建立匹配关系。

---

# 34. Topic 名称必须匹配

例如 Publisher：

```text
/chatter
```

Subscriber：

```text
/chatter
```

Topic 名称一致。

如果 Subscriber 订阅：

```text
/chat
```

那么它们不是同一个 Topic。

因此：

```text
Publisher
```

不会因为：

```text
名字很像
```

就自动发送给 Subscriber。

---

# 35. 消息类型也需要匹配

例如：

Publisher：

```text
Topic:
/chatter

Type:
std_msgs/msg/String
```

Subscriber：

```text
Topic:
/chatter

Type:
std_msgs/msg/String
```

可以继续匹配。

但如果：

```text
Topic 名称相同
```

消息类型却不符合通信要求，

就不能按照正常方式建立相同 Topic 的数据通信。

---

# 36. QoS 也会影响匹配

即使：

```text
Topic 名称一致
```

并且：

```text
消息类型一致
```

还有一个非常重要的条件：

```text
QoS
```

例如：

```text
Reliability
Durability
```

等策略之间可能存在兼容性要求。

所以最终匹配更接近：

```text
Discovery
↓
Topic
↓
Type
↓
QoS Compatibility
↓
Match
```

---

# 37. 这解释了一个经典现象

有时候执行：

```bash
ros2 topic list
```

能够看到：

```text
/sensor_data
```

但是：

```bash
ros2 topic echo /sensor_data
```

却没有数据。

这说明：

```text
Topic 被发现
```

不一定意味着：

```text
Subscriber 已经成功收到数据
```

还要继续检查：

```text
Publisher
Subscriber
QoS
数据是否真正发布
网络
```

---

# 38. Discovery 成功也不等于数据一定正常

可以记住：

```text
Discovery Success
≠
Data Communication Success
```

Discovery 只说明：

```text
我知道你存在
```

真正通信还需要：

```text
Endpoint 匹配
QoS
数据发送
网络传输
接收处理
```

全部正常。

---

# 39. 一个推荐检查命令

查看 Topic：

```bash
ros2 topic list
```

查看详细信息：

```bash
ros2 topic info /topic_name -v
```

例如：

```bash
ros2 topic info /chatter -v
```

可以观察：

```text
Publisher count
Subscription count
Node name
QoS profile
```

这些信息对理解 DDS Endpoint 匹配非常有帮助。

---

# 40. Discovery 和 Multicast

DDS Discovery 在很多常见配置下会使用：

```text
UDP
```

并可能利用：

```text
Multicast
```

进行 Participant Discovery。

Multicast 可以简单理解成：

```text
向一个组发送数据
```

而不是：

```text
逐个提前知道所有对方的 IP
```

这对于：

```text
自动发现
```

非常方便。

---

# 41. 为什么网络会影响 Discovery

如果：

```text
Multicast 被阻止
```

或者：

```text
防火墙阻止相关流量
```

或者：

```text
虚拟网络环境无法正确传递 Discovery 数据
```

那么可能出现：

```text
两个设备 IP 可以互相 ping
```

但是：

```text
ROS2 Node 互相发现不到
```

这是 ROS2 多机通信排障中非常重要的现象。

---

# 42. ping 通不代表 ROS2 Discovery 一定正常

例如：

```bash
ping 192.168.1.100
```

成功，

只能说明：

```text
基础 IP 网络具有一定连通性
```

但 ROS2 Discovery 还可能涉及：

```text
UDP
Multicast
DDS 配置
Firewall
RMW
ROS_DOMAIN_ID
```

所以：

```text
Ping Success
≠
DDS Discovery Success
```

---

# 43. 这对 WSL2 特别重要

当前环境：

```text
Windows 11
↓
WSL2
↓
Ubuntu 22.04
↓
ROS2 Humble
```

如果 ROS2 只在：

```text
同一个 WSL2 Ubuntu 内部
```

运行，

通常问题相对简单。

但如果需要：

```text
WSL2 ROS2
↕
另一台电脑
```

或者：

```text
WSL2
↕
无人机 NUC
```

就会涉及：

```text
WSL2 虚拟网络
Windows 网络
防火墙
Multicast
DDS Discovery
```

等更多层次。

---

# 44. 为什么 WSL2 会让 ROS2 网络更复杂

因为不是简单的：

```text
Ubuntu
↓
物理网卡
```

而可能是：

```text
Ubuntu
↓
WSL2 Virtual Network
↓
Windows Network Stack
↓
Physical NIC
```

DDS Discovery 数据需要经过：

```text
更多网络层
```

因此出现问题时，

不能只检查：

```text
ROS2 Node
```

---

# 45. ROS_DOMAIN_ID 也会导致发现不到

假设：

Computer A：

```bash
export ROS_DOMAIN_ID=0
```

Computer B：

```bash
export ROS_DOMAIN_ID=10
```

那么即使：

```text
网络完全正常
```

双方也通常不会：

```text
出现在同一个 ROS Graph
```

所以多机通信时要优先检查：

```bash
echo $ROS_DOMAIN_ID
```

---

# 46. ROS_LOCALHOST_ONLY

还要检查：

```bash
echo $ROS_LOCALHOST_ONLY
```

如果 ROS2 被限制：

```text
只进行本机通信
```

那么本机：

```text
talker
↕
listener
```

可能正常。

但：

```text
Computer A
↕
Computer B
```

可能发现不到。

---

# 47. 这形成一个典型排障现象

例如：

```text
本机 talker / listener 正常
```

但是：

```text
两台电脑的 ROS2 Node 看不到彼此
```

这时候问题更可能在：

```text
ROS_DOMAIN_ID
ROS_LOCALHOST_ONLY
DDS Discovery
Multicast
Firewall
Network Interface
WSL2 Network
```

而不是：

```text
rclcpp 程序本身
```

---

# 48. DDS Discovery 与 daemon 的关系

前面学习了：

```text
ros2 daemon
```

现在可以把整个关系连起来：

```text
DDS Discovery
↓
RMW
↓
ROS Graph
↓
daemon
↓
ros2 CLI
```

所以执行：

```bash
ros2 node list
```

背后并不是：

```text
daemon 自己发现节点
```

真正底层发现能力来自：

```text
DDS / Middleware Discovery
```

daemon 主要：

```text
观察
维护
缓存
```

Graph 信息。

---

# 49. DDS Discovery 与 RMW 的关系

DDS 产生的是：

```text
Middleware 层信息
```

ROS2 上层需要的是：

```text
ROS Graph 信息
```

中间由：

```text
RMW
```

完成重要映射。

所以：

```text
DDS Discovery
↓
RMW
↓
ROS Graph
```

这条关系非常重要。

---

# 50. DDS Discovery 与 QoS 的关系

Discovery 期间不仅需要知道：

```text
Endpoint 是否存在
```

还需要交换：

```text
通信属性
```

其中就包括：

```text
QoS
```

然后判断：

```text
DataWriter
```

和：

```text
DataReader
```

是否兼容。

因此 QoS 不只是：

```text
数据发送以后再决定
```

它还会影响：

```text
Endpoint Matching
```

---

# 51. 为什么下一篇必须讲 QoS

因为现在我们已经知道：

```text
Participant 被发现
↓
Endpoint 被发现
↓
Topic / Type 检查
↓
QoS 检查
↓
匹配
```

所以：

```text
QoS
```

实际上决定：

```text
双方应该按照什么规则通信
```

甚至可能影响：

```text
双方能不能成功匹配
```

---

# 52. 用激光雷达理解 Discovery

假设：

```text
Livox Driver
```

发布：

```text
/livox/lidar
```

FAST-LIO 订阅：

```text
/livox/lidar
```

首先：

```text
Livox 相关 DDS Participant
↓
Discovery
↓
FAST-LIO 一侧发现相关 Endpoint
```

然后：

```text
Topic 匹配
↓
Type 匹配
↓
QoS 匹配
```

最后：

```text
DataWriter
↓
PointCloud / CustomMsg
↓
DataReader
```

真正开始传输。

---

# 53. 用无人机系统理解 DDS

假设无人机系统：

```text
LiDAR
FAST-LIO
Planner
Controller
PX4 Bridge
```

这些 Node 可能形成：

```text
livox_node
↓
/livox/lidar
↓
fastlio_node
↓
/odometry
↓
planner_node
↓
trajectory
↓
controller_node
↓
px4_bridge
```

表面看：

```text
都是 ROS2 Topic
```

向下看：

```text
ROS2 Publisher / Subscriber
↓
RMW
↓
DDS DataWriter / DataReader
↓
DDS Discovery
↓
Network
```

---

# 54. ROS2 去中心化真正发生在哪里

现在可以更准确地回答：

```text
ROS2 为什么没有 ROS Master？
```

不是因为：

```text
ROS2 完全不需要发现机制
```

而是因为：

```text
发现机制从中心式 Master
```

变成了：

```text
Middleware / DDS 分布式 Discovery
```

也就是说：

```text
中心发现
↓
分布式发现
```

---

# 55. ROS1 与 ROS2 再对比

ROS1：

```text
Node A
↓
ROS Master
↑
Node B
```

ROS Master 负责：

```text
注册
查询
介绍通信双方
```

ROS2：

```text
Node A
↓
RMW
↓
DDS Discovery
↕
DDS Discovery
↑
RMW
↑
Node B
```

没有必须存在的中心 Master。

---

# 56. DDS 不等于 ROS2

这里还要避免一个误区：

```text
DDS
=
ROS2
```

这是错误的。

ROS2 包含：

```text
Node
Topic
Service
Action
Parameters
Client Libraries
rcl
RMW
CLI
Graph
Launch
Tools
```

等等。

DDS 只是：

```text
ROS2 Middleware 层常用的重要基础
```

---

# 57. ROS2 也不等于 DDS API

普通 ROS2 开发者通常不会直接写：

```text
DDS DataWriter API
DDS DataReader API
```

而是写：

```cpp
create_publisher()
```

```cpp
create_subscription()
```

中间经过：

```text
rclcpp / rclpy
↓
rcl
↓
RMW
↓
DDS
```

所以 DDS 的复杂性被 ROS2 上层封装起来。

---

# 58. 从上往下看一次

假设：

```text
lidar_node
```

发布一帧点云。

链路：

```text
Application
↓
rclcpp
↓
rcl
↓
RMW
↓
DDS DataWriter
↓
Linux Network
↓
DDS DataReader
↓
RMW
↓
rcl
↓
rclcpp
↓
FAST-LIO
```

这就是 ROS2 Topic 数据通信的核心主链之一。

---

# 59. Discovery 在这条链中的位置

通信开始之前：

```text
Participant Discovery
↓
Endpoint Discovery
↓
Topic / Type / QoS Matching
```

匹配完成以后：

```text
DataWriter
↓
Data Transport
↓
DataReader
```

所以完整逻辑：

```text
先发现
↓
再匹配
↓
再通信
```

---

# 60. 一个必须记住的三阶段模型

学习 DDS 时，可以先牢牢记住：

```text
第一阶段：
Discovery
```

解决：

```text
谁存在？
```

第二阶段：

```text
Matching
```

解决：

```text
我们能不能通信？
```

第三阶段：

```text
Data Transport
```

解决：

```text
真正怎么传数据？
```

---

# 61. ROS2 Discovery 排障思路

如果：

```bash
ros2 node list
```

看不到另一台机器，

优先考虑：

```text
Discovery
```

可以依次检查：

```text
ROS_DOMAIN_ID
↓
ROS_LOCALHOST_ONLY
↓
RMW
↓
DDS
↓
Network Interface
↓
Firewall
↓
Multicast
↓
WSL2 Network
```

---

# 62. 如果 Node 能看到但 Topic 没数据

如果：

```bash
ros2 node list
```

能够看到对方 Node，

说明：

```text
至少部分 Discovery
```

已经工作。

然后：

```bash
ros2 topic list
```

也可以看到 Topic，

但：

```bash
ros2 topic echo /topic_name
```

收不到数据，

此时更应该继续检查：

```text
Publisher 是否真的发布
Subscriber 是否存在
QoS
消息类型
网络数据传输
```

---

# 63. 分层排障再次体现价值

可以整理：

```text
完全看不到 Node
↓
优先查 Discovery
```

```text
能看到 Node
但看不到 Topic
↓
查 Endpoint / Graph
```

```text
能看到 Topic
但没有数据
↓
查 QoS / Publisher / Transport
```

这种：

```text
按通信阶段定位问题
```

比直接重装 ROS2 更有效。

---

# 64. DDS 在整个 ROS2 架构中的位置

现在完整架构已经可以画到：

```text
ROS2 Application
│
├── Node
├── Topic
├── Service
└── Action
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
       ├── DomainParticipant
       ├── Topic
       ├── DataWriter
       ├── DataReader
       ├── Discovery
       ├── Matching
       └── Data Transport
               │
               ▼
       Linux Network Stack
```

---

# 65. 本篇核心总结

第一：

```text
DDS
=
Data Distribution Service
```

是一套：

```text
面向数据的分布式通信中间件体系
```

第二：

```text
ROS2 不再依赖 ROS Master
```

而是通过：

```text
DDS / Middleware Discovery
```

实现分布式发现。

第三：

```text
Discovery
```

可以先粗略理解成：

```text
Participant Discovery
+
Endpoint Discovery
```

第四：

```text
ROS2 Publisher
```

向底层通常会映射到：

```text
DDS DataWriter 一侧
```

而 ROS2 Subscriber：

```text
DDS DataReader 一侧
```

第五：

```text
发现成功
≠
数据通信一定成功
```

真正通信还需要：

```text
Topic
Type
QoS
Matching
Network
```

全部满足要求。

---

# 66. 最重要的一条链

现在需要牢牢记住：

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
Endpoint Matching
↓
Data Transport
↓
Linux Network
```

这已经把：

```text
ROS2 应用层
```

一路连接到了：

```text
真正的分布式通信层
```

---

# 67. 下一篇

现在已经知道：

```text
Participant 被发现
↓
Endpoint 被发现
↓
Topic / Type 匹配
```

但还剩下一个非常关键的问题：

```text
双方到底应该按照什么规则通信？
```

例如：

```text
消息必须保证送达吗？

允许丢数据吗？

Subscriber 启动得晚，
还能收到之前的数据吗？

保存多少条历史消息？

为什么激光雷达经常使用 Best Effort？

为什么 QoS 不兼容时，
Topic 明明存在却收不到数据？
```

这些问题全部指向：

```text
QoS
```

下一篇继续拆解：

```text
09-QoS.md
```
