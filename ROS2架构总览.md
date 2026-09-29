# ROS2 架构总览

本文基于以下环境理解 ROS2：

```text
Windows 11
└── WSL2
    └── Ubuntu 22.04
        └── ROS2 Humble
```

学习 ROS2 时，最重要的不是单独记住 `Node`、`Topic`、`DDS`、`RMW` 这些名词，而是理解它们处在整个系统中的什么位置，以及数据是如何从上到下流动的。

---

# 1. ROS2 整体架构

可以先记住下面这条主线：

```text
Windows 11
└── WSL2
    └── Ubuntu 22.04
        └── ROS2 Application
            ├── Node
            ├── Publisher
            ├── Subscriber
            ├── Service
            ├── Action
            │
            ├── rclcpp / rclpy
            │
            ├── rcl
            │
            ├── RMW
            │
            ├── DDS
            │
            └── Linux 网络栈
                └── WSL2 虚拟网络
                    └── Windows 网络
```

可以把 ROS2 理解成一个分层系统：

```text
应用层
↓
ROS2 API 层
↓
ROS2 核心层
↓
中间件适配层
↓
DDS 通信层
↓
操作系统网络层
```

---

# 2. 操作系统与运行环境

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

ROS2 实际运行在：

```text
Ubuntu 22.04
```

中。

但是 Ubuntu 又运行在：

```text
WSL2
```

中。

因此 ROS2 的通信问题有时不一定只来自 ROS2 本身，还可能与下面这些部分有关：

```text
DDS
Linux 网络
WSL2 网络
Windows 防火墙
Windows 网卡
```

因此后续排查 ROS2 网络问题时，需要有分层思维。

---

# 3. ROS2 Application

最上层就是我们真正编写的 ROS2 程序。

例如：

```text
激光雷达节点
相机节点
飞控节点
定位节点
导航节点
```

这些程序在 ROS2 中通常以：

```text
Node
```

的形式存在。

一个 Node 可以拥有：

```text
Publisher
Subscriber
Service
Client
Action Server
Action Client
Timer
Parameter
```

例如：

```text
lidar_node
├── 发布 /scan
├── 发布 /points
├── 提供参数
└── 提供 Service
```

---

# 4. Node

Node 是 ROS2 中最基础的运行单元。

例如：

```text
/talker
/listener
/lidar_node
/camera_node
```

查看当前节点：

```bash
ros2 node list
```

查看某个节点的信息：

```bash
ros2 node info /talker
```

一个 Node 内部通常会创建多个通信实体。

例如：

```text
Node
├── Publisher
├── Subscriber
├── Service
├── Client
├── Timer
└── Parameter
```

---

# 5. Topic

Topic 用于：

```text
连续的数据流通信
```

典型模式：

```text
Publisher
↓
Topic
↓
Subscriber
```

例如：

```text
/talker
   │
   │ publish
   ▼
/chatter
   │
   │ subscribe
   ▼
/listener
```

查看 Topic：

```bash
ros2 topic list
```

查看 Topic 数据：

```bash
ros2 topic echo /chatter
```

查看 Topic 信息：

```bash
ros2 topic info /chatter
```

---

# 6. Service

Service 是一种：

```text
请求
↓
响应
```

形式的通信方式。

结构：

```text
Client
↓ Request
Service Server
↓ Response
Client
```

更适合：

```text
请求一次
得到一次结果
```

例如：

```text
打开设备
关闭设备
查询状态
修改配置
```

查看 Service：

```bash
ros2 service list
```

---

# 7. Action

Action 适合执行：

```text
持续一段时间的任务
```

例如：

```text
导航到目标点
机械臂运动
无人机执行航点任务
```

Action 可以提供：

```text
Goal
Feedback
Result
```

结构可以理解为：

```text
Action Client
↓ Goal
Action Server
↓ Feedback
Action Client
↓ Result
```

查看：

```bash
ros2 action list
```

---

# 8. ROS Graph

当多个 ROS2 Node 启动以后，它们之间会形成：

```text
ROS Graph
```

ROS Graph 可以理解为：

```text
当前 ROS2 系统中所有节点以及通信关系的总图
```

里面包括：

```text
Node
Publisher
Subscriber
Topic
Service
Client
Action
```

例如：

