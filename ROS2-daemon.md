# ROS2 daemon

在上一篇中，我们已经认识了：

```text
ROS Graph
ros2 CLI
```

并且知道：

```bash
ros2 node list
```

```bash
ros2 topic list
```

这些命令并不是简单地扫描 Linux 进程，而是在获取：

```text
ROS Graph
```

中的信息。

但是这里会产生一个新的问题：

```text
既然 ROS2 已经通过 DDS Discovery
实现了分布式节点发现，

为什么 ros2 CLI 还需要 daemon？
```

同时还有一个非常容易产生的误区：

```text
ROS1 有 roscore

ROS2 有 ros2 daemon

所以：

ros2 daemon = ROS2 的 roscore？
```

答案是：

```text
不是。
```

本篇重点就是搞清楚：

```text
ros2 CLI
↓
daemon
↓
ROS Graph
↓
RMW
↓
DDS Discovery
```

这条链路到底是什么逻辑。

---

# 1. daemon 是什么

`daemon` 中文通常翻译为：

```text
守护进程
```

可以简单理解成：

```text
长期运行在后台
为其他程序提供服务的进程
```

ROS2 中的：

```text
ros2 daemon
```

就是一个后台进程。

它主要服务于：

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

```bash
ros2 service list
```

这些命令都需要知道：

```text
当前 ROS Graph 中有哪些通信实体
```

daemon 的作用之一，就是帮助：

```text
维护
缓存
查询
```

这些 Graph 信息。

---

# 2. daemon 在 ROS2 架构中的位置

可以先粗略表示为：

```text
用户
↓
ros2 CLI
↓
ros2 daemon
↓
ROS Graph
↓
RMW
↓
DDS Discovery
```

所以 daemon 主要属于：

```text
CLI / Graph 查询体系
```

而不是：

```text
ROS2 实际数据传输主链
```

---

# 3. 为什么需要 daemon

假设 ROS2 没有 daemon。

每次执行：

```bash
ros2 node list
```

CLI 都需要重新：

```text
创建 ROS2 通信实体
↓
加入 DDS 网络
↓
等待 Discovery
↓
发现其他 Participant
↓
收集 Graph 信息
↓
显示节点
↓
退出
```

这样会产生两个问题：

```text
每次 CLI 查询都需要重新发现
```

以及：

```text
命令响应可能比较慢
```

因此 ROS2 使用：

```text
daemon
```

作为一个长期存在的后台进程。

可以理解成：

```text
daemon 持续观察 ROS Graph
↓
保存已经发现的信息
↓
CLI 查询时直接获取
```

这样：

```bash
ros2 node list
```

就不需要每次都从零开始发现整个 ROS2 网络。

---

# 4. daemon 和 ros2 CLI 的关系

例如执行：

```bash
ros2 node list
```

可以粗略理解为：

```text
用户
↓
ros2 node list
↓
ros2 CLI
↓
daemon
↓
ROS Graph
↓
返回 Node 信息
↓
终端显示结果
```

所以：

```text
ros2 CLI
```

更像：

```text
用户操作入口
```

而：

```text
daemon
```

更像：

```text
后台 Graph 信息服务
```

---

# 5. daemon 与 ROS Graph 的关系

daemon 并不是：

```text
ROS Graph 的创造者
```

ROS Graph 的信息本质来源于：

```text
Node
Publisher
Subscriber
Service
Action
```

这些 ROS2 通信实体。

而这些通信实体之间的发现关系，又依赖：

```text
RMW
↓
DDS Discovery
```

因此更准确的关系是：

```text
ROS2 通信实体
↓
DDS Discovery
↓
形成分布式 Graph 信息
↓
daemon 获取 / 缓存这些信息
↓
ros2 CLI 查询
```

所以：

```text
daemon
```

更像一个：

```text
ROS Graph 的观察者和缓存者
```

---

# 6. daemon 不是 ROS Master

这是本篇最重要的概念。

ROS1 中：

```text
ROS Master
```

负责：

```text
Node 注册
Topic 注册
Publisher / Subscriber 信息管理
节点发现
```

