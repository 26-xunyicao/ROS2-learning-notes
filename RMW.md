# RMW

在上一篇中，我们已经把 ROS2 的通信链拆到了：

```text
Application
↓
rclcpp / rclpy
↓
rcl
```

同时我们知道：

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

最后都会进入：

```text
rcl
```

这一层。

但是这里马上会出现一个新的问题：

```text
为什么 rcl 不直接连接 DDS？

为什么中间还需要一个 RMW？

RMW 到底做了什么？

为什么 ROS2 可以更换不同 DDS 实现？
```

本篇重点就是搞清楚：

```text
rcl
↓
RMW
↓
DDS
```

这三层之间到底是什么关系。

---

# 1. RMW 是什么

RMW 全称：

```text
ROS Middleware Interface
```

可以翻译为：

```text
ROS 中间件接口
```

它位于：

```text
rcl
```

和：

```text
DDS / Middleware
```

之间。

结构：

```text
rcl
↓
RMW
↓
DDS
```

---

# 2. 为什么不能让 rcl 直接连接 DDS

假设没有 RMW。

那么结构可能变成：

```text
rcl
↓
Fast DDS
```

如果以后想换成另一个 DDS：

```text
Cyclone DDS
```

那就可能需要：

```text
修改 rcl
```

甚至：

```text
修改大量 ROS2 核心代码
```

这显然不合理。

---

# 3. RMW 的核心意义

RMW 最重要的作用就是：

```text
解耦
```

也就是：

```text
让 ROS2 上层
不要直接依赖某一个具体 DDS
```

于是架构变成：

```text
rcl
↓
统一的 RMW 接口
↓
不同 Middleware 实现
```

这样 ROS2 上层只需要：

```text
调用统一接口
```

而不需要知道：

```text
底层到底是哪一种 DDS
```

---

# 4. 可以把 RMW 理解成什么

可以把 RMW 理解成：

```text
翻译层
```

也可以理解成：

```text
适配层
```

或者：

```text
抽象接口层
```

例如 ROS2 上层说：

```text
创建一个 Publisher
```

RMW 会把这个操作转换成：

```text
底层 Middleware 能理解的操作
```

---

# 5. 一个简单类比

假设：

```text
rcl
```

只会说：

```text
ROS2 语言
```

而：

```text
Fast DDS
Cyclone DDS
其他 Middleware
```

各自有不同接口。

RMW 就像：

```text
翻译器
```

把：

```text
ROS2 请求
```

转换成：

```text
具体 Middleware 请求
```

---

# 6. RMW 在完整架构中的位置

整体结构：

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
Linux Network
```

所以 RMW 正好处于：

```text
ROS2 核心
```

和：

```text
通信中间件
```

之间。

---

# 7. RMW 不等于 DDS

这是一个非常重要的区别。

很多初学者会把：

```text
RMW
```

和：

```text
DDS
```

混在一起。

但它们不是同一层。

---

# 8. RMW 是接口层

RMW 更接近：

```text
统一接口
```

例如：

```text
创建 Node
创建 Publisher
创建 Subscriber
发送消息
接收消息
获取 Graph 信息
```

这些操作都可以通过 RMW 提供统一接口。

---

# 9. DDS 是具体通信机制

DDS 则负责：

```text
Discovery
Topic Matching
QoS
Data Transport
```

等真正的中间件能力。

所以：

```text
RMW
=
接口 / 适配层
```

而：

```text
DDS
=
具体中间件体系
```

---

# 10. 一个直观对比

可以记成：

```text
rcl
↓
RMW
↓
DDS
```

其中：

```text
rcl
=
ROS2 核心
```

```text
RMW
=
中间件抽象接口
```

```text
DDS
=
具体通信实现
```

---

# 11. RMW 为什么很重要

因为不同 DDS 实现：

```text
API 不同
配置不同
行为细节不同
实现方式不同
```

如果 ROS2 直接依赖某一个 DDS：

```text
整个 ROS2
```

就会被：

```text
某一个 DDS 厂商 / 实现
```

绑定。

RMW 可以避免这个问题。

---

# 12. RMW 接口大概负责什么

RMW 层会涉及：

```text
Node
Publisher
Subscriber
Service
Client
Graph
QoS
Serialization
Wait Set
Guard Condition
```

等功能。

也就是说：

```text
ROS2 很多通信操作
最终都会经过 RMW
```

---

# 13. 创建 Publisher 时发生什么

假设 C++ 中写：

```cpp
create_publisher(...)
```

大致链路：

```text
用户代码
↓
rclcpp
↓
rcl
↓
RMW
↓
DDS
```

在 RMW 这一层：

```text
ROS2 的 Publisher 请求
```

会被转换成：

```text
具体 Middleware 对应的 Publisher / DataWriter 创建操作
```

---

# 14. 创建 Subscriber 也是一样

大致链路：

```text
用户代码
↓
rclcpp / rclpy
↓
rcl
↓
RMW
↓
DDS
↓
创建对应 Subscriber / DataReader
```

所以：

```text
rcl
```

不用关心：

```text
DDS 具体 API
```

---

# 15. 发布消息时发生什么

假设：

```text
Publisher
```

要发送一条消息。

链路：

```text
Application
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

