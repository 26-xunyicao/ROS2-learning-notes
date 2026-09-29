# Node、Topic、Service、Action

在 ROS2 中，最常见的四个核心概念是：

```text
Node
Topic
Service
Action
```

它们分别解决不同类型的机器人通信问题。

可以先用一句话记住：

```text
Node = 谁在工作
Topic = 持续发数据
Service = 问一次，答一次
Action = 执行一个过程，并持续反馈
```

---

# 1. Node 是什么

Node 是 ROS2 中最基本的运行单元。

一个 ROS2 系统通常不是一个大程序，而是由很多 Node 组成。

例如：

```text
camera_node
lidar_node
imu_node
localization_node
planner_node
controller_node
```

每个 Node 负责一部分功能。

例如无人机系统可以拆成：

```text
无人机系统
├── flight_control_node
├── lidar_node
├── fastlio_node
├── planner_node
├── mavlink_bridge_node
└── visualization_node
```

这样做的好处是：

```text
功能解耦
模块独立
便于调试
便于复用
便于多机通信
```

---

# 2. Node 可以包含什么

一个 Node 内部可以拥有很多 ROS2 通信实体。

例如：

```text
Node
├── Publisher
├── Subscriber
├── Service Server
├── Service Client
├── Action Server
├── Action Client
├── Timer
└── Parameter
```

所以：

```text
Node
```

并不是单纯“一个程序名字”。

它更像：

```text
一个 ROS2 功能模块
```

---

# 3. 查看 Node

查看当前 ROS2 系统中的节点：

```bash
ros2 node list
```

例如：

```text
/talker
/listener
```

查看某个节点详细信息：

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

也就是说：

```bash
ros2 node info
```

可以帮助我们查看一个 Node 和其他通信实体之间的关系。

---

# 4. Node 名称

ROS2 Node 通常有自己的名字。

例如：

```text
/talker
/listener
/lidar_node
/camera_node
```

Node 名称用于：

```text
识别节点
查询节点
构建 ROS Graph
```

例如：

```bash
ros2 node info /listener
```

---

# 5. Topic 是什么

Topic 是 ROS2 中最常用的通信方式之一。

它用于：

```text
持续的数据流通信
```

结构：

```text
Publisher
↓
Topic
↓
Subscriber
```

例如：

```text
激光雷达
↓
/scan
↓
导航节点
```

或者：

```text
相机
↓
/camera/image
↓
图像处理节点
```

---

# 6. Topic 的核心特点

Topic 通信通常是：

```text
异步
持续
多对多
```

一个 Publisher 可以对应多个 Subscriber。

例如：

```text
             ┌── visualization_node
             │
lidar_node ──┼── localization_node
             │
             └── mapping_node
```

它们都可以订阅：

```text
/points
```

---

# 7. Publisher

Publisher：

```text
发布者
```

负责向 Topic 发送消息。

例如：

```text
lidar_node
↓
Publisher
↓
/scan
```

在代码逻辑上：

```text
Node
↓
创建 Publisher
↓
周期性 publish
```

---

# 8. Subscriber

Subscriber：

```text
订阅者
```

负责监听某个 Topic。

例如：

```text
/scan
↓
Subscriber
↓
navigation_node
```

当新的消息到达以后，会触发相应的回调函数。

---

# 9. Topic 名称

例如：

```text
/chatter
/scan
/imu
/cmd_vel
/odom
/camera/image_raw
```

Topic 名称通常代表：

```text
这条数据流是什么
```

例如：

```text
/scan
```

常用于激光扫描数据。

```text
/cmd_vel
```

常用于速度控制命令。

```text
/odom
```

常用于里程计数据。

---

# 10. Topic 消息类型

Topic 不只有名字。

它还有：

```text
Message Type
```

例如：

```text
/chatter
```

可能使用：

```text
std_msgs/msg/String
```

而：

```text
/cmd_vel
```

常见类型：

```text
geometry_msgs/msg/Twist
```

所以 Publisher 和 Subscriber 不仅需要 Topic 名称匹配，还需要消息类型匹配。

---

# 11. 查看 Topic

查看所有 Topic：

```bash
ros2 topic list
```

查看 Topic 类型：

```bash
ros2 topic list -t
```

查看某个 Topic：

```bash
ros2 topic info /chatter
```

查看更详细信息：

```bash
ros2 topic info /chatter -v
```

---

# 12. 查看 Topic 数据

可以直接监听 Topic：

```bash
ros2 topic echo /chatter
```

例如可能看到：

```text
data: Hello World: 1
---
data: Hello World: 2
---
```

这条命令非常适合：

```text
判断 Topic 是否真的有数据
```

---

# 13. 手动发布 Topic

ROS2 CLI 也可以手动发布消息。

