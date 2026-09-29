# ROS Graph 与 ros2 CLI

在上一篇中，我们已经认识了 ROS2 中最基本的几个通信元素：

```text
Node
Topic
Service
Action
```

但是当多个 Node 同时运行以后，会产生一个新的问题：

```text
ROS2 怎么知道当前系统里有哪些 Node？

怎么知道哪个 Node 发布了哪些 Topic？

又怎么知道哪些 Node 正在订阅这些 Topic？
```

这就需要理解：

```text
ROS Graph
```

同时，我们平时使用的：

```bash
ros2 node list
ros2 topic list
ros2 service list
ros2 action list
```

这些命令属于：

```text
ros2 CLI
```

本篇重点就是搞清楚：

```text
Node / Topic / Service / Action
        ↓
     ROS Graph
        ↓
     ros2 CLI
```

三者之间到底是什么关系。

---

# 1. 什么是 ROS Graph

ROS Graph 可以理解成：

```text
当前整个 ROS2 系统的“通信关系图”
```

它描述：

```text
有哪些 Node

哪些 Node 创建了 Publisher

哪些 Node 创建了 Subscriber

Publisher 和 Subscriber 对应哪些 Topic

有哪些 Service

有哪些 Action

它们之间是什么关系
```

所以：

```text
ROS Graph
```

不是某一个具体程序。

它更像：

```text
当前 ROS2 分布式系统通信状态的抽象表示
```

---

# 2. 一个最简单的 ROS Graph

假设现在运行：

```bash
ros2 run demo_nodes_cpp talker
```

同时运行：

```bash
ros2 run demo_nodes_py listener
```

那么系统中存在：

```text
/talker

/listener
```

其中：

```text
/talker
```

向：

```text
/chatter
```

发布消息。

而：

```text
/listener
```

订阅：

```text
/chatter
```

因此 ROS Graph 可以表示成：

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

这就是一个非常简单的：

```text
ROS Graph
```

---

# 3. ROS Graph 里有什么

ROS Graph 中最重要的实体包括：

```text
Node
Publisher
Subscriber
Topic
Service
Client
Action Server
Action Client
```

可以大致表示为：

```text
ROS Graph
│
├── Nodes
│
├── Topics
│   ├── Publishers
│   └── Subscribers
│
├── Services
│   ├── Servers
│   └── Clients
│
└── Actions
    ├── Action Servers
    └── Action Clients
```

因此上一篇学习的：

```text
Node
Topic
Service
Action
```

实际上正是构成 ROS Graph 的核心元素。

---

# 4. ROS Graph 不是一个真正的“图文件”

这里要避免一个误区。

ROS Graph 并不是：

```text
系统里保存着一个 graph.txt 文件
```

也不是：

```text
有一个中央服务器保存完整 Graph
```

在 ROS2 中，它更接近：

```text
通过分布式 Discovery
得到当前系统中的通信实体信息
↓
然后形成 ROS Graph 的视图
```

这和 ROS1 的设计存在明显区别。

---

# 5. ROS1 中的 Graph

ROS1 中：

```text
ROS Master
```

承担了非常重要的注册和发现作用。

大致过程：

```text
Node A
↓
向 ROS Master 注册

Node B
↓
向 ROS Master 注册

ROS Master
↓
知道系统中有哪些 Node / Topic
```

所以 ROS1 中：

```text
ROS Master
```

天然掌握了大量通信关系信息。

---

# 6. ROS2 中的 Graph

ROS2 不存在 ROS1 那种中心式：

```text
ROS Master
```

ROS2 更接近：

```text
Node A
      ↘
       DDS Discovery
      ↗
Node B
```

各个通信参与者通过：

```text
DDS Discovery
```

发现彼此。

然后：

```text
Node
Publisher
Subscriber
Topic
Service
Action
```

等信息逐渐形成整个 ROS2 网络的：

```text
ROS Graph
```

因此 ROS2 Graph 是一种：

```text
分布式通信关系
```

---