结构：

```text
Node A
↓
ROS Master
↑
Node B
```

ROS Master 是 ROS1 中非常重要的：

```text
中心式协调组件
```

---

# 7. ROS2 为什么不需要 ROS Master

ROS2 使用：

```text
DDS Discovery
```

进行分布式发现。

结构更接近：

```text
Node A
   ↘
    DDS Discovery
   ↗
Node B
```

节点之间不需要先：

```text
向中央 Master 注册
```

而是通过 DDS：

```text
自动发现彼此
```

所以 ROS2 不需要：

```bash
roscore
```

才能开始节点通信。

---

# 8. ROS1 的 roscore 到底做什么

ROS1 中运行：

```bash
roscore
```

通常会启动多个重要组件。

包括：

```text
ROS Master
Parameter Server
rosout
```

因此：

```text
roscore
```

不是简单的命令行辅助工具。

它实际上启动了 ROS1 系统中非常关键的：

```text
中心基础设施
```

很多 ROS1 节点都依赖：

```text
ROS Master
```

完成发现。

---

# 9. ROS2 daemon 和 roscore 的本质区别

可以直接对比：

```text
ROS1

roscore
↓
ROS Master
↓
节点注册与发现
↓
ROS1 通信体系的重要中心
```

而 ROS2：

```text
ROS2 Node
↓
RMW
↓
DDS Discovery
↓
分布式节点发现
```

同时：

```text
ros2 CLI
↓
daemon
↓
Graph 查询
```

所以：

```text
roscore
≠
ros2 daemon
```

---

# 10. 一个非常重要的判断

如果：

```text
ros2 daemon
```

停止，

并不意味着：

```text
ROS2 节点之间立刻无法通信
```

因为真正的通信主链仍然是：

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
Network
↓
另一个 Node
```

daemon：

```text
不在这条数据主链中
```

---

# 11. 用 talker / listener 理解

运行：

终端 1：

```bash
ros2 run demo_nodes_cpp talker
```

终端 2：

```bash
ros2 run demo_nodes_py listener
```

实际数据链：

```text
/talker
↓
Publisher
↓
RMW
↓
DDS
↓
Subscriber
↓
/listener
```

这里真正负责通信的是：

```text
RMW
DDS
```

而不是：

```text
ros2 daemon
```

---

# 12. daemon 不负责 Topic 数据传输

例如激光雷达节点：

```text
livox_node
```

持续发布：

```text
/livox/lidar
```

FAST-LIO：

```text
fastlio_node
```

订阅该 Topic。

数据链：

```text
livox_node
↓
Publisher
↓
RMW
↓
DDS
↓
Network
↓
Subscriber
↓
fastlio_node
```

daemon 不参与：

```text
点云数据本身的传输
```

---

# 13. daemon 不负责 Publisher 和 Subscriber 匹配

Publisher 与 Subscriber 是否能够建立通信，主要与：

```text
Topic 名称
消息类型
QoS
Domain
DDS Discovery
```

有关。

真正负责底层匹配的是：

```text
RMW
↓
DDS
```

而不是：

```text
daemon
```

---

# 14. daemon 也不负责 QoS

例如：

```text
Publisher
QoS = BEST_EFFORT
```

Subscriber：

```text
QoS = RELIABLE
```

双方是否能够匹配，是：

```text
QoS Compatibility
```

问题。

这属于：

```text
ROS2
↓
RMW
↓
DDS QoS
```

体系。

daemon 不负责：

```text
决定 QoS 是否兼容
```

---

# 15. 那 daemon 到底主要影响什么

主要影响：

```text
ROS Graph 查询
```

例如：

```bash
ros2 node list
```

```bash
ros2 topic list
```

```bash
ros2 service list
```

这些命令需要快速获得：

```text
当前 Graph 状态
```

daemon 就主要服务于这一类需求。

---

# 16. 查看 daemon 状态

执行：

```bash
ros2 daemon status
```

如果 daemon 正在运行，通常会显示类似：

```text
The daemon is running
```

如果没有运行：

```text
The daemon is not running
```

---

# 17. 启动 daemon

执行：

```bash
ros2 daemon start
```

用于启动 daemon。

---

# 18. 停止 daemon

执行：

```bash
ros2 daemon stop
```

用于停止 daemon。

---

# 19. daemon 通常会自动启动

通常不需要每次启动 ROS2 后手动执行：

```bash
ros2 daemon start
```

例如第一次运行：

```bash
ros2 node list
```

ROS2 CLI 在需要时通常会启动对应 daemon。

所以正常使用 ROS2 时：

```text
daemon 往往是自动管理的
```

---

# 20. 查看 daemon 进程

可以使用 Linux 命令：

```bash
ps aux | grep ros2
```

或者：

```bash
ps aux | grep daemon
```

可能会看到类似：

```text
_ros2_daemon
```

相关进程。

这说明：

```text
daemon 本质上也是 Linux 中运行的一个进程
```

---

# 21. daemon 本身也需要 ROS2 环境

daemon 不是一个完全脱离 ROS2 的特殊程序。

它本身同样需要：

```text
ROS2
RMW
DDS
Discovery
```

等底层机制。

因此：

```text
daemon 自己也可能受到 DDS 或网络问题影响
```

---

# 22. 为什么 daemon 可能出现异常

例如当前环境：

```text
Windows 11
↓
WSL2
↓
Ubuntu 22.04
↓
ROS2 Humble
```

运行一段时间以后，可能发生：

```text
WSL2 网络发生变化
```

例如：

```text
Windows 睡眠
WSL 重启
VPN 开启
VPN 关闭
虚拟网卡变化
IP 地址变化
```

而旧的 daemon 可能仍然保持：

```text
之前的通信状态
```

这时候 ROS2 CLI 就可能表现异常。

---

# 23. 一个我们实际遇到过的现象

例如执行：

```bash
ros2 node list
```

没有返回。

然后：

```bash
ros2 topic list
```

也没有返回。

再执行：

```bash
ros2 daemon stop
```

甚至：

```text
daemon stop 本身也可能卡住
```

这种时候就需要开始判断：

```text
到底是 CLI

