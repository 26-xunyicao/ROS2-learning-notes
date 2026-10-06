# QoS

在上一篇中，我们已经把 ROS2 的通信链继续向下拆到了：

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
```

同时已经知道：

```text
Discovery
```

主要解决：

```text
谁存在？
```

而：

```text
Endpoint Matching
```

主要解决：

```text
双方能不能通信？
```

这里就会出现一个非常关键的问题：

```text
双方应该按照什么规则通信？
```

例如：

```text
消息必须保证送达吗？

允许丢数据吗？

Subscriber 晚启动以后，
还能不能收到之前的数据？

最多保存多少条历史消息？

数据多久没有更新算异常？
```

这些问题都属于：

```text
QoS
```

本篇重点就是搞清楚：

```text
ROS2 QoS
↓
RMW
↓
DDS QoS
↓
通信行为
```

这一整套逻辑。

---

# 1. QoS 是什么

QoS 全称：

```text
Quality of Service
```

可以翻译为：

```text
服务质量
```

在 ROS2 中，它主要描述：

```text
通信应该按照什么规则进行
```

也就是说：

```text
Topic 名称
```

只决定：

```text
传什么数据
```

而 QoS 决定：

```text
这些数据应该怎么传
```

---

# 2. 为什么 ROS2 需要 QoS

不同机器人数据的特点完全不同。

例如：

```text
相机图像
激光雷达点云
IMU
控制命令
地图
参数
状态信息
```

这些数据不能全部用同一种通信策略。

比如：

```text
IMU
```

可能每秒发布几百次。

如果丢掉一两帧：

```text
问题可能不大
```

因为马上就会有新数据。

但是：

```text
起飞命令
解锁命令
任务指令
```

如果丢失：

```text
可能非常严重
```

所以 ROS2 需要：

```text
不同数据
使用不同 QoS 策略
```

---

# 3. QoS 在 ROS2 架构中的位置

可以表示为：

```text
Application
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
Data Transport
```

也就是说：

```text
开发者在 ROS2 上层配置 QoS
```

然后：

```text
RMW
```

把这些设置映射到底层：

```text
DDS QoS
```

最终影响实际通信。

---

# 4. QoS 不只是“网络质量”

QoS 容易被误解成：

```text
网络快不快
```

但 ROS2 中的 QoS 不只是：

```text
带宽
延迟
```

它还包括：

```text
可靠性
历史数据
缓存深度
持久性
截止时间
数据寿命
活跃状态
```

所以：

```text
QoS
```

其实是一整套：

```text
通信行为规则
```

---

# 5. ROS2 中最常见的 QoS 策略

初学阶段最重要的几个是：

```text
Reliability
History
Depth
Durability
```

之后还会遇到：

```text
Deadline
Lifespan
Liveliness
```

可以先分成两组。

第一组：

```text
最常用
```

包括：

```text
Reliability
History
Depth
Durability
```

第二组：

```text
更高级的时间 / 活跃性策略
```

包括：

```text
Deadline
Lifespan
Liveliness
```

---

# 6. Reliability

Reliability：

```text
可靠性
```

主要回答：

```text
消息是否必须尽量保证送达？
```

最常见两种：

```text
Reliable
```

以及：

```text
Best Effort
```

---

# 7. Reliable

Reliable 可以理解为：

```text
尽量保证消息可靠送达
```

如果某些数据没有正确到达，

底层中间件可能：

```text
检测丢失
尝试重传
```

所以更适合：

```text
不能轻易丢失的重要信息
```

例如：

```text
任务命令
地图更新
关键状态
控制相关信息
```

---

# 8. Best Effort

Best Effort 可以理解为：

```text
尽力发送
但不保证每一条都送达
```

如果某一帧丢失：

```text
通常不重传
```

更适合：

```text
高频
实时
下一帧马上会到
```

的数据。

例如：

```text
激光雷达
相机
IMU
```

等传感器数据经常会使用这种思路。

---

# 9. Reliable 和 Best Effort 的核心区别

可以简单记：

```text
Reliable
=
更重视“不能丢”
```

```text
Best Effort
=
更重视“及时到”
```

---

# 10. 为什么传感器常用 Best Effort

假设激光雷达：

```text
10 Hz
```

也就是：

```text
每 100 ms
来一帧新点云
```

如果某一帧丢了，

与其：

```text
等待重传旧帧
```

有时候更希望：

```text
直接处理下一帧最新数据
```

因为机器人更关心：

```text
现在发生什么
```

而不是：

```text
把过去每一帧都补齐
```

---

# 11. 为什么命令可能更适合 Reliable

例如：

```text
起飞
降落
解锁
任务开始
```

这种数据不是：

```text
持续高频刷新
```

而是：

```text
某一个命令本身就很重要
```

所以更强调：

```text
可靠送达
```

---

# 12. History

History：

```text
历史策略
```

主要决定：

```text
系统应该保存多少历史消息
```

常见两种：

```text
Keep Last
Keep All
```

---

# 13. Keep Last

Keep Last 表示：

```text
只保留最近 N 条消息
```

这里：

```text
N
```

就是：

```text
Depth
```

例如：

```text
History = Keep Last
Depth = 10
```

表示：

```text
最多保存最近 10 条消息
```

---

# 14. Depth

Depth：

```text
队列深度
```

例如：

```text
Depth = 1
```

表示：

```text
只保存最新一条
```

如果新消息不断到来：

```text
旧消息可能被覆盖
```

---

# 15. 为什么 Depth 很重要

假设：

```text
Publisher = 100 Hz
```

Subscriber 处理速度：

```text
20 Hz
```

也就是说：

```text
数据来的速度
>
数据处理速度
```

这时消息就可能：

```text
堆积
```

Depth 决定：

```text
最多允许积压多少条
```

---

# 16. Depth 太小会怎样

例如：

```text
Depth = 1
```

而 Subscriber 很慢。

那么：

```text
新数据不断到来
↓
旧数据快速被替换
```

结果可能是：

```text
Subscriber 只看到最新数据
```

这对于：

```text
实时状态
```

可能反而是好事。

---

# 17. Depth 太大又会怎样

例如：

```text
Depth = 10000
```

如果处理速度长期跟不上：

```text
消息大量堆积
↓
延迟越来越大
↓
内存占用增加
```

最后可能变成：

```text
你收到的数据很完整
但已经严重过时
```

这在实时机器人系统中通常不是好事情。

---

# 18. Keep All

Keep All 表示：

```text
尽可能保存所有消息
```

直到受到：

```text
底层资源限制
```

影响。

相比：

```text
Keep Last
```

它通常需要更谨慎使用。

---

# 19. History 和 Depth 的关系

最常见组合：

```text
History = Keep Last
Depth = 10
```

表示：

```text
保留最近 10 条消息
```

因此：

```text
Depth
```

主要和：

```text
Keep Last
```

一起理解。

---

# 20. Durability

Durability：

```text
持久性
```

主要回答：

```text
Subscriber 如果晚启动，
还能不能收到之前发布的数据？
```

常见：

```text
Volatile
```

和：

```text
Transient Local
```

---

# 21. Volatile

Volatile 可以理解为：

```text
只接收连接建立之后的新数据
```

例如：

```text
Publisher
先运行
↓
已经发了 100 条消息
↓
Subscriber 现在才启动
```

如果使用：

```text
Volatile
```

那么 Subscriber 一般不会要求补之前那些历史数据。

它只关注：

```text
之后的新消息
```

---

# 22. Transient Local

Transient Local 可以理解为：

```text
Publisher 保留一定历史数据
让后来加入的 Subscriber
也有机会获得之前的数据
```

这有点类似：

```text
后来者也能拿到最近保存的状态
```

---

# 23. 一个非常直观的例子

假设：

```text
地图节点
```

只发布一次：

```text
当前地图
```

然后一直不再发布。

如果 Subscriber：

```text
晚 10 秒启动
```

使用纯：

```text
Volatile
```

它可能：

```text
错过这条地图消息
```

但如果使用合适的：

```text
Transient Local
```

Publisher 可以保留历史样本，

新 Subscriber 加入后：

```text
仍有机会获得这条数据
```

---

# 24. ROS1 中类似的概念

如果熟悉 ROS1，

可能会想到：

```text
latched topic
```

ROS2 中：

```text
Transient Local
```

在某些使用场景上可以帮助实现类似：

```text
后来订阅者获取已有状态数据
```

的效果。

但不要简单认为：

```text
Transient Local
=
ROS1 latch 完全一模一样
```

它们属于不同通信体系。

---

# 25. Reliability、History、Depth、Durability 放在一起

可以表示：

```text
QoS
│
├── Reliability
│   ├── Reliable
│   └── Best Effort
│
├── History
│   ├── Keep Last
│   └── Keep All
│
├── Depth
│   └── Queue Size
│
└── Durability
    ├── Volatile
    └── Transient Local