例如：

```bash
ros2 topic pub /chatter std_msgs/msg/String "{data: 'hello'}"
```

这样即使不写程序，也可以快速测试：

```text
Subscriber
Topic
消息类型
```

是否工作正常。

---

# 14. Topic 通信模型

最简单：

```text
Publisher
↓
Topic
↓
Subscriber
```

复杂一些可以是：

```text
Publisher A
        \
         \
          → Topic → Subscriber A
         /        → Subscriber B
        /
Publisher B
```

因此 Topic 天然适合：

```text
广播式数据传输
传感器数据
状态数据
连续控制数据
```

---

# 15. 哪些场景适合 Topic

典型例子：

```text
激光雷达数据
相机图像
IMU 数据
里程计
速度命令
定位结果
机器人状态
```

这些数据通常具有：

```text
持续产生
不断更新
不需要请求一次才发送
```

的特点。

---

# 16. Service 是什么

Service 用于：

```text
请求一次
响应一次
```

结构：

```text
Client
↓ Request
Service Server
↓ Response
Client
```

它和 Topic 最大区别是：

```text
Topic = 持续数据流
Service = 一问一答
```

---

# 17. Service 举例

例如：

```text
请求：请打开激光雷达
响应：打开成功
```

或者：

```text
请求：当前电池电量是多少
响应：72%
```

再比如：

```text
请求：重置地图
响应：重置完成
```

这些都很适合 Service。

---

# 18. Service Server

Service Server：

```text
服务提供者
```

它等待 Client 的请求。

例如：

```text
reset_map_server
```

收到：

```text
Reset Request
```

然后执行操作，再返回：

```text
Success
```

---

# 19. Service Client

Service Client：

```text
服务请求者
```

结构：

```text
Client
↓
发送 Request
↓
Server
↓
执行
↓
返回 Response
```

---

# 20. 查看 Service

查看所有 Service：

```bash
ros2 service list
```

查看类型：

```bash
ros2 service list -t
```

查看某个 Service 类型：

```bash
ros2 service type /service_name
```

---

# 21. 调用 Service

例如：

```bash
ros2 service call /service_name package/srv/ServiceType "{...}"
```

Service 调试时，CLI 非常有用。

---

# 22. Service 的特点

Service 的核心特征：

```text
Request
Response
```

它适合：

```text
短时间完成
明确返回结果
一次性操作
```

不适合：

```text
长时间运行的任务
```

例如：

```text
导航到 500 米外的位置
```

如果这个任务需要几十秒甚至几分钟，Service 就不太合适。

这时候应该考虑：

```text
Action
```

---

# 23. Action 是什么

Action 用于：

```text
需要一段时间才能完成的任务
```

同时支持：

```text
Goal
Feedback
Result
```

也就是说：

```text
发出目标
↓
任务开始执行
↓
不断反馈执行进度
↓
最终返回结果
```

---

# 24. Action 结构

Action 通信结构：

```text
Action Client
↓ Goal
Action Server
↓
执行任务
↓
Feedback
↓
Feedback
↓
Feedback
↓
Result
```

相比 Service：

```text
Service
Request
↓
Response
```

Action 多出了：

```text
过程反馈
```

---

# 25. Action 举例

例如：

```text
导航到目标点
```

Client 发送：

```text
Goal:
x = 10
y = 20
```

Server 开始执行。

期间不断返回：

```text
Feedback:
当前距离目标还有 8 m
```

然后：

```text
Feedback:
当前距离目标还有 3 m
```

最后：

```text
Result:
到达目标
```

---

# 26. 为什么导航适合 Action

导航通常：

```text
不是瞬间完成
```

而且用户可能希望知道：

```text
当前执行到哪里了
是否还在运行
是否失败
是否完成
```

所以：

```text
Navigation
```

非常适合 Action。

---

# 27. Action 还支持取消

Action 的另一个重要能力是：

```text
Cancel
```

例如：

```text
无人机正在飞向目标点
↓
用户要求取消
↓
Action Client 发送 Cancel
↓
Action Server 停止任务
```

Service 一般没有这种天然的长任务取消机制。

---

# 28. Action Server

Action Server：

```text
负责执行任务
```

例如：

```text
navigation_server
```

收到：

```text
Goal
```

以后开始执行导航。

期间发送：

```text
Feedback
```

最后返回：

```text
Result
```

---

# 29. Action Client

Action Client：

```text
负责发送任务
```

它可以：

```text
发送 Goal
接收 Feedback
接收 Result
发送 Cancel
```

---

# 30. 查看 Action

查看：

```bash
ros2 action list
```

查看 Action 类型：

```bash
ros2 action list -t
```

查看某个 Action：

```bash
ros2 action info /action_name
```