还是 daemon

还是 DDS Discovery

还是网络
```

出了问题。

---

# 24. daemon stop 卡住怎么办

如果：

```bash
ros2 daemon stop
```

长时间不返回，

可以：

```text
Ctrl + C
```

停止当前命令。

然后执行：

```bash
pkill -f _ros2_daemon
```

强制结束 daemon 进程。

---

# 25. 然后重新启动 daemon

执行：

```bash
ros2 daemon start
```

然后：

```bash
ros2 daemon status
```

最后重新测试：

```bash
ros2 node list
```

---

# 26. 一个基本 daemon 重置流程

可以记住：

```bash
ros2 daemon stop
```

如果失败：

```bash
pkill -f _ros2_daemon
```

然后：

```bash
ros2 daemon start
```

再测试：

```bash
ros2 node list
```

或者：

```bash
ros2 topic list
```

---

# 27. 但不要把所有问题都归结为 daemon

这是排障时很重要的一点。

如果：

```bash
ros2 node list
```

卡住，

可能的链路其实是：

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

任何一层异常：

```text
都可能导致最终 CLI 表现异常
```

所以：

```text
重启 daemon
```

只是：

```text
排障中的一个步骤
```

而不是最终答案。

---

# 28. 怎么判断是不是 daemon 层的问题

一个非常有效的方法是：

```text
把“Graph 查询”
和
“真实数据通信”
分开测试
```

例如：

```bash
ros2 node list
```

卡住。

这属于：

```text
Graph 查询
```

然后另外启动：

终端 1：

```bash
ros2 run demo_nodes_cpp talker
```

终端 2：

```bash
ros2 run demo_nodes_py listener
```

---

# 29. 如果 talker / listener 正常

假设：

```text
talker
↓
listener
```

可以正常传递：

```text
Hello World
```

但是：

```bash
ros2 node list
```

仍然异常。

那么可以初步判断：

```text
Publisher / Subscriber 基本通信
```

是正常的。

这时候应该重点检查：

```text
ros2 CLI
daemon
Graph 查询
```

---

# 30. 如果 talker / listener 也失败

如果：

```text
ros2 node list
```

异常，

同时：

```text
talker
↓
listener
```

也无法正常通信，

问题就很可能不只是 daemon。

这时候需要继续往底层检查：

```text
RMW
↓
DDS
↓
Discovery
↓
Linux 网络
↓
WSL2 网络
```

---

# 31. ROS2 可以看成两条逻辑链

理解 daemon 最好的方式之一，就是把 ROS2 分成：

```text
数据通信链
```

和：

```text
管理 / 查询链
```

---

# 32. 数据通信链

真正的数据通信：

```text
Node A
↓
Publisher
↓
rclcpp / rclpy
↓
rcl
↓
RMW
↓
DDS
↓
Network
↓
DDS
↓
RMW
↓
Subscriber
↓
Node B
```

这是：

```text
消息真正流动的路径
```

---

# 33. 管理 / 查询链

例如：

```bash
ros2 node list
```

更接近：

```text
用户
↓
ros2 CLI
↓
daemon
↓
ROS Graph
↓
RMW / DDS Discovery
```

这是：

```text
观察当前 ROS2 系统的路径
```

---

# 34. 两条链的关系

可以表示成：

```text
                         ROS2
                          │
             ┌────────────┴────────────┐
             │                         │
             │                         │
        数据通信链                管理 / 查询链
             │                         │
            Node                   ros2 CLI
             │                         │
      rclcpp / rclpy                daemon
             │                         │
            rcl                  ROS Graph
             │                         │
            RMW ◄──────────────────────┘
             │
            DDS
             │
         Network