```

这是学习 ROS2 QoS 最重要的一张结构图。

---

# 26. 一个完整 QoS 示例

例如：

```text
Reliability = Best Effort
History     = Keep Last
Depth       = 5
Durability  = Volatile
```

可以理解成：

```text
尽力发送
↓
不保证每条都到
↓
只保留最近 5 条
↓
后来加入的 Subscriber 不补旧数据
```

这非常适合：

```text
高频实时传感器数据
```

一类场景。

---

# 27. Reliable 示例

另一个：

```text
Reliability = Reliable
History     = Keep Last
Depth       = 10
Durability  = Volatile
```

可以理解成：

```text
尽量保证可靠送达
↓
最多缓存最近 10 条
↓
只关注连接后的数据
```

---

# 28. QoS Profile

ROS2 中经常把一整组 QoS 配置称为：

```text
QoS Profile
```

也就是说：

```text
一个 QoS Profile
```

里面不只是：

```text
Reliability
```

而是：

```text
多种 QoS Policy 的组合
```

---

# 29. ROS2 为什么提供预定义 QoS

因为很多场景非常常见。

例如：

```text
Sensor Data
Services
Parameters
System Default
```

所以 ROS2 提供一些预定义 QoS Profile，

避免开发者：

```text
每次从零配置
```

---

# 30. Sensor Data QoS

传感器数据通常强调：

```text
实时
低延迟
允许一定丢包
```

因此常见思路是：

```text
Best Effort
```

配合有限：

```text
Depth
```

这也是为什么：

```text
雷达
相机
IMU
```

相关 Topic 经常会遇到：

```text
Best Effort
```

---

# 31. 为什么 ros2 topic echo 有时收不到传感器数据

这是 ROS2 中一个非常经典的问题。

假设：

```text
LiDAR Publisher
QoS = Best Effort
```

而你的 Subscriber：

```text
QoS 配置不兼容
```

那么可能出现：

```text
ros2 topic list
```

能看到：

```text
/livox/lidar
```

但是：

```bash
ros2 topic echo /livox/lidar
```

没有数据。

---

# 32. 为什么 Topic 能看到但没数据

因为：

```text
Discovery
```

已经成功。

所以：

```text
Topic 存在
```

能够被 Graph 看见。

但是：

```text
Endpoint Matching
```

还要检查：

```text
QoS Compatibility
```

如果 QoS 不兼容：

```text
发现成功
↓
但不能成功建立数据通信
```

---

# 33. 这就是 QoS 在 Discovery 后的位置

完整逻辑：

```text
Participant Discovery
↓
Endpoint Discovery
↓
Topic 匹配
↓
Type 匹配
↓
QoS Compatibility
↓
Endpoint Match
↓
Data Transport
```

所以：

```text
QoS
```

是：

```text
Endpoint 是否真正建立通信
```

的重要条件之一。

---

# 34. Publisher 和 Subscriber 的 QoS 不必完全相同

这是一个非常重要的点。

不是：

```text
所有 QoS 参数必须一模一样
```

才能通信。

而是：

```text
Publisher 提供的能力
```

和：

```text
Subscriber 请求的能力
```

需要满足：

```text
兼容性规则
```

---

# 35. Request / Offered 模型

DDS QoS 中经常使用：

```text
Requested
vs
Offered
```

的思想。

可以理解为：

```text
Publisher
=
我能提供什么
```

Subscriber：

```text
我要求什么
```

只有：

```text
Publisher 提供的能力
满足 Subscriber 的要求
```

双方才能匹配。

---

# 36. Reliability 兼容性的直观理解

例如 Publisher：

```text
Best Effort
```

Subscriber：

```text
Reliable
```

Subscriber 要求：

```text
可靠通信
```

但是 Publisher 只能提供：

```text
Best Effort
```

那么：

```text
可能不兼容
```

因为：

```text
Publisher 无法满足 Subscriber 的可靠性要求
```

---

# 37. 反过来呢

Publisher：

```text
Reliable
```

Subscriber：

```text
Best Effort
```

Publisher 能提供：

```text
Reliable
```

Subscriber 只要求：

```text
Best Effort
```

这种情况下：

```text
通常可以满足
```

可以直观理解成：

```text
我能提供更高要求
你只需要较低要求
```

---

# 38. 一个方便记忆的方法

可以粗略记成：

```text
Publisher = Offered
Subscriber = Requested
```

然后问：

```text
Offered
能不能满足
Requested？
```

如果能：

```text
Match
```

如果不能：

```text
No Match
```

---

# 39. Durability 也存在兼容性

类似：

```text
Publisher 提供什么 Durability
```

和：

```text
Subscriber 要求什么 Durability
```

也会影响：

```text
Endpoint Matching
```

因此 QoS 不是：

```text
Publisher 自己随便设置
Subscriber 自己随便设置
```

双方必须考虑：

```text
Compatibility
```

---

# 40. 查看 Topic QoS

非常推荐：

```bash
ros2 topic info /topic_name -v
```

例如：

```bash
ros2 topic info /livox/lidar -v
```

这里可以看到：

```text
Publisher
Subscriber
Node Name
Topic Type
QoS Profile
```

相关信息。

---

# 41. 为什么 -v 很重要

普通：

```bash
ros2 topic info /topic_name
```

可能只显示：

```text
Publisher count
Subscription count
```

而：

```bash
ros2 topic info /topic_name -v
```

会显示更完整的：

```text
Endpoint
QoS
```

信息。

所以排查：

```text
Topic 有但没有数据
```

时，

这是非常重要的命令。

---

# 42. ros2 topic echo 指定 QoS

某些情况下，

CLI Subscriber 需要使用与 Publisher 合适的 QoS。

可以根据当前 ROS2 CLI 支持的参数，

调整：

```text
Reliability
Durability
Depth
```

等设置。

核心思路不是：

```text
盲目换参数
```

而是：

```text
先查看 Publisher QoS
↓
再让 Subscriber 使用兼容 QoS
```

---

# 43. 用 Livox 和 FAST-LIO 理解 QoS

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

链路：

```text
Livox Publisher
↓
QoS Offered
↓
DDS
↓
QoS Compatibility
↓
FAST-LIO Subscriber
```

如果两边不兼容：

```text
FAST-LIO
```

可能：

```text
看得到 Topic
```

但是：

```text
收不到点云
```

---

# 44. 这对无人机传感器非常重要

无人机系统中：

```text
LiDAR
Camera
IMU
Odometry
```

通常都是：

```text
高频数据
```

这些数据很容易涉及：

```text
Best Effort
Depth
Latency
```

之间的权衡。

---

# 45. 控制命令又不同

例如：

```text
/cmd
```

或者关键任务命令，

更可能关心：

```text
Reliability
```

因为：

```text
某一条数据本身很重要
```

所以：

```text
传感器 QoS
```

和：

```text
控制 QoS
```

通常不会完全一样。

---

# 46. QoS 本质上是在做权衡

QoS 没有：

```text
所有参数都调到最高
=
最好
```

这样的逻辑。

例如：

```text
Reliable
```

更可靠，

但可能带来：

```text
重传
等待
额外开销
```

而：

```text
Best Effort
```

可能丢包，

但是：

```text
延迟更低
更适合最新数据优先
```

所以真正的问题是：

```text
你的应用最需要什么？
```

---

# 47. 实时性和可靠性的权衡

机器人系统中经常面对：

```text
数据完整
```

和：

```text
数据最新
```

之间的权衡。

例如：

```text
激光雷达
```

很多时候：

```text
最新一帧
```

比：

```text
补齐 500 ms 前丢掉的一帧
```

更重要。

---

# 48. History 和实时性的关系

如果：

```text
Depth 很大
```

数据大量堆积，

Subscriber 可能一直处理：

```text
旧数据
```

这对于实时控制来说可能很危险。

因此：

```text
Depth
```

不能只理解为：

```text
越大越不丢数据
```

还要考虑：

```text
延迟
```

---

# 49. 一个典型错误

假设：

```text
LiDAR 100 Hz
```

Subscriber 只能处理：

```text
20 Hz
```

如果：

```text
Depth = 1000
```

可能出现：

```text
数据不断积压
↓
Subscriber 一直处理旧点云
↓
机器人看到的是过去的环境
```

这可能比：

```text
偶尔丢帧
```

更糟糕。

---

# 50. Deadline

接下来进入更高级的 QoS。

Deadline：

```text
截止时间
```

可以理解为：

```text
我期望某类数据
至少多久更新一次
```

例如：

```text
IMU 应该每 10 ms 更新
```

如果超过：

```text
Deadline
```

还没有新数据，

系统可以检测到：

```text
Deadline Missed
```

---

# 51. Deadline 的用途

例如传感器理论上：

```text
100 Hz
```

也就是：

```text
10 ms 一帧
```

如果：

```text
500 ms
```

都没有收到数据，

可能意味着：

```text
传感器异常
网络异常
Publisher 卡死
```

Deadline 可以帮助检测这种情况。

---

# 52. Lifespan

Lifespan：

```text
数据寿命
```

主要描述：

```text
一条消息多长时间以后
就应该被认为过期
```

例如：

```text
位置数据
```

如果已经是：

```text
10 秒前
```

那么它可能已经：

```text
没有意义
```

---

# 53. 为什么 Lifespan 很适合实时系统

机器人很多数据具有：

```text
时效性
```

例如：

```text
障碍物位置
速度
姿态
控制状态
```

旧数据即使完整：

```text
也可能不能再使用
```

所以可以通过：

```text
Lifespan
```

描述：

```text
数据超过多久就失效
```

---

# 54. Liveliness

Liveliness：

```text
活跃性
```

主要用于判断：

```text
Publisher 是否还活着
```

也就是：

```text
这个通信实体
是不是仍然正常运行？
```

---

# 55. Liveliness 和 Deadline 不一样

Deadline 更关心：

```text
数据有没有按时更新
```

Liveliness 更关心：

```text
Publisher 是否仍处于活跃状态
```

所以两者解决的问题不同。

---

# 56. QoS 结构进一步扩展

现在可以画成：

```text
QoS
│
├── Reliability
│
├── History
│
├── Depth
│
├── Durability
│
├── Deadline
│
├── Lifespan
│
└── Liveliness
```

初学阶段最重要：

```text
Reliability
History
Depth
Durability
```

后面做复杂系统时再重点理解：

```text
Deadline
Lifespan
Liveliness
```

---

# 57. QoS 和 RMW 的关系

开发者配置：

```text
ROS2 QoS
```

例如：

```text
Reliable
Depth = 10
```

经过：

```text
rclcpp / rclpy
↓
rcl
↓
RMW
```

RMW 会将它映射到：

```text
底层 Middleware / DDS QoS
```

所以：

```text
RMW
```

在 QoS 中同样非常重要。

---

# 58. QoS 和 DDS 的关系

QoS 真正执行的位置主要在：

```text
DDS / Middleware
```

因为 DDS 需要根据 QoS 决定：

```text
是否重传
如何缓存
历史如何保存
Endpoint 是否匹配
数据是否已经过期
```

所以可以记住：

```text
ROS2 设置 QoS
↓
RMW 转换
↓
DDS 执行
```

---

# 59. QoS 和 Discovery 的关系

上一篇我们学习：

```text
Discovery
```

现在可以进一步补完整：

```text
Participant Discovery
↓
Endpoint Discovery
↓
交换 Topic / Type / QoS 信息
↓
QoS Compatibility
↓
Endpoint Matching
```

所以：

```text
QoS
```

并不是：

```text
真正开始发数据后
才有作用
```

它在：

```text
通信匹配阶段
```

就已经非常重要。

---

# 60. 一个完整通信过程

现在可以把：

```text
Discovery
QoS
Data Transport
```

全部连接起来：

```text
Participant A 启动
↓
Participant B 启动
↓
Participant Discovery
↓
Endpoint Discovery
↓
Topic 匹配
↓
Type 匹配
↓
QoS Compatibility
↓
Endpoint Match
↓
Data Transport
```

---

# 61. 为什么 QoS 是 ROS2 相比 ROS1 的重要变化

ROS1 中通信策略：

```text
相对固定
```

常见：

```text
TCPROS
UDPROS
```

ROS2 则把大量通信行为：

```text
显式暴露给开发者
```

通过：

```text
QoS
```

控制。

这使 ROS2 更适合：

```text
实时机器人
无人机
自动驾驶
工业控制
复杂网络系统
```

---

# 62. 但 QoS 也增加了学习成本

ROS1 中：

```text
Topic 名字和类型对了
```

很多时候就比较容易通信。

ROS2 中还需要继续考虑：

```text
QoS
```

所以会出现：

```text
Topic 明明存在
为什么就是没数据？
```

这种现象。

---

# 63. 一个典型排障流程

如果：

```bash
ros2 topic list
```

能看到：

```text
/livox/lidar
```

但：

```bash
ros2 topic echo /livox/lidar
```

没有数据，

第一步：

```bash
ros2 topic info /livox/lidar -v
```

---

# 64. 重点看什么

重点检查：

```text
Publisher count
Subscription count
Reliability
Durability
History
Depth
```

然后判断：

```text
Publisher QoS
```

和：

```text
Subscriber QoS
```

是否兼容。

---

# 65. 然后再判断是不是 QoS 问题

如果：

```text
QoS 兼容
```

但仍然没有数据，

继续向下检查：

```text
Publisher 是否真的 publish
DDS Transport
Linux Network
WSL2 Network
```

所以：

```text
QoS
```

只是：

```text
故障链中的一个层次
```

---

# 66. 一个 QoS 排障思路

可以整理成：

```text
Topic 存在
│
├── Publisher 存在吗？
│
├── Subscriber 存在吗？
│
├── Type 一致吗？
│
├── QoS 兼容吗？
│
└── 数据真的在发送吗？
```

---

# 67. 如果完全看不到 Topic

如果：

```bash
ros2 topic list
```

完全看不到目标 Topic，

那就不应该：

```text
第一时间怀疑 QoS
```

应该优先回到：

```text
Discovery
ROS_DOMAIN_ID
RMW
DDS
网络
```

因为：

```text
连 Topic 都没有发现
```

说明问题可能发生得更早。

---

# 68. 分层判断

可以记：

```text
Node 都看不到
↓
Discovery 问题
```

```text
Node 看得到
Topic 看不到
↓
Endpoint / Graph 问题
```

```text
Topic 看得到
但无数据
↓
QoS / 数据发布 / Transport
```

这种分层方式非常重要。

---

# 69. 用无人机系统做一次完整判断

假设：

```text
FAST-LIO 没有收到 Livox 点云
```

第一步：

```bash
ros2 node list
```

确认：

```text
livox_node
fastlio_node
```

是否存在。

---

第二步：

```bash
ros2 topic list
```

确认：

```text
/livox/lidar
```

是否存在。

---

第三步：

```bash
ros2 topic info /livox/lidar -v
```

检查：

```text
Publisher
Subscriber
Type
QoS
```

---

第四步：

```bash
ros2 topic echo /livox/lidar
```

确认：

```text
是否真的有点云数据
```

---

# 70. 这样就形成完整排障链

```text
Node
↓
Topic
↓
Endpoint
↓
QoS
↓
Message
↓
Transport
```

这比：

```text
一发现 FAST-LIO 没数据
就去改 FAST-LIO
```

要有效得多。

---

# 71. QoS 在整个 ROS2 架构中的位置

现在可以放回完整结构：

```text
ROS2 Application
│
├── Node
├── Publisher
└── Subscriber
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
       ├── Topic Matching
       ├── Type Matching
       ├── QoS Compatibility
       └── Data Transport
               │
               ▼
       Linux Network Stack
