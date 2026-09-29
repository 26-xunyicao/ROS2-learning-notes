# ROS1 与 ROS2 对比

在学习 ROS2 之前，先理解它和 ROS1 的区别非常重要。

ROS2 并不是简单的：


ROS1 升级版


更准确地说，ROS2 对通信架构、中间件、网络模型、实时性、安全性、多机通信等方面都进行了重新设计。

---

# 1. 整体架构对比

ROS1 可以简单理解为：

```text
ROS1 Application
↓
roscpp / rospy
↓
ROS Master
↓
TCPROS / UDPROS
↓
Linux 网络
```

ROS2 更接近：

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
Linux 网络
```

最核心的变化之一就是：

```text
ROS1：
依赖 ROS Master

ROS2：
不再依赖中心式 ROS Master
```

---

# 2. ROS Master 对比

## ROS1

ROS1 中存在：

```text
roscore
```

其中包含非常重要的：

```text
ROS Master
```

节点需要通过 ROS Master 获取其他节点的信息。

例如：

```text
Node A
↓
ROS Master
↓
Node B
```

可以理解为：

```text
ROS Master 负责“介绍双方认识”
```

---

## ROS2

ROS2 中没有 ROS1 那种必须存在的中心式 ROS Master。

节点通过 DDS 的：

```text
Discovery
```

机制自动发现彼此。

结构更接近：

```text
Node A
←→ DDS Discovery ←→
Node B
```

因此：

```text
ROS2 不需要先启动 roscore
```

这也是为什么 ROS2 中直接运行：

```bash
ros2 run demo_nodes_cpp talker
```

就可以启动节点。

---

# 3. 节点发现机制对比

ROS1：

```text
Node
↓
ROS Master
↓
注册自己
↓
查询其他节点
↓
建立通信
```

ROS2：

```text
Node
↓
DDS Participant
↓
Discovery
↓
自动发现其他节点
↓
自动匹配通信实体
```

所以 ROS2 更强调：

```text
去中心化
```

---

# 4. 通信中间件对比

ROS1 主要使用：

```text
TCPROS
UDPROS
```

也就是 ROS 自己定义的一套通信机制。

ROS2 则增加了：

```text
RMW
```

和：

```text
DDS
```

完整结构：

```text
ROS2
↓
RMW
↓
DDS
↓
网络
```

这使 ROS2 可以使用不同的中间件实现。

---

# 5. RMW 是 ROS2 新的重要层

ROS1 中没有和 ROS2 中完全对应的 RMW 层。

ROS2 加入：

```text
RMW
```

全称：

```text
ROS Middleware Interface
```

它的作用是：

```text
让 ROS2 上层
不直接绑定某一种 DDS 实现
```

结构：

```text
rclcpp / rclpy
↓
rcl
↓
RMW
↓
DDS
```

所以 ROS2 的底层通信更容易替换和扩展。

---

# 6. DDS 是 ROS2 的核心变化之一

DDS：

```text
Data Distribution Service
```

在 ROS2 中，它主要负责：

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
通信策略
```

ROS1 没有这一整套 DDS 架构。

---

# 7. QoS 对比

ROS1 的通信策略相对固定。

例如：

```text
TCPROS
```

更偏向可靠传输。

ROS2 则引入了更系统的：

```text
QoS
```

全称：

```text
Quality of Service
```

它可以控制：

```text
Reliability
History
Depth
Durability
Deadline
Lifespan
```

例如：

```text
Reliable
Best Effort
```

这对于机器人中的不同数据非常重要。

比如：

```text
控制指令
激光雷达
相机
地图
状态信息
```

它们并不一定应该使用相同的通信策略。

---

# 8. Topic 通信对比

ROS1：

```text
Publisher
↓
ROS Master 辅助发现
↓
Subscriber
↓
TCPROS / UDPROS
```

ROS2：

```text
Publisher
↓
RMW
↓
DDS
↓
QoS 匹配
↓
Subscriber
```

因此 ROS2 中会出现一种很典型的情况：

```text
能看到 Topic
但是收不到数据
```

这时候除了网络问题，还要考虑：