RMW 负责：

```text
把 ROS2 的发送动作
转换成底层中间件发送动作
```

---

# 16. 接收消息时发生什么

接收端：

```text
Network
↓
DDS
↓
RMW
↓
rcl
↓
rclcpp / rclpy
↓
Callback
```

所以：

```text
发送
接收
```

都会经过：

```text
RMW
```

---

# 17. ROS2 为什么可以换 DDS

因为 ROS2 上层依赖的是：

```text
RMW 接口
```

而不是：

```text
某个具体 DDS API
```

结构：

```text
ROS2
↓
RMW API
↓
不同 RMW Implementation
↓
不同 DDS
```

因此理论上可以更换：

```text
Middleware 实现
```

---

# 18. RMW Interface 和 RMW Implementation 的区别

这里还要再细分一个概念。

```text
RMW Interface
```

是：

```text
统一接口规范
```

而：

```text
RMW Implementation
```

是：

```text
这个接口的具体实现
```

例如：

```text
RMW Interface
↓
某个具体 rmw_xxx 实现
↓
具体 DDS
```

---

# 19. 可以类比成 USB 接口

例如：

```text
USB
```

定义：

```text
统一接口标准
```

但 USB 后面可以连接：

```text
键盘
鼠标
硬盘
摄像头
```

RMW 也是类似思想：

```text
ROS2 上层只认统一接口
```

至于底层具体是什么：

```text
由具体实现决定
```

---

# 20. ROS2 Humble 常见 RMW

在 ROS2 Humble 中，常见的 RMW 实现可能包括：

```text
rmw_fastrtps_cpp
rmw_cyclonedds_cpp
```

它们分别对应不同 DDS 实现。

例如：

```text
rmw_fastrtps_cpp
↓
Fast DDS
```

```text
rmw_cyclonedds_cpp
↓
Cyclone DDS
```

---

# 21. 查看当前 RMW

可以执行：

```bash
echo $RMW_IMPLEMENTATION
```

如果没有输出：

```text
不一定代表没有 RMW
```

而可能表示：

```text
使用 ROS2 默认选择的 RMW
```

---

# 22. 为什么环境变量可能是空的

例如：

```bash
echo $RMW_IMPLEMENTATION
```

输出为空。

这并不表示：

```text
ROS2 没有 Middleware
```

因为 ROS2 仍然必须使用：

```text
某个 RMW Implementation
```

只是：

```text
没有显式通过环境变量指定
```

---

# 23. 显式指定 RMW

例如：

```bash
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
```

或者：

```bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
```

这样可以告诉 ROS2：

```text
使用哪个 RMW Implementation
```

---

# 24. 为什么要切换 RMW

不同 RMW / DDS 可能在：

```text
Discovery
性能
延迟
吞吐量
网络行为
QoS 支持细节
多机通信表现
```

方面存在差异。

因此某些项目可能会：

```text
主动指定 RMW
```

---

# 25. 一个重要现象

假设：

```text
终端 A
```

使用：

```bash
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
```

而：

```text
终端 B
```

使用：

```bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
```

它们是否一定无法通信？

不一定。

因为它们底层仍然可能遵循：

```text
DDS / RTPS 相关标准
```

但具体互操作表现要看实现和配置。

所以：

```text
RMW 不同
```

不等于：

```text
一定完全隔离
```

---

# 26. RMW 和 DDS 的关系

可以进一步画成：

```text
rcl
↓
RMW Interface
↓
RMW Implementation
↓
DDS Implementation
```

例如：

```text
rcl
↓
RMW
↓
rmw_fastrtps_cpp
↓
Fast DDS
```

---

# 27. Fast DDS 是什么位置

Fast DDS 是：

```text
具体 DDS 实现
```

不是：

```text
RMW
```

所以不要把：

```text
rmw_fastrtps_cpp
```

和：

```text
Fast DDS
```

当成同一个概念。

---

# 28. Cyclone DDS 也是一样

结构：

```text
ROS2
↓
RMW
↓
rmw_cyclonedds_cpp
↓
Cyclone DDS
```

所以：