```text
/talker
   │
   │ Publisher
   ▼
/chatter
   │
   │ Subscriber
   ▼
/listener
```

这就是一个最简单的 ROS Graph。

---

# 9. ros2 CLI

我们经常执行：

```bash
ros2 node list
```

```bash
ros2 topic list
```

```bash
ros2 service list
```

这些都属于：

```text
ros2 CLI
```

也就是：

```text
ROS2 Command Line Interface
```

它的主要作用是：

```text
查看 ROS2 系统
调试 ROS2 系统
操作 ROS2 系统
```

注意：

```text
ros2 CLI
```

本身不是节点间真正的数据通信层。

它只是一个管理和观察工具。

---

# 10. daemon

ROS2 CLI 背后还有一个比较容易忽略的组件：

```text
ros2 daemon
```

daemon 是一个后台进程。

它主要帮助：

```text
ros2 node list
ros2 topic list
ros2 service list
```

等命令快速获取：

```text
ROS Graph
```

信息。

可以理解为：

```text
ros2 CLI
↓
daemon
↓
ROS Graph 信息
```

查看 daemon：

```bash
ros2 daemon status
```

停止：

```bash
ros2 daemon stop
```

启动：

```bash
ros2 daemon start
```

如果 daemon 异常，也可以：

```bash
pkill -f _ros2_daemon
```

然后重新启动：

```bash
ros2 daemon start
```

需要注意：

```text
daemon ≠ ROS2 数据通信核心
```

它主要服务于 CLI。

---

# 11. rclcpp 与 rclpy

ROS2 给不同编程语言提供不同的 Client Library。

C++ 使用：

```text
rclcpp
```

Python 使用：

```text
rclpy
```

例如 C++：

```cpp
rclcpp::Node
```

Python：

```python
rclpy.node.Node
```

它们提供程序员最常使用的高级接口：

```text
Node
Publisher
Subscriber
Service
Action
Timer
Parameter
Executor
```

可以理解为：

```text
程序员直接使用的 ROS2 API 层
```

---

# 12. rcl

在：

```text
rclcpp
```

和：

```text
rclpy
```

下面还有：

```text
rcl
```

完整关系：

```text
C++
↓
rclcpp
↓
rcl
```

以及：

```text
Python
↓
rclpy
↓
rcl
```

因此不同语言最终都会逐渐进入统一的：

```text
rcl
```

核心层。

可以将 rcl 理解为：

```text
ROS2 Client Library 的公共核心层
```

---

# 13. RMW

RMW 全称：

```text
ROS Middleware Interface
```

位置：

```text
rcl
↓
RMW
↓
DDS
```

RMW 的作用非常重要。

ROS2 不希望自己的上层代码直接绑定某一个具体 DDS。

因此设计成：

```text
ROS2
↓
RMW
↓
DDS
```

RMW 可以理解为：

```text
适配层
接口层
翻译层
```

它负责将 ROS2 的操作转换成底层中间件能够理解的操作。

例如：

```text
创建 Publisher
创建 Subscriber
发送消息
接收消息
发现其他节点
处理 QoS
```

---

# 14. DDS

DDS 全称：

```text
Data Distribution Service
```

DDS 是 ROS2 底层通信体系中非常重要的一部分。

它主要负责：

```text
Discovery
Topic Matching
Data Transport
QoS
```

也就是：

```text
发现
匹配
传输
通信规则
```

---

# 15. Discovery

Discovery 就是：

```text
发现
```

例如：

```text
Node A
发布 /chatter
```

Node B：

```text
订阅 /chatter
```

Node A 和 Node B 首先需要知道：

```text
对方存在
```

然后 DDS 才能进一步建立通信关系。

这与 ROS1 有很大的区别。

ROS1 中：

```text
Node
↓
ROS Master
↓
寻找其他 Node
```

ROS2 中：

```text
Node
↓
DDS Discovery
↓
自动发现其他参与者
```

因此 ROS2 不再依赖一个中心式 ROS Master。

---

# 16. Topic Matching

即使两个节点发现了彼此，也不代表一定可以通信。

DDS 还需要判断：

```text
Topic 名称
消息类型
QoS
```

是否可以匹配。

例如：

```text
Publisher
Topic: /chatter
Type: std_msgs/msg/String
```