```text
QoS 是否兼容
```

---

# 9. 命令行工具对比

ROS1 常见命令：

```bash
rosnode list
```

```bash
rostopic list
```

```bash
rosservice list
```

```bash
rosparam list
```

ROS2 则统一成：

```bash
ros2 node list
```

```bash
ros2 topic list
```

```bash
ros2 service list
```

```bash
ros2 param list
```

结构变得更加统一：

```text
ros2 + 功能模块 + 子命令
```

例如：

```bash
ros2 node list
ros2 node info
ros2 topic list
ros2 topic echo
```

---

# 10. roscore 与 ros2 daemon 的区别

这是非常容易混淆的一点。

ROS1：

```text
roscore
```

是系统中的核心基础组件。

它包含：

```text
ROS Master
Parameter Server
rosout
```

很多 ROS1 节点正常工作依赖它。

---

ROS2 中的：

```text
ros2 daemon
```

不是 ROS1 `roscore` 的替代品。

daemon 主要服务于：

```text
ros2 CLI
```

例如：

```bash
ros2 node list
```

```bash
ros2 topic list
```

它会帮助缓存和获取：

```text
ROS Graph
```

信息。

所以：

```text
roscore
≠
ros2 daemon
```

这是两套完全不同的角色。

---

# 11. 参数系统对比

ROS1 中有：

```text
Parameter Server
```

参数集中存放在 ROS Master 体系中。

ROS2 中参数更多是：

```text
Node 自己管理自己的参数
```

结构更接近：

```text
Node A
└── Parameters

Node B
└── Parameters
```

因此 ROS2 的参数系统更加节点化。

---

# 12. 实时性对比

ROS1 在设计之初，并不是以：

```text
实时系统
```

为主要目标。

ROS2 从架构上更加重视：

```text
Real-Time
```

包括：

```text
DDS QoS
Executor
内存管理
线程模型
中间件
```

因此 ROS2 更适合：

```text
工业机器人
自动驾驶
无人机
实时控制
```

等更复杂场景。

---

# 13. 多机通信对比

ROS1 多机通信通常需要重点处理：

```text
ROS_MASTER_URI
ROS_IP
ROS_HOSTNAME
```

例如：

```bash
export ROS_MASTER_URI=http://192.168.1.10:11311
```

ROS2 不依赖 ROS Master。

更多依赖：

```text
DDS Discovery
ROS_DOMAIN_ID
网络接口
组播
防火墙
QoS
```

例如：

```bash
echo $ROS_DOMAIN_ID
```

所以 ROS2 多机通信的问题排查思路也发生了变化。

---

# 14. 网络排障思路对比

ROS1 常见思路：

```text
roscore 是否运行
↓
ROS_MASTER_URI 是否正确
↓
ROS_IP / ROS_HOSTNAME
↓
网络是否互通
```

ROS2 更接近：

```text
ROS_DOMAIN_ID
↓
ROS_LOCALHOST_ONLY
↓
RMW
↓
DDS Discovery
↓
QoS
↓
Linux 网络
↓
防火墙 / WSL2 网络
```

因此 ROS2 网络排障通常比 ROS1 更强调：

```text
分层排查
```

---

# 15. ROS Graph 对比

ROS1 和 ROS2 都可以理解为存在：

```text
ROS Graph
```

也就是节点和通信关系组成的图。

但底层建立 Graph 信息的方式不同。

ROS1：

```text
ROS Master
↓
维护节点注册信息
```

ROS2：

```text
DDS Discovery
↓
分布式发现通信实体
```

所以 ROS2 的 Graph 更偏向：

```text
分布式生成
```

---

# 16. 编程接口对比

ROS1：

```text
C++ → roscpp
Python → rospy
```

ROS2：

```text
C++ → rclcpp
Python → rclpy
```

ROS2 中又增加了：

```text
rcl
```

作为更加统一的核心层。

所以：

```text
rclcpp
   ↓
  rcl
   ↑
rclpy
```

这是 ROS2 架构更加分层化的体现。

---

# 17. 生命周期节点

ROS2 支持：