```text
rmw_cyclonedds_cpp
```

是适配层实现。

```text
Cyclone DDS
```

才是底层 DDS 实现。

---

# 29. 为什么名字容易混淆

因为很多包名里同时出现：

```text
rmw
fastrtps
cyclonedds
```

所以初学时很容易认为：

```text
RMW 就是 DDS
```

实际上应该分层看：

```text
RMW
=
ROS2 Middleware 抽象层

DDS
=
底层通信体系
```

---

# 30. RMW 与 ROS Graph

前面学习过：

```text
ROS Graph
```

而 Graph 信息最终也需要通过：

```text
RMW
```

向上提供。

例如：

```text
DDS Discovery
↓
发现 Participant / Endpoint
↓
RMW
↓
ROS2 Graph
```

所以：

```text
ros2 node list
```

最终也会受到：

```text
RMW
```

影响。

---

# 31. RMW 出问题会有什么现象

如果 RMW 层异常，

可能出现：

```text
Node 无法发现
Topic 看不到
Publisher / Subscriber 无法匹配
ros2 node list 异常
ros2 topic list 异常
talker / listener 无法通信
```

所以：

```text
RMW
```

是非常重要的排障层。

---

# 32. 为什么 daemon 异常也可能和 RMW 有关

上一篇已经知道：

```text
daemon
↓
ROS Graph
```

而 ROS Graph 又依赖：

```text
RMW
↓
DDS Discovery
```

所以：

```text
RMW 异常
```

可能表现成：

```text
daemon / CLI 查询异常
```

这就是为什么：

```text
ros2 node list 卡住
```

不能只盯着 daemon。

---

# 33. RMW 和 QoS

QoS 设置通常从：

```text
rclcpp / rclpy
```

开始。

例如：

```text
Reliable
Best Effort
Depth
Durability
```

然后：

```text
rcl
↓
RMW
↓
DDS QoS
```

所以：

```text
RMW
```

还负责把：

```text
ROS2 QoS 设置
```

转换成：

```text
底层 Middleware QoS 配置
```

---

# 34. 一个 QoS 转换链路

例如上层设置：

```text
Reliability = BEST_EFFORT
```

可以理解为：

```text
rclcpp QoS
↓
rcl
↓
RMW
↓
DDS Reliability QoS
```

所以 RMW 又承担了：

```text
语义映射
```

作用。

---

# 35. RMW 和消息类型

ROS2 消息最终也需要：

```text
序列化
```

才能通过网络发送。

例如：

```text
std_msgs/msg/String
```

最终需要转换成：

```text
网络上传输的字节数据
```

RMW 和类型支持系统会参与这个过程。

---

# 36. 可以理解成什么

应用层看到的是：

```text
String
PointCloud2
Imu
Odometry
```

底层网络看到的是：

```text
字节流 / 数据包
```

中间需要：

```text
类型支持
序列化
RMW
DDS
```

完成转换。

---

# 37. RMW 与 Service

Service 同样会经过 RMW。

例如：

```text
Client
↓
Request
↓
RMW
↓
DDS / Middleware
↓
Server
```

返回：

```text
Server
↓
Response
↓
RMW
↓
Client
```

---

# 38. RMW 与 Action

Action 上层看起来是：

```text
Goal
Feedback
Result
Cancel
```

但底层最终仍然依赖：

```text
ROS2 通信实体
↓
RMW
↓
DDS
```

所以 Action 也离不开 RMW。

---

# 39. RMW 与 Node Discovery

Node Discovery 表面上属于：

```text
ROS Graph
```

但底层依赖：

```text
DDS Discovery
```

而中间：

```text
RMW
```

负责：

```text
把 DDS 层发现结果
映射成 ROS2 可以理解的 Graph 信息
```

---

# 40. 用无人机系统理解 RMW

假设：

```text
Livox Driver
```

发布：

```text
/livox/lidar
```

FAST-LIO：

```text
订阅 /livox/lidar
```

发送端：

```text
Livox Node
↓
rclcpp
↓
rcl
↓
RMW
↓
DDS
```

接收端：

```text
DDS
↓
RMW
↓
rcl
↓
rclcpp
↓
FAST-LIO
```

所以 RMW 处于：

```text
每一条 ROS2 通信路径中
```

---

# 41. 为什么理解 RMW 很重要

如果只知道：

```text
Node
Topic
DDS
```

中间会感觉：

```text
ROS2 怎么突然就进入 DDS 了？
```

RMW 正好补上这块。

完整链路：

```text
Application
↓
Client Library
↓
ROS2 Core
↓
Middleware Interface
↓
Middleware
```

也就是：