```

---

# 72. 本篇核心总结

第一：

```text
QoS
=
Quality of Service
```

第二：

```text
QoS
=
ROS2 通信规则
```

第三：

最重要的几个策略：

```text
Reliability
History
Depth
Durability
```

第四：

```text
QoS 不兼容
```

可能导致：

```text
Topic 能发现
但是收不到数据
```

第五：

```text
Publisher = Offered

Subscriber = Requested
```

需要满足：

```text
QoS Compatibility
```

第六：

```text
ROS2 设置 QoS
↓
RMW 映射
↓
DDS 执行 QoS
```

---

# 73. 最重要的几个概念

可以直接记：

```text
Reliable
=
尽量保证送达
```

```text
Best Effort
=
尽力发送，允许丢失
```

```text
Keep Last + Depth
=
保留最近 N 条消息
```

```text
Volatile
=
晚加入者不补之前的数据
```

```text
Transient Local
=
允许后来加入者获得 Publisher 保存的历史数据
```

---

# 74. 最重要的一条链

现在完整通信建立流程可以记成：

```text
Discovery
↓
Topic / Type Matching
↓
QoS Compatibility
↓
Endpoint Match
↓
Data Transport
```

而 QoS 在其中负责：

```text
决定双方按照什么规则通信
```

以及：

```text
双方是否具有兼容的通信条件
```

---

# 75. 下一篇

现在 ROS2 通信已经拆到了：

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
```

但是数据最终还是必须通过：

```text
操作系统网络
```

真正发送出去。

接下来就会出现新的问题：

```text
DDS 最后怎么把数据发到网卡？

Socket 是什么？

UDP / TCP 在哪里？

IP 在哪里？

Linux 网络栈到底负责什么？

为什么 WSL2 会让 ROS2 多机通信变复杂？

Windows、WSL2 和 Ubuntu 的网络关系是什么？
```

下一篇继续拆解：

```text
10-Linux网络栈与WSL2网络.md
```