```

所以：

```text
daemon
```

与 ROS2 通信体系：

```text
有关联
```

但并不是：

```text
所有数据必须经过 daemon
```

---

# 35. daemon 与 ROS_DOMAIN_ID

ROS2 中还有一个非常重要的环境变量：

```text
ROS_DOMAIN_ID
```

查看：

```bash
echo $ROS_DOMAIN_ID
```

例如：

```text
Terminal A
ROS_DOMAIN_ID=0
```

而：

```text
Terminal B
ROS_DOMAIN_ID=10
```

那么两边看到的 ROS Graph 可能不同。

---

# 36. daemon 也受到 Domain 影响

因为 daemon 需要观察：

```text
ROS Graph
```

而 ROS Graph 又受：

```text
DDS Domain
```

影响。

所以：

```text
daemon 看到什么
```

取决于它所在的：

```text
ROS_DOMAIN_ID
```

环境。

这也是为什么修改：

```text
ROS_DOMAIN_ID
```

后，有时需要重新启动相关 ROS2 进程。

---

# 37. ROS_LOCALHOST_ONLY

还可以检查：

```bash
echo $ROS_LOCALHOST_ONLY
```

它会影响：

```text
ROS2 的发现范围
```

如果通信被限制到：

```text
localhost
```

那么其他设备上的 ROS2 节点可能：

```text
无法被发现
```

---

# 38. daemon 看到的 Graph 也会受到影响

逻辑是：

```text
ROS_LOCALHOST_ONLY
↓
影响 DDS Discovery 范围
↓
影响 ROS Graph
↓
daemon 获取到的 Graph 发生变化
↓
ros2 node list 结果变化
```

所以 CLI 显示异常时，也不能只看：

```text
daemon
```

还需要继续看环境变量。

---

# 39. 常用环境变量检查

可以执行：

```bash
echo $ROS_DISTRO
```

```bash
echo $ROS_VERSION
```

```bash
echo $RMW_IMPLEMENTATION
```

```bash
echo $ROS_DOMAIN_ID
```

```bash
echo $ROS_LOCALHOST_ONLY
```

这些命令可以帮助确认当前：

```text
ROS2 版本
RMW
DDS Domain
Discovery 范围
```

等基础环境。

---

# 40. 一个推荐的 daemon 排障流程

如果出现：

```bash
ros2 node list
```

长时间没有返回，

第一步：

```text
Ctrl + C
```

然后检查：

```bash
ros2 daemon status
```

---

第二步，尝试：

```bash
ros2 daemon stop
```

然后：

```bash
ros2 daemon start
```

---

第三步，如果 daemon stop 也卡住：

```bash
pkill -f _ros2_daemon
```

然后：

```bash
ros2 daemon start
```

---

第四步，再测试：

```bash
ros2 node list
```

```bash
ros2 topic list
```

---

第五步，如果仍然异常：

```bash
echo $ROS_DISTRO
echo $ROS_VERSION
echo $RMW_IMPLEMENTATION
echo $ROS_DOMAIN_ID
echo $ROS_LOCALHOST_ONLY
```

---

第六步，测试真实通信：

```bash
ros2 run demo_nodes_cpp talker
```

以及：

```bash
ros2 run demo_nodes_py listener
```

---

# 41. 排障判断树

可以整理成：

```text
ros2 node list 卡住
│
├── daemon 正常吗？
│
├── 重启 daemon 后恢复吗？
│
├── talker / listener 能通信吗？
│
│
├── 能
│   ↓
│   优先检查：
│   CLI / daemon / Graph 查询
│
└── 不能
    ↓
    继续检查：
    RMW
    DDS
    Discovery
    Linux 网络
    WSL2 网络