# 7. ROS Graph 的核心意义

为什么一定要理解 ROS Graph？

因为很多 ROS2 问题，本质上不是：

```text
程序有没有运行
```

而是：

```text
程序有没有进入正确的 ROS Graph
```

例如：

```text
程序进程存在

但是：

ros2 node list

看不到它
```

那么问题就可能出现在：

```text
Discovery
RMW
DDS
Domain
网络
```

而不一定是：

```text
程序没有启动
```

---

# 8. 什么是 ros2 CLI

CLI：

```text
Command Line Interface
```

所以：

```text
ros2 CLI
```

就是：

```text
ROS2 Command Line Interface
```

也就是 ROS2 的命令行管理工具。

平时我们输入：

```bash
ros2 node list
```

实际上就是在调用：

```text
ros2 CLI
```

---

# 9. ros2 CLI 的基本结构

ROS2 的 CLI 设计比较统一。

基本形式是：

```text
ros2
+
对象
+
操作
```

例如：

```bash
ros2 node list
```

可以拆成：

```text
ros2
↓
node
↓
list
```

表示：

```text
使用 ros2 CLI
查看 Node
执行 list 操作
```

---

# 10. 常见 ros2 CLI 分类

我们经常使用：

```text
ros2 node
ros2 topic
ros2 service
ros2 action
ros2 param
ros2 interface
ros2 pkg
ros2 run
ros2 launch
ros2 bag
ros2 doctor
ros2 daemon
```

因此可以理解成：

```text
ros2
│
├── node
├── topic
├── service
├── action
├── param
├── interface
├── pkg
├── run
├── launch
├── bag
├── doctor
└── daemon
```

---

# 11. ros2 node

Node 相关命令：

```bash
ros2 node list
```

查看当前发现的所有 Node。

例如：

```text
/talker
/listener
```

查看某个节点：

```bash
ros2 node info /talker
```

可能看到：

```text
Subscribers:
Publishers:
Service Servers:
Service Clients:
Action Servers:
Action Clients:
```

因此：

```bash
ros2 node info
```

实际上就是在观察：

```text
某个 Node 在 ROS Graph 中连接了什么
```

---

# 12. ros2 topic

查看 Topic：

```bash
ros2 topic list
```

查看 Topic 和消息类型：

```bash
ros2 topic list -t
```

例如：

```text
/chatter [std_msgs/msg/String]
/parameter_events [rcl_interfaces/msg/ParameterEvent]
/rosout [rcl_interfaces/msg/Log]
```

查看某个 Topic：

```bash
ros2 topic info /chatter
```

---

# 13. 查看 Topic 的完整关系

非常推荐使用：

```bash
ros2 topic info /chatter -v
```

这里：

```text
-v
```

代表：

```text
verbose
```

也就是显示更详细的信息。

你可以看到：

```text
Publisher
Subscriber
Node name
QoS
```

等信息。

这条命令以后学习：

```text
QoS
```

时非常重要。

---

# 14. ros2 topic echo

执行：

```bash
ros2 topic echo /chatter
```

表示：

```text
订阅 /chatter
↓
把收到的消息显示在终端
```

所以：

```bash
ros2 topic echo
```

本质上并不是：

```text
偷偷查看网络数据包
```

而是 ROS2 CLI 创建相关通信实体参与 ROS2 通信。

这一点非常重要。

---

# 15. CLI 本身也可能参与 ROS Graph

这是理解 ROS2 CLI 时很容易忽略的一点。

比如执行：

```bash
ros2 topic echo /chatter
```

CLI 为了收到消息，本身需要：

```text
创建 Subscriber
```

因此：

```text
CLI 并不是永远站在 ROS2 系统外面观察
```

某些命令实际上会：

```text
主动参与 ROS2 通信
```

---

# 16. ros2 service

查看 Service：

```bash
ros2 service list
```

查看类型：

```bash
ros2 service list -t
```

查看单个 Service：

```bash
ros2 service type /service_name
```

例如：