Subscriber：

```text
Topic: /chatter
Type: std_msgs/msg/String
```

如果条件匹配，就可以建立通信。

---

# 17. QoS

QoS：

```text
Quality of Service
```

它决定：

```text
Publisher 和 Subscriber 应该按照什么通信规则交换数据
```

例如：

```text
Reliable
Best Effort
Keep Last
Keep All
Depth
Durability
```

QoS 在整个架构中的位置可以理解为：

```text
ROS2 Application
↓
rclcpp / rclpy
↓
QoS 设置
↓
rcl
↓
RMW
↓
DDS QoS
↓
实际通信
```

所以：

```text
QoS 是 ROS2 对通信需求的描述
```

而：

```text
DDS 负责真正执行这些通信规则
```

---

# 18. Linux 网络栈

DDS 最终必须依赖操作系统进行实际的数据传输。

例如：

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
网卡
```

在 WSL2 中还要继续向下：

```text
Ubuntu
↓
WSL2 虚拟网卡
↓
Windows 网络
↓
物理网卡
```

因此完整链路是：

```text
ROS2
↓
DDS
↓
Linux 网络栈
↓
WSL2 网络
↓
Windows 网络
↓
物理网络
```

---

# 19. 一条 ROS2 消息到底是怎么走的

假设：

```text
/talker
```

发布：

```text
/chatter
```

完整数据链路可以理解为：

```text
/talker
↓
rclcpp
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
Linux 网络栈
↓
WSL2 网络
↓
Windows 网络
↓
目标设备
↓
DDS
↓
RMW
↓
rcl
↓
rclcpp / rclpy
↓
/listener
```

这条链非常重要。

以后很多 ROS2 通信问题，都可以沿着这条链逐层排查。

---

# 20. ros2 node list 是怎么工作的

输入：

```bash
ros2 node list
```

可以粗略理解为：

```text
用户输入命令
↓
ros2 CLI
↓
daemon
↓
ROS Graph
↓
RMW
↓
DDS Discovery
↓
获取节点信息
↓
返回 CLI
```

因此：

```bash
ros2 node list
```

卡住时，不一定是：

```text
ROS2 完全坏了
```

可能出问题的是：

```text
CLI
daemon
ROS Graph
RMW
DDS Discovery
Linux 网络
```

中的某一层。

---

# 21. ROS2 架构总图

可以最终总结成：

```text
Windows 11
│
└── WSL2
    │
    └── Ubuntu 22.04
        │
        ├── ROS2 Application
        │   ├── Node
        │   ├── Publisher
        │   ├── Subscriber
        │   ├── Service
        │   └── Action
        │
        ├── ROS Graph
        │
        ├── ros2 CLI
        │   └── daemon
        │
        ├── rclcpp / rclpy
        │
        ├── rcl
        │
        ├── RMW
        │
        ├── DDS
        │   ├── Discovery
        │   ├── Topic Matching
        │   ├── QoS
        │   └── Data Transport
        │
        ├── Linux Network Stack
        │   ├── Socket
        │   ├── UDP / TCP
        │   ├── IP
        │   └── eth0 / lo
        │
        └── WSL2 Virtual Network
            │
            └── Windows Network
                │
                └── Physical Network
```

---

# 22. 最重要的理解

ROS2 并不是一个单独的软件程序。

更准确地说：

```text
ROS2 是一整套分层的软件系统
```

从上到下依次涉及：

```text
机器人应用
↓
ROS2 API
↓
ROS2 核心
↓
Middleware
↓
DDS
↓
操作系统网络
↓
真实网络
```

因此以后排查问题时，最重要的思维方式是：

```text
先判断问题发生在哪一层
↓
再针对这一层继续排查
```

而不是：

```text
一遇到问题
↓
重装 ROS2
```

---

# 23. 后续学习顺序

建议继续按照下面顺序学习：

```text
1. Node
2. Topic
3. Service
4. Action
5. ROS Graph
6. ros2 CLI
7. daemon
8. rclcpp / rclpy
9. rcl
10. RMW
11. DDS
12. Discovery
13. QoS
14. Linux 网络栈
15. WSL2 网络
16. ROS2 通信故障排查
```

这样可以逐步建立完整的 ROS2 通信体系。