```text
Application
↓
rclcpp / rclpy
↓
rcl
↓
RMW
↓
DDS
```

---

# 42. RMW 是 ROS2 模块化的关键

没有 RMW：

```text
ROS2
↓
绑定某一种 DDS
```

有了 RMW：

```text
ROS2
↓
统一接口
↓
不同 Middleware
```

因此：

```text
RMW
```

是 ROS2：

```text
模块化
可替换
可扩展
```

设计的重要基础。

---

# 43. 一个常见误区

不要认为：

```text
RMW
=
网络协议
```

RMW 不是：

```text
TCP
UDP
IP
```

它也不是：

```text
DDS 网络包
```

RMW 是：

```text
ROS2 与 Middleware 之间的接口抽象层
```

---

# 44. 另一个常见误区

不要认为：

```text
RMW
=
某一种 DDS
```

正确关系：

```text
RMW Interface
↓
RMW Implementation
↓
DDS Implementation
```

---

# 45. 再看整个 ROS2 通信栈

现在可以画成：

```text
ROS2 Application
│
├── C++ Node
│     ↓
│   rclcpp
│
├── Python Node
│     ↓
│   rclpy
│
└──────┬──────
       ↓
      rcl
       ↓
 RMW Interface
       ↓
RMW Implementation
       ↓
 DDS Implementation
       ↓
 Linux Network
```

---

# 46. 从上往下看

开发者写：

```text
Publisher
Subscriber
Service
Action
```

然后：

```text
rclcpp / rclpy
```

把它们转成：

```text
ROS2 Client Library 操作
```

再：

```text
rcl
```

调用：

```text
RMW
```

RMW 再调用：

```text
具体 DDS
```

最后进入：

```text
Linux 网络
```

---

# 47. 从下往上看

收到网络数据以后：

```text
Linux Network
↓
DDS
↓
RMW
↓
rcl
↓
rclcpp / rclpy
↓
Callback
↓
Application
```

所以：

```text
RMW
```

是上下两边的重要桥梁。

---

# 48. RMW 排障思路

如果遇到：

```text
Node 看不到
Topic 无法匹配
talker / listener 不通信
```

可以检查：

```bash
echo $RMW_IMPLEMENTATION
```

然后确认：

```text
当前到底使用什么 RMW
```

---

# 49. 进一步确认 ROS2 环境

还可以一起检查：

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

这样可以确认：

```text
ROS2 版本
RMW
Domain
```

是否一致。

---

# 50. 一个很重要的排障思维

如果：

```text
某一个 Node 自身报错
```

可能优先检查：

```text
Application
rclcpp / rclpy
```

如果：

```text
所有 Node 都无法互相发现
```

则应该继续往下看：

```text
RMW
DDS
Discovery
Network
```

---

# 51. RMW 在故障链中的位置

例如：

```text
ros2 node list 卡住
```

可能链路：

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
Network
```

如果：

```text
daemon 重启也没用
```

那下一层就应该关注：

```text
RMW
```

---

# 52. RMW 与下一篇 DDS 的关系

到这里我们已经知道：

```text
RMW
```

负责：

```text
把 ROS2 上层操作
映射到具体 Middleware
```

但是：

```text
真正负责节点自动发现是谁？

Publisher / Subscriber 怎么匹配？

为什么不需要 ROS Master？

QoS 到底在哪里执行？

数据真正怎么传出去？
```

这些问题就已经进入：

```text
DDS
```

这一层了。

---

# 53. 本篇核心总结

第一：

```text
RMW
=
ROS Middleware Interface
```

第二：

```text
RMW
=
ROS2 与底层 Middleware 之间的抽象接口层
```

第三：

```text
RMW
≠
DDS
```

正确关系是：

```text
rcl
↓
RMW
↓
DDS
```

第四：

```text
RMW 的核心价值
=
解耦 ROS2 和具体 Middleware
```

第五：

```text
ROS2 可以切换不同 DDS
```

很大程度上依赖：

```text
RMW
```

这一层设计。

---

# 54. 最重要的一条链

现在需要牢牢记住：

```text
Application
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

到这里，

ROS2 从：

```text
应用程序
```

已经一路进入：

```text
真正的通信中间件
```

---

# 55. 下一篇

接下来就是整个 ROS2 架构中非常核心的一层：

```text
DDS
```

下一篇重点解决：

```text
DDS 到底是什么？

为什么 ROS2 要使用 DDS？

DDS Discovery 怎么替代 ROS Master？

Participant 是什么？

Publisher / Subscriber 如何发现彼此？

Topic 怎么匹配？

为什么 ROS2 能够去中心化？
```

下一篇继续拆解：

```text
08-DDS与Discovery.md
```