```text
Node A
│
└── Service Server
       ▲
       │
       │
Service Client
│
Node B
```

CLI 可以帮助我们查看这部分 Graph。

---

# 17. ros2 action

查看 Action：

```bash
ros2 action list
```

查看类型：

```bash
ros2 action list -t
```

查看 Action 信息：

```bash
ros2 action info /action_name
```

例如导航系统：

```text
Navigation Client
       │
       │ Goal
       ▼
Navigation Server
       │
       ├── Feedback
       │
       └── Result
```

这些关系同样属于 ROS Graph。

---

# 18. ros2 interface

ROS2 通信除了：

```text
Topic 名字
```

还需要：

```text
消息类型
```

例如：

```text
std_msgs/msg/String
```

可以执行：

```bash
ros2 interface show std_msgs/msg/String
```

查看消息结构。

例如：

```text
string data
```

还可以查看：

```bash
ros2 interface list
```

---

# 19. ros2 pkg

查看 ROS2 Package：

```bash
ros2 pkg list
```

查找：

```bash
ros2 pkg executables
```

例如：

```bash
ros2 pkg executables demo_nodes_cpp
```

可以看到该 Package 中有哪些可执行程序。

---

# 20. ros2 run

执行：

```bash
ros2 run demo_nodes_cpp talker
```

结构：

```text
ros2 run
↓
Package
↓
Executable
```

这里：

```text
demo_nodes_cpp
```

是 Package。

```text
talker
```

是其中的可执行程序。

程序启动以后，里面创建：

```text
Node
```

于是：

```text
Node
```

进入 ROS Graph。

---

# 21. Package、Executable、Node 不要混淆

这是初学 ROS2 时非常容易混淆的三个概念。

例如：

```bash
ros2 run demo_nodes_cpp talker
```

其中：

```text
demo_nodes_cpp
=
Package
```

```text
talker
=
Executable
```

Executable 启动后可能创建：

```text
Node
```

所以关系是：

```text
Package
↓
包含 Executable
↓
运行 Executable
↓
创建 Node
↓
Node 加入 ROS Graph
```

它们不是同一个东西。

---

# 22. ros2 launch

一个机器人系统通常不会只有一个 Node。

例如无人机系统：

```text
livox_node
fastlio_node
planner_node
controller_node
rviz2
```

如果每次都分别：

```bash
ros2 run ...
```

会很麻烦。

因此可以使用：

```bash
ros2 launch
```

一次启动多个组件。

可以理解为：

```text
Launch
↓
启动多个 Executable
↓
创建多个 Node
↓
形成完整 ROS Graph
```

---

# 23. ros2 param

Node 可以拥有：

```text
Parameter
```

查看：

```bash
ros2 param list
```

读取：

```bash
ros2 param get /node_name parameter_name
```

设置：

```bash
ros2 param set /node_name parameter_name value
```

因此 Parameter 也是 ROS2 系统管理的重要组成部分。

---

# 24. ros2 doctor

如果系统出现问题，可以执行：

```bash
ros2 doctor --report
```

用于查看：

```text
ROS2 版本
RMW
网络
环境变量
平台信息
```

等内容。

这条命令在后续：

```text
ROS2 故障排查
```

中会再次使用。

---

# 25. ros2 CLI 到底怎么知道 Graph 信息

这就进入真正重要的部分了。

当我们输入：

```bash
ros2 node list
```

它并不是：

```text
扫描 Linux 当前所有进程
```

然后找：

```text
谁名字像 ROS2 Node
```

实际上 ROS2 CLI 需要获得：

```text
ROS Graph
```

信息。

逻辑可以粗略理解为：

```text
ros2 CLI
↓
获取 Graph 信息
↓
知道当前有哪些 Node
↓
显示结果
```

---

# 26. ROS2 CLI 和 Linux ps 的区别

例如：

```bash
ps aux
```

查看的是：

```text
Linux Process
```

而：

```bash
ros2 node list
```

查看的是：

```text
ROS Graph 中发现的 Node
```