```

---

# 42. 为什么理解 daemon 很重要

如果不知道 daemon 的位置，

看到：

```bash
ros2 node list
```

卡住，

很容易认为：

```text
ROS2 完全坏了
```

甚至直接：

```text
重装 ROS2
```

但实际上可能只是：

```text
CLI / daemon 层异常
```

而真正的：

```text
Publisher / Subscriber
```

通信仍然正常。

---

# 43. daemon 在整个 ROS2 架构中的位置

现在把前面的知识全部连接起来：

```text
ROS2 Application
│
├── Node
├── Topic
├── Service
└── Action
       │
       ▼
    ROS Graph
       ▲
       │
    daemon
       ▲
       │
   ros2 CLI
```

真正的节点通信链：

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
Linux Network
```

所以一定要区分：

```text
CLI / Graph 管理体系
```

与：

```text
节点真实通信体系
```

---

# 44. ROS1 与 ROS2 再对比一次

ROS1：

```text
rosnode list
↓
ROS Master
↓
节点注册信息
```

ROS2：

```text
ros2 node list
↓
daemon
↓
ROS Graph
↓
RMW
↓
DDS Discovery
```

所以最核心的区别仍然是：

```text
ROS1
=
中心式 Master
```

而：

```text
ROS2
=
分布式 Discovery
```

---

# 45. 最重要的三个结论

第一：

```text
ros2 daemon
≠
ROS Master
```

第二：

```text
ros2 daemon
≠
ROS2 数据传输核心
```

第三：

```text
ros2 daemon
=
ros2 CLI 的 Graph 查询辅助进程
```

---

# 46. 本篇核心总结

可以把 daemon 简化理解为：

```text
ros2 daemon
=
长期运行的 ROS Graph 缓存 / 查询辅助进程
```

它主要服务：

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

而真正的 ROS2 通信主链仍然是：

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
Network
```

这也是为什么：

```text
daemon 出问题
```

并不一定代表：

```text
整个 ROS2 通信系统失效
```

---

# 47. 下一篇

现在我们已经知道：

```text
Node / Topic / Service / Action
↓
ROS Graph
↓
ros2 CLI
↓
daemon
```

接下来需要回到真正的：

```text
ROS2 Node
```

本身。

当我们写：

```text
C++
```

时，经常看到：

```text
rclcpp
```

当我们写：

```text
Python
```

时，经常看到：

```text
rclpy
```

但是它们下面为什么还有：

```text
rcl
```

？

以及：

```text
C++ 和 Python 为什么最终都能进入同一套 ROS2 通信体系？
```

下一篇继续拆解：

```text
06-rclcpp-rclpy与rcl.md
```