```text
Lifecycle Node
```

节点可以拥有明确状态。

例如：

```text
Unconfigured
↓
Inactive
↓
Active
↓
Finalized
```

这对于复杂系统非常有用。

例如：

```text
传感器初始化
设备启动
系统切换
安全关闭
```

都可以有明确生命周期管理。

---

# 18. ROS1 与 ROS2 核心对比表

| 项目 | ROS1 | ROS2 |
|---|---|---|
| 中心节点 | ROS Master | 无中心 Master |
| 启动核心 | `roscore` | 不需要 `roscore` |
| 节点发现 | ROS Master | DDS Discovery |
| C++ API | roscpp | rclcpp |
| Python API | rospy | rclpy |
| 核心 Client 层 | 无完全对应层 | rcl |
| 中间件抽象 | 较弱 | RMW |
| 底层通信 | TCPROS / UDPROS | DDS 等中间件 |
| QoS | 较简单 | 完整 QoS 体系 |
| 参数 | Parameter Server | Node 自主管理 |
| 多机通信 | ROS_MASTER_URI 等 | DDS Discovery |
| CLI | rosnode / rostopic | ros2 node / ros2 topic |
| 实时性 | 支持有限 | 设计上更重视 |
| 生命周期 | 基础机制较少 | Lifecycle Node |
| 安全机制 | 较有限 | DDS / SROS2 等体系 |
| 分布式能力 | 依赖 Master | 更去中心化 |

---

# 19. 最直观的架构对比

## ROS1

```text
Application
│
├── roscpp
├── rospy
│
└── ROS Master
    │
    ├── Node Discovery
    ├── Parameter Server
    └── Registration
         │
         ▼
   TCPROS / UDPROS
         │
         ▼
   Linux Network
```

---

## ROS2

```text
Application
│
├── rclcpp
├── rclpy
│
└── rcl
     │
     ▼
    RMW
     │
     ▼
    DDS
     │
     ├── Discovery
     ├── Topic Matching
     ├── QoS
     └── Data Transport
          │
          ▼
    Linux Network
```

---

# 20. 从 ROS1 迁移思维到 ROS2

如果以前习惯 ROS1，可以建立下面的对应关系：

```text
roscpp
→
rclcpp
```

```text
rospy
→
rclpy
```

```text
rosnode
→
ros2 node
```

```text
rostopic
→
ros2 topic
```

```text
rosservice
→
ros2 service
```

但有一些内容不能简单一一对应：

```text
ROS Master
≠
ros2 daemon
```

ROS2 中 ROS Master 的功能实际上被：

```text
DDS Discovery
+
ROS2 分布式架构
```

重新设计了。

---

# 21. 最关键的区别

如果只记几个最重要的区别，可以记：

```text
ROS1：
ROS Master
+
TCPROS / UDPROS
+
中心式发现
```

ROS2：

```text
DDS
+
RMW
+
QoS
+
分布式 Discovery
```

因此 ROS2 可以简单理解为：

```text
从“ROS 自己管理通信”

变成

“ROS2 上层抽象 + RMW + DDS 分布式通信”
```

---

# 22. 为什么后面要学习 RMW、DDS 和 QoS

在 ROS1 中，学习到：

```text
Node
Topic
Service
ROS Master
```

基本已经可以理解大部分常见通信。

但 ROS2 中，仅仅理解：

```text
Node
Topic
Service
```

还不够。

还需要继续理解：

```text
RMW
DDS
Discovery
QoS
Linux 网络
```

否则当出现：

```text
ros2 node list 卡住
```

或者：

```text
Topic 能看到但收不到消息
```

或者：

```text
两台机器互相发现不到
```

时，就很难判断问题到底发生在哪一层。

---

# 23. 后续学习路线

理解 ROS1 与 ROS2 的区别以后，再继续学习：

```text
ROS2 整体架构
↓
Node / Topic / Service / Action
↓
ROS Graph
↓
ros2 CLI
↓
daemon
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
Linux 网络栈
↓
WSL2 网络
↓
ROS2 故障排查
```

这样整个 ROS2 的知识体系会更加清晰。