这两个概念完全不同。

因此可能出现：

```text
Linux 进程存在
```

但是：

```bash
ros2 node list
```

看不到对应 Node。

这说明：

```text
Process 存在
≠
一定成功进入 ROS Graph
```

---

# 27. ROS Graph 信息从哪里来

继续向底层看：

```text
ROS Graph
↓
RMW
↓
DDS Discovery
```

DDS 会发现：

```text
Participant
Publisher
Subscriber
Topic
```

等通信实体。

ROS2 再通过：

```text
RMW
```

把这些底层信息转换成 ROS2 可以理解的：

```text
Node
Topic
Service
Action
```

相关 Graph 信息。

---

# 28. ros2 CLI、daemon 和 Graph

这里要提前引入下一篇的重要角色：

```text
ROS2 daemon
```

很多：

```text
ros2 node list
ros2 topic list
ros2 service list
```

操作会借助：

```text
daemon
```

维护或查询 Graph 信息。

可以先粗略理解为：

```text
ros2 CLI
↓
ROS2 daemon
↓
Graph 信息
↓
RMW
↓
DDS Discovery
```

但是：

```text
daemon
```

并不是：

```text
ROS2 的 Master
```

这一点非常重要。

下一篇会专门解释。

---

# 29. ROS1 与 ROS2 Graph 获取方式的区别

ROS1：

```text
rosnode
↓
ROS Master
↓
获取节点注册信息
```

ROS2：

```text
ros2 CLI
↓
Graph
↓
RMW
↓
DDS Discovery
```

所以：

```text
ROS1
=
中心式发现
```

而：

```text
ROS2
=
分布式发现
```

---

# 30. 为什么 ros2 node list 可能卡住

现在就可以理解之前遇到的问题了。

输入：

```bash
ros2 node list
```

看起来只是一个简单命令。

但背后可能涉及：

```text
ros2 CLI
↓
daemon
↓
ROS Graph
↓
RMW
↓
DDS
↓
Discovery
↓
Linux 网络
```

任何一层异常，都可能导致：

```text
ros2 node list
```

表现异常。

---

# 31. 为什么 ros2 topic list 也可能卡住

同样：

```bash
ros2 topic list
```

也需要知道：

```text
当前 Graph 中有哪些 Topic
```

因此它同样依赖：

```text
Graph Discovery
```

相关机制。

所以：

```text
node list 卡住
topic list 卡住
service list 卡住
```

如果同时出现，很可能需要向：

```text
daemon
RMW
DDS
网络
```

方向排查。

---

# 32. 先判断数据通信还是 Graph 查询异常

这是一个非常有用的排障思想。

比如：

```bash
ros2 node list
```

卡住。

不要马上认为：

```text
DDS 完全坏了
```

可以另外运行：

终端 1：

```bash
ros2 run demo_nodes_cpp talker
```

终端 2：

```bash
ros2 run demo_nodes_py listener
```

如果：

```text
talker
↓
listener
```

仍然能够正常传输数据，那么说明：

```text
ROS2 基本数据通信
```

可能仍然正常。

此时应该优先检查：

```text
CLI
daemon
Graph 查询
```

相关问题。

---

# 33. 用无人机系统理解 ROS Graph

假设现在运行：

```text
Livox LiDAR
FAST-LIO
Planner
Controller
PX4 Bridge
```

可能形成：

```text
livox_node
    │
    │ /livox/lidar
    ▼
fastlio_node
    │
    │ /Odometry
    ▼
planner_node
    │
    │ trajectory
    ▼
controller_node
    │
    │ control command
    ▼
px4_bridge
```

整个通信网络：

```text
Node
+
Topic
+
Publisher
+
Subscriber
+
Service
+
Action
```

组合起来，就是：

```text
ROS Graph
```

---

# 34. ROS Graph 可以帮助我们回答什么

通过 Graph，我们可以回答：

```text
节点启动了吗？
```

```text
Topic 存在吗？
```