---

# 31. Node、Topic、Service、Action 的关系

可以把它们放在一起理解：

```text
Node
│
├── Publisher
│    └── Topic
│
├── Subscriber
│    └── Topic
│
├── Service Server
│    └── Service
│
├── Service Client
│    └── Service
│
├── Action Server
│    └── Action
│
└── Action Client
     └── Action
```

所以：

```text
Topic
Service
Action
```

不是独立运行的程序。

它们通常都依附于：

```text
Node
```

---

# 32. 三种通信方式对比

| 类型 | Topic | Service | Action |
|---|---|---|---|
| 通信方式 | 持续发布 | 请求-响应 | 目标-反馈-结果 |
| 是否持续 | 是 | 否 | 任务执行期间持续 |
| 是否有返回结果 | 不强调 | 有 | 有 |
| 是否有中间反馈 | 无 | 无 | 有 |
| 是否适合长任务 | 否 | 通常不适合 | 适合 |
| 是否可取消 | 不适用 | 通常没有 | 支持 |
| 常见用途 | 传感器数据 | 查询/命令 | 导航/机械臂任务 |

---

# 33. 如何选择

可以直接按照下面判断。

如果数据是：

```text
不断产生
不断更新
```

使用：

```text
Topic
```

如果是：

```text
请求一次
得到一次结果
```

使用：

```text
Service
```

如果是：

```text
需要一段时间执行
需要查看进度
需要最终结果
可能需要取消
```

使用：

```text
Action
```

---

# 34. 用无人机系统理解

假设现在有一架无人机。

## IMU

IMU 持续输出：

```text
加速度
角速度
姿态
```

适合：

```text
Topic
```

例如：

```text
/imu/data
```

---

## 激光雷达

激光雷达不断输出点云：

```text
PointCloud
```

适合：

```text
Topic
```

例如：

```text
/livox/lidar
```

---

## 查询设备状态

例如：

```text
查询飞控是否已经解锁
```

适合：

```text
Service
```

---

## 重置定位

例如：

```text
重置定位系统
```

通常可以使用：

```text
Service
```

---

## 飞到目标点

例如：

```text
飞到 x=10, y=20, z=5
```

任务可能需要：

```text
10 秒
30 秒
几分钟
```

而且需要反馈：

```text
当前位置
剩余距离
任务状态
```

适合：

```text
Action
```

---

# 35. 用整个无人机系统表示

可以形成：

```text
                 /imu/data
IMU Node ──────────────────────→ FAST-LIO Node
                                      │
                                      │ /odom
                                      ▼
                                Planner Node
                                      │
                                      │ Goal
                                      ▼
                              Navigation Action
                                      │
                                      ▼
                               Control Node
                                      │
                                      │ /cmd_vel
                                      ▼
                                  PX4 / 飞控
```

这里同时出现了：

```text
Node
Topic
Action
```

如果还需要：

```text
重置 FAST-LIO
```

则可以增加：

```text
Service
```

---

# 36. Node 与 ROS Graph

多个 Node 和这些通信关系组合起来，就形成：

```text
ROS Graph
```

例如：

```text
imu_node
   │
   │ /imu/data
   ▼
fastlio_node
   │
   │ /odom
   ▼
planner_node
   │
   │ Action Goal
   ▼
controller_node
```

因此：

```text
Node
Topic
Service
Action
```

共同构成了 ROS Graph 的核心内容。

---

# 37. CLI 与这些实体的对应关系

Node：

```bash
ros2 node list
```

Topic：

```bash
ros2 topic list
```

Service：

```bash
ros2 service list
```

Action：

```bash
ros2 action list
```

所以 ROS2 CLI 的设计非常统一：

```text
ros2
+
对象
+
操作
```

例如：

```text
ros2 node list
ros2 topic list
ros2 service list
ros2 action list
```

---

# 38. 最核心的理解

ROS2 系统可以先理解成：

```text
很多 Node
↓
通过 Topic / Service / Action
↓
互相协作
```

其中：

```text
Node
=
功能模块
```

```text
Topic
=
持续数据流
```

```text
Service
=
一次请求 + 一次响应
```

```text
Action
=
长时间任务 + Feedback + Result + Cancel
```

如果能先把这四个概念分清，后面再学习：

```text
ROS Graph
ros2 CLI
daemon
rclcpp
rcl
RMW
DDS
QoS
```

就会容易很多。

---

# 39. 下一步

下一步需要理解：

```text
这些 Node、Topic、Service、Action
是如何被 ROS2 “看到”的？
```

这就需要继续学习：

```text
ROS Graph
```

以及：

```text
ros2 CLI
```

下一篇：

```text
04-ROS-Graph与ros2-CLI.md
```