```text
有没有 Publisher？
```

```text
有没有 Subscriber？
```

```text
消息类型是什么？
```

```text
QoS 是什么？
```

```text
Service Server 在哪里？
```

```text
Action Server 是否存在？
```

所以排障时 ROS Graph 是非常重要的观察对象。

---

# 35. 常用 ROS Graph 检查命令

查看 Node：

```bash
ros2 node list
```

查看 Node 详细信息：

```bash
ros2 node info /node_name
```

查看 Topic：

```bash
ros2 topic list
```

查看 Topic 和类型：

```bash
ros2 topic list -t
```

查看 Topic 详细信息：

```bash
ros2 topic info /topic_name -v
```

监听数据：

```bash
ros2 topic echo /topic_name
```

查看 Service：

```bash
ros2 service list
```

查看 Action：

```bash
ros2 action list
```

---

# 36. 一个推荐的 ROS2 检查顺序

当一个 ROS2 系统启动以后，可以按照：

```text
Node
↓
Topic
↓
Publisher / Subscriber
↓
Message
```

逐层检查。

第一步：

```bash
ros2 node list
```

确认节点是否存在。

第二步：

```bash
ros2 topic list
```

确认 Topic 是否存在。

第三步：

```bash
ros2 topic info /topic_name -v
```

确认：

```text
Publisher
Subscriber
QoS
```

第四步：

```bash
ros2 topic echo /topic_name
```

确认真正有没有数据。

---

# 37. 一个典型故障案例

例如：

```text
FAST-LIO 没有输出里程计
```

不要直接去改 FAST-LIO 源代码。

可以先：

```bash
ros2 node list
```

确认：

```text
fastlio_node
```

是否存在。

然后：

```bash
ros2 topic list
```

确认输入点云 Topic 是否存在。

再：

```bash
ros2 topic info /livox/lidar -v
```

检查：

```text
有没有 Publisher
有没有 Subscriber
QoS 是否合理
```

最后：

```bash
ros2 topic echo /livox/lidar
```

确认真正有数据。

这样就形成了：

```text
Graph
↓
连接关系
↓
真实数据
```

的排查流程。

---

# 38. ROS Graph 和 ROS2 架构的位置关系

现在可以把前几篇内容连起来。

```text
ROS2 Application
│
├── Node
│
├── Topic
│
├── Service
│
└── Action
       │
       ▼
   ROS Graph
       │
       ├── ros2 CLI
       │
       └── daemon
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
```

---

# 39. 一个重要区分

一定要区分三个概念：

```text
ROS2 Node
```

是：

```text
真正执行功能的通信实体
```

```text
ROS Graph
```

是：

```text
这些通信实体和关系形成的整体视图
```

```text
ros2 CLI
```

是：

```text
我们观察、调试和操作这些实体的工具
```

也就是说：

```text
Node / Topic / Service / Action
        ↓
      Graph
        ↑
      CLI 查看
```

---

# 40. 本篇核心总结

首先：

```text
Node
Topic
Service
Action
```

共同形成：

```text
ROS Graph
```

然后：

```text
ros2 CLI
```

帮助我们：

```text
查看
操作
调试
```

这个 Graph。

最重要的逻辑是：

```text
ROS2 通信实体
↓
ROS Graph
↓
ros2 CLI 观察
```

继续向底层：

```text
ros2 CLI
↓
daemon
↓
Graph
↓
RMW
↓
DDS Discovery
```

因此：

```bash
ros2 node list
```

看起来很简单，但背后实际上已经涉及到了 ROS2 的：

```text
CLI
Graph
daemon
RMW
DDS
Discovery
```

---

# 41. 下一篇

接下来有一个非常重要的问题：

```text
既然 ROS2 已经通过 DDS 分布式发现节点，
为什么还需要 ros2 daemon？
```

以及：

```text
daemon 是不是 ROS2 版本的 ROS Master？
```

答案是：

```text
不是。
```

下一篇专门拆解：

```text
05-ROS2-daemon.md
```
