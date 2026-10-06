# ROS2 故障排查

在前面几篇中，我们已经把 ROS2 从应用层一路拆到了底层网络：

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
```

现在最重要的问题就是：

```text
当 ROS2 出问题时，
到底应该从哪一层开始查？
```

例如：

```text
ros2 node list 卡住怎么办？

ros2 topic list 没有输出怎么办？

Topic 能看到，但 echo 没有数据怎么办？

talker / listener 不通信怎么办？

本机通信正常，但另一台机器发现不到怎么办？

什么时候检查 daemon？

什么时候检查 RMW？

什么时候检查 DDS？

什么时候检查 QoS？

什么时候检查 Linux / WSL2 网络？
```

本篇重点就是建立一套：

```text
分层
定位
验证
逐步缩小范围
```

的 ROS2 故障排查方法。

---

# 1. ROS2 排障最重要的原则

遇到问题时，不要一开始就：

```text
重装 ROS2
```

也不要：

```text
直接修改大量配置
```

更不要：

```text
同时改 daemon、DDS、QoS、网络
```

否则最后很难知道：

```text
到底是哪一步解决了问题
```

正确思路应该是：

```text
先定位问题在哪一层
↓
再检查这一层
↓
再向上或向下缩小范围
```

---

# 2. 先建立分层思维

当前 ROS2 架构可以拆成：

```text
Application
↓
Client Library
↓
ROS2 Core
↓
RMW
↓
DDS
↓
Network
```

进一步展开：

```text
Node / Topic / Service / Action
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
Linux Network
↓
WSL2 Network
↓
Windows Network
```

不同故障：

```text
应该对应不同层次
```

---

# 3. 第一件事：不要只看最终现象

例如：

```bash
ros2 node list
```

卡住。

表面现象是：

```text
node list 不工作
```

但背后可能是：

```text
CLI
daemon
ROS Graph
RMW
DDS
Discovery
Linux Network
```

任何一层出问题。

所以：

```text
最终现象
≠
故障所在层
```

---

# 4. ROS2 故障大致分成几类

可以先分成：

```text
环境问题
```

```text
CLI / daemon 问题
```

```text
Node / Graph 问题
```

```text
Topic / QoS 问题
```

```text
RMW / DDS / Discovery 问题
```

```text
Linux / WSL2 网络问题
```

这样排查会清晰很多。

---

# 5. 第一层：检查 ROS2 环境是否加载

最先检查：

```bash
echo $ROS_DISTRO
```

正常 Humble 环境应该看到：

```text
humble
```

然后：

```bash
echo $ROS_VERSION
```

正常应该看到：

```text
2
```

如果：

```text
ROS_DISTRO
```

为空，

很可能：

```text
ROS2 环境没有 source
```

---

# 6. 手动加载 ROS2 环境

执行：

```bash
source /opt/ros/humble/setup.bash
```

然后重新检查：

```bash
echo $ROS_DISTRO
```

```bash
echo $ROS_VERSION
```

再：

```bash
which ros2
```

---

# 7. 检查 ros2 命令是否存在

执行：

```bash
which ros2
```

如果正常，

可能看到：

```text
/opt/ros/humble/bin/ros2
```

如果没有输出，

说明可能：

```text
ROS2 环境没有加载
```

或者：

```text
ROS2 安装存在问题
```

---

# 8. 检查 ROS2 CLI 是否基本正常

执行：

```bash
ros2 --help
```

如果：

```text
ros2 --help
```

都无法正常返回，

那问题还没有进入：

```text
DDS
QoS
Topic
```

这些层。

应该先解决：

```text
ROS2 安装
Python 环境
PATH
source
```

等基础问题。

---

# 9. 第二层：检查 daemon

如果：

```bash
ros2 node list
```

或者：

```bash
ros2 topic list
```

长时间没有返回，

可以先检查：

```bash
ros2 daemon status
```

---

# 10. 尝试重启 daemon

执行：

```bash
ros2 daemon stop
```

然后：

```bash
ros2 daemon start
```

再测试：

```bash
ros2 node list
```

---

# 11. 如果 daemon stop 本身也卡住

先：

```text
Ctrl + C
```

停止命令。

然后：

```bash
pkill -f _ros2_daemon
```

再：

```bash
ros2 daemon start
```

最后重新：

```bash
ros2 node list
```

---

# 12. daemon 正常不代表通信一定正常

这里必须记住：

```text
daemon
```

主要影响：

```text
CLI / Graph 查询
```

真正的数据通信：

```text
Publisher
↓
RMW
↓
DDS
↓
Subscriber
```

并不需要所有数据经过 daemon。

所以：

```text
daemon 正常
≠
ROS2 通信一定正常
```

---

# 13. daemon 异常也不代表整个 ROS2 都坏了

反过来也一样。

如果：

```bash
ros2 node list
```

异常，

但：

```text
talker → listener
```

仍然正常，

说明：

```text
ROS2 数据通信
```

可能仍然没有问题。

这时应重点检查：

```text
CLI
daemon
Graph 查询
```

---

# 14. 第三个步骤：用最小系统测试

排障时不要一开始就用：

```text
FAST-LIO
Livox
PX4
复杂 Launch
```

因为这些系统同时包含：

```text
参数
Topic
消息类型
QoS
算法
设备
驱动
网络
```

变量太多。

应该先使用：

```text
最简单 ROS2 demo
```

---

# 15. 启动 talker

终端 1：

```bash
ros2 run demo_nodes_cpp talker
```

正常情况下会看到类似：

```text
Publishing: 'Hello World: 1'
Publishing: 'Hello World: 2'
```

---

# 16. 启动 listener

终端 2：

```bash
ros2 run demo_nodes_py listener
```

如果正常，

应该看到类似：

```text
I heard: [Hello World: 1]
I heard: [Hello World: 2]
```

---

# 17. talker / listener 是非常重要的隔离测试

如果：

```text
talker
↓
listener
```

正常，

说明很多基础层已经工作：

```text
rclcpp / rclpy
rcl
RMW
DDS
基础 Discovery
本机通信
```

这时复杂项目出问题，

更可能在：

```text
项目代码
Topic
Type
QoS
参数
Launch
```

等上层。

---

# 18. 如果 talker / listener 也失败

那么问题可能更底层：

```text
ROS2 环境
RMW
DDS
Discovery
Linux Network
```

这时继续向下排查。

---

# 19. 第四层：检查 ROS Graph

先：

```bash
ros2 node list
```

看看 Node 是否存在。

例如：

```text
/talker
/listener
```

---

# 20. 如果 Node 看不到

可能原因包括：

```text
Node 没启动
Node 启动后崩溃
DDS Discovery 异常
ROS_DOMAIN_ID 不一致
RMW 异常
网络异常
```

所以先判断：

```text
Linux 进程还在吗？
```

---

# 21. Linux 进程和 ROS Node 是两个概念

可以执行：

```bash
ps aux | grep talker
```

如果进程存在，

但是：

```bash
ros2 node list
```

看不到，

说明：

```text
Process 存在
≠
成功进入 ROS Graph
```

这时应该向：

```text
RMW
DDS
Discovery
```

方向排查。

---

# 22. 如果 Node 能看到

继续：

```bash
ros2 node info /node_name
```

例如：

```bash
ros2 node info /talker
```

查看：

```text
Publisher
Subscriber
Service
Action
```

等信息。

---

# 23. 第五层：检查 Topic

执行：

```bash
ros2 topic list
```

确认目标 Topic 是否存在。

例如：

```text
/chatter
```

或者：

```text
/livox/lidar
```

---

# 24. Topic 完全看不到怎么办

如果：

```text
Node 能看到
```

但是：

```text
Topic 看不到
```

需要检查：

```text
Publisher 有没有创建
节点参数
Topic Remap
程序是否走到 create_publisher
```

也可能存在：

```text
Graph / Endpoint Discovery
```

问题。

---

# 25. 查看 Topic 类型

执行：

```bash
ros2 topic list -t
```

例如：

```text
/chatter [std_msgs/msg/String]
```

确认：

```text
Topic 名称
```

和：

```text
消息类型
```

是否符合预期。

---

# 26. 查看 Topic 详细信息

执行：

```bash
ros2 topic info /topic_name -v
```

例如：

```bash
ros2 topic info /livox/lidar -v
```

重点看：

```text
Publisher count
Subscription count
Node name
Topic type
QoS
```

---

# 27. Publisher count = 0

如果：

```text
Publisher count: 0
```

说明：

```text
没有 Publisher 在发布这个 Topic
```

这时：

```bash
ros2 topic echo /topic_name
```

当然不会有数据。

应该回到：

```text
发布节点
```

检查。

---

# 28. Subscription count = 0

如果 Publisher 存在，

但是：

```text
Subscription count: 0
```

说明当前没有 Subscriber 匹配。

可能是：

```text
Subscriber 没启动
Topic 名称不一致
Type 不一致
QoS 不兼容
```

---

# 29. 第六层：检查真实数据

执行：

```bash
ros2 topic echo /topic_name
```

例如：

```bash
ros2 topic echo /chatter
```

如果有数据，

说明：

```text
Publisher
↓
Topic
↓
Subscriber
```

基本数据链已经工作。

---

# 30. Topic 存在但 echo 没数据

这是 ROS2 中非常经典的问题。

可能原因：

```text
Publisher 实际没有 publish
QoS 不兼容
消息频率很低
数据只发布过一次
Durability 不匹配
网络 Transport 异常
```

---

# 31. 优先检查 QoS

执行：

```bash
ros2 topic info /topic_name -v
```

检查：

```text
Reliability
Durability
History
Depth
```

尤其是：

```text
Reliable
Best Effort
```

是否兼容。

---

# 32. 一个典型 QoS 问题

Publisher：

```text
Best Effort
```

Subscriber：

```text
Reliable
```

Subscriber 要求：

```text
Reliable
```

但是 Publisher 只能提供：

```text
Best Effort
```

这时可能：

```text
无法正常匹配
```

---

# 33. 为什么传感器 Topic 特别容易遇到 QoS 问题

例如：

```text
LiDAR
Camera
IMU
```

这些高频传感器数据，

经常使用：

```text
Best Effort
```

所以：

```text
ros2 topic list
```

能看到，

但：

```text
echo
```

没数据，

就要优先想到：

```text
QoS
```

---

# 34. 第七层：检查 RMW

执行：

```bash
echo $RMW_IMPLEMENTATION
```

如果没有输出：

```text
不一定代表错误
```

可能只是：

```text
使用默认 RMW
```

---

# 35. 如果显式指定 RMW

例如：

```bash
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
```

或者：

```bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
```

要确保：

```text
对应 RMW 包已经安装
```

否则 Node 可能启动失败。

---

# 36. RMW 问题通常表现在哪

可能表现为：

```text
Node 无法发现
Topic 无法发现
talker / listener 不通信
daemon 查询异常
```

如果：

```text
大量 Node 同时异常
```

相比只怀疑某个应用，

更应该关注：

```text
RMW / DDS
```

公共层。

---

# 37. 第八层：检查 DDS Discovery

如果：

```text
本机 Node 正常
```

但是：

```text
另一台机器完全发现不到
```

应该重点检查：

```text
DDS Discovery
```

---

# 38. 先检查 ROS_DOMAIN_ID

执行：

```bash
echo $ROS_DOMAIN_ID
```

所有需要互相发现的 ROS2 设备：

```text
应该使用合适且一致的 Domain
```

例如：

```text
Laptop:
ROS_DOMAIN_ID=0
```

```text
NUC:
ROS_DOMAIN_ID=0
```

---

# 39. Domain 不一致的典型表现

例如：

```text
Laptop
ROS_DOMAIN_ID=0
```

```text
NUC
ROS_DOMAIN_ID=10
```

这时：

```text
两边 ROS2 都正常
```

但：

```text
互相看不到 Node
```

这种现象很容易被误认为：

```text
网络坏了
```

---

# 40. 检查 ROS_LOCALHOST_ONLY

执行：

```bash
echo $ROS_LOCALHOST_ONLY
```

如果 ROS2 被限制为：

```text
只允许本机通信
```

则：

```text
本机 talker / listener 正常
```

但是：

```text
跨机器 Discovery 失败
```

---

# 41. 本机正常、跨机异常说明什么

如果：

```text
同一 Ubuntu 中
talker / listener 正常
```

但是：

```text
Laptop ↔ NUC
```

通信失败，

那么应该重点检查：

```text
DDS Discovery
ROS_DOMAIN_ID
ROS_LOCALHOST_ONLY
Multicast
Firewall
Linux Network
WSL2 Network
```

而不是：

```text
rclcpp 代码
```

---

# 42. 第九层：检查 Linux 网络

先：

```bash
ip addr
```

查看：

```text
网络接口
IP 地址
```

---

# 43. 查看路由

执行：

```bash
ip route
```

确认：

```text
default route
目标网段路由
```

是否合理。

---

# 44. 测试 IP 层

执行：

```bash
ping <对方IP>
```

如果：

```text
ping 都不通
```

应该先解决：

```text
基础网络
```

而不是继续研究：

```text
QoS
```

---

# 45. ping 通也不能说明 DDS 一定正常

因为：

```text
ping
```

主要测试：

```text
ICMP / IP 层
```

DDS Discovery 还可能依赖：

```text
UDP
Multicast
Firewall
DDS 配置
```

所以：

```text
ping success
≠
ROS2 Discovery success
```

---

# 46. 查看 Socket

可以执行：

```bash
ss -a
```

查看所有 Socket。

或者：

```bash
ss -u
```

查看 UDP Socket。

---

# 47. 第十层：检查 WSL2

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

如果：

```text
WSL2 内部通信正常
```

但：

```text
外部设备通信失败
```

WSL2 网络就是必须检查的一层。

---

# 48. 查看 WSL 状态

Windows PowerShell：

```powershell
wsl -l -v
```

确认：

```text
Ubuntu-22.04
```

正在：

```text
WSL2
```

模式运行。

---

# 49. 查看 Windows 网络

PowerShell：

```powershell
ipconfig
```

观察：

```text
Wi-Fi
Ethernet
WSL Virtual Adapter
VPN Adapter
```

等接口。

---

# 50. 为什么虚拟网卡很多会影响 ROS2

电脑上可能同时存在：

```text
Wi-Fi
Ethernet
WSL
Hyper-V
VPN
VMware
VirtualBox
```

DDS 需要选择网络接口。

如果接口环境复杂，

可能影响：

```text
Discovery
Multicast
Route
```

---

# 51. VPN 是一个重点检查项

如果：

```text
之前 ROS2 多机通信正常
```

打开 VPN 后：

```text
突然不能发现
```

很可能是：

```text
VPN 修改了路由
增加虚拟网卡
改变 Multicast 行为
```

---

# 52. Windows 防火墙

如果 WSL2 ROS2 需要和：

```text
NUC
另一台电脑
无人机计算机
```

通信，

还需要考虑：

```text
Windows Defender Firewall
```

可能阻止：

```text
UDP
Multicast
入站数据
```

---

# 53. WSL2 网络异常可以尝试重启

PowerShell：

```powershell
wsl --shutdown
```

然后重新：

```powershell
wsl -d Ubuntu-22.04
```

再：

```bash
source /opt/ros/humble/setup.bash
```

重新测试 ROS2。

---

# 54. 但不要把 wsl --shutdown 当成最终解决方案

如果问题来自：

```text
Firewall
ROS_DOMAIN_ID
DDS 配置
Multicast
VPN
```

那么重启之后：

```text
问题仍可能重新出现
```

所以一定要继续定位：

```text
真正原因
```

---

# 55. 一个完整的本机故障排查流程

如果：

```bash
ros2 node list
```

卡住，

可以按照下面顺序。

第一步：

```bash
echo $ROS_DISTRO
echo $ROS_VERSION
```

---

第二步：

```bash
ros2 --help
```

---

第三步：

```bash
ros2 daemon status
```

---

第四步：

```bash
ros2 daemon stop
ros2 daemon start
```

---

第五步：

如果 stop 卡住：

```bash
pkill -f _ros2_daemon
```

---

第六步：

```bash
ros2 run demo_nodes_cpp talker
```

另一个终端：

```bash
ros2 run demo_nodes_py listener
```

---

第七步：

```bash
ros2 node list
```

```bash
ros2 topic list
```

---

# 56. 如果仍然异常

继续：

```bash
echo $RMW_IMPLEMENTATION
echo $ROS_DOMAIN_ID
echo $ROS_LOCALHOST_ONLY
```

然后：

```bash
ip addr
```

```bash
ip route
```

---

# 57. 一个完整的 Topic 故障排查流程

假设：

```text
Topic 没数据
```

第一步：

```bash
ros2 node list
```

确认：

```text
Publisher Node
Subscriber Node
```

是否存在。

---

第二步：

```bash
ros2 topic list
```

确认：

```text
目标 Topic
```

是否存在。

---

第三步：

```bash
ros2 topic list -t
```

确认：

```text
消息类型
```

---

第四步：

```bash
ros2 topic info /topic_name -v
```

查看：

```text
Publisher
Subscriber
QoS
```

---

第五步：

```bash
ros2 topic echo /topic_name
```

确认：

```text
数据是否真正存在
```

---

# 58. Topic 故障判断树

```text
Topic 无数据
│
├── Node 存在吗？
│   │
│   ├── 否
│   │   ↓
│   │   Node / Discovery
│   │
│   └── 是
│       ↓
│
├── Topic 存在吗？
│   │
│   ├── 否
│   │   ↓
│   │   Publisher / Remap / Graph
│   │
│   └── 是
│       ↓
│
├── Publisher count > 0？
│
├── Subscriber count > 0？
│
├── Type 正确吗？
│
├── QoS 兼容吗？
│
└── Publisher 真的在发送吗？
```

---

# 59. 跨机器通信故障流程

假设：

```text
Laptop WSL2
↕
NUC Ubuntu
```

无法通信。

第一层：

```bash
ping <对方IP>
```

---

第二层：

```bash
echo $ROS_DOMAIN_ID
```

两边都检查。

---

第三层：

```bash
echo $ROS_LOCALHOST_ONLY
```

---

第四层：

```bash
echo $RMW_IMPLEMENTATION
```

---

第五层：

一边启动：

```bash
ros2 run demo_nodes_cpp talker
```

另一边：

```bash
ros2 run demo_nodes_py listener
```

---

# 60. 如果跨机 demo 都失败

重点检查：

```text
DDS Discovery
Multicast
Firewall
WSL2 Network
VPN
Router
```

---

# 61. 如果 demo 正常但项目不正常

这说明：

```text
底层基础通信
```

大概率正常。

应该返回：

```text
项目层
```

检查：

```text
Topic 名称
Remap
Message Type
QoS
Parameters
Launch
Namespace
```

---

# 62. Namespace 也可能导致“看起来 Topic 不见了”

例如：

```text
无人机 A
/uav1/odom
```

而你一直在找：

```text
/odom
```

就会误以为：

```text
Topic 没启动
```

所以：

```bash
ros2 topic list
```

非常重要。

---

# 63. Remap 也要检查

一个 Node 代码里可能写：

```text
/scan
```

但 Launch 中：

```text
remap
```

成：

```text
/lidar/scan
```

这时：

```text
代码默认 Topic
≠
实际运行 Topic
```

---

# 64. 参数也可能影响 Topic

例如：

```text
use_lidar = false
```

可能导致：

```text
Publisher 根本没有创建
```

或者某些节点根据参数决定：

```text
订阅哪个 Topic
```

所以复杂项目还要检查：

```bash
ros2 param list
```

---

# 65. 查看 Node 参数

执行：

```bash
ros2 param list /node_name
```

读取：

```bash
ros2 param get /node_name parameter_name
```

这对于：

```text
驱动
FAST-LIO
导航节点
```

等非常重要。

---

# 66. Launch 文件也是故障来源

一个 ROS2 Launch 文件可能同时控制：

```text
Node
Namespace
Parameter
Remap
Environment
Arguments
```

所以：

```text
Node 单独运行正常
```

但：

```text
Launch 启动异常
```

很可能问题在：

```text
Launch 配置
```

---

# 67. 推荐做“逐层简化”

例如复杂系统：

```text
Livox
+
FAST-LIO
+
Planner
+
PX4
```

出现问题。

不要一次全部运行。

可以先：

```text
Livox
```

确认点云。

然后：

```text
Livox + FAST-LIO
```

确认里程计。

然后：

```text
+ Planner
```

最后：

```text
+ PX4
```

---

# 68. 这种方法叫隔离变量

每次只增加：

```text
一个模块
```

就更容易知道：

```text
从哪一步开始坏
```

这是比：

```text
一次全启动
然后看一堆报错
```

更高效的方法。

---

# 69. 用 FAST-LIO 举例

假设：

```text
FAST-LIO 没有输出 Odometry
```

不要直接认为：

```text
FAST-LIO 算法坏了
```

先检查：

```bash
ros2 node list
```

有没有：

```text
Livox Node
FAST-LIO Node
```

---

# 70. 然后检查输入 Topic

```bash
ros2 topic list
```

确认：

```text
/livox/lidar
```

存在。

---

# 71. 再看连接关系

```bash
ros2 topic info /livox/lidar -v
```

检查：

```text
Publisher
Subscriber
QoS
Type
```

---

# 72. 再看真实数据

```bash
ros2 topic echo /livox/lidar
```

如果：

```text
没有点云
```

说明问题还没有到：

```text
FAST-LIO 算法
```

而是在：

```text
Livox Driver / Topic / QoS / 网络
```

更前面。

---

# 73. 如果点云正常但 FAST-LIO 没输出

继续：

```bash
ros2 node info /fastlio_node
```

检查：

```text
FAST-LIO 是否真的订阅正确 Topic
```

以及：

```text
参数
消息类型
时间戳
TF
```

等上层问题。

---

# 74. 这就是“从输入向输出排查”

一个节点可以抽象成：

```text
Input
↓
Node
↓
Output
```

所以检查顺序：

```text
输入有没有
↓
节点有没有运行
↓
连接关系对不对
↓
输出有没有
```

非常适合 ROS2。

---

# 75. ros2 doctor

ROS2 还提供：

```bash
ros2 doctor --report
```

可以查看：

```text
ROS2 配置
环境
平台
Middleware
网络相关信息
```

它适合作为：

```text
辅助诊断工具
```

---

# 76. 但 ros2 doctor 不能替代分层排障

不要认为：

```text
ros2 doctor 没报错
=
系统一定正常
```

很多实际问题：

```text
Topic QoS
应用代码
网络设备
路由器 Multicast
```

仍然需要自己判断。

---

# 77. 查看日志也非常重要

ROS2 节点通常会输出：

```text
INFO
WARN
ERROR
FATAL
```

不要忽略：

```text
WARN
ERROR
```

因为很多问题其实已经明确写在日志里。

---

# 78. 一看到 Error 不要只看最后一行

例如一个节点启动失败，

最后一行可能只是：

```text
process has died
```

真正原因可能在前面几十行：

```text
library not found
parameter invalid
topic type mismatch
permission denied
```

所以日志要：

```text
从第一个明显异常开始看
```

---

# 79. 环境污染问题

如果同时安装：

```text
ROS1 Noetic
ROS2 Humble
```

需要特别小心：

```text
source 顺序
环境变量
工作空间 overlay
```

例如错误地混合：

```text
ROS1
+
ROS2
```

环境，

可能产生很难理解的问题。

---

# 80. 检查当前 ROS 版本

```bash
echo $ROS_VERSION
```

ROS1：

```text
1
```

ROS2：

```text
2
```

当前 Humble 应该是：

```text
2
```

---

# 81. 检查 ROS_DISTRO

```bash
echo $ROS_DISTRO
```

当前应看到：

```text
humble
```

如果看到：

```text
noetic
```

说明当前终端环境可能并不是预期的 ROS2 环境。

---

# 82. Workspace overlay 也会影响结果

以后创建工作空间：

```text
ros2_ws
```

并执行：

```bash
source install/setup.bash
```

之后，

当前环境会变成：

```text
/opt/ros/humble
+
ros2_ws
```

也就是：

```text
Overlay Workspace
```

如果 source 错工作空间，

也可能导致：

```text
Package 找不到
版本不一致
消息类型异常
```

---

# 83. 一个推荐的环境检查命令组

遇到奇怪问题时：

```bash
echo $ROS_VERSION
echo $ROS_DISTRO
echo $RMW_IMPLEMENTATION
echo $ROS_DOMAIN_ID
echo $ROS_LOCALHOST_ONLY
```

再：

```bash
which ros2
```

这样可以快速知道：

```text
当前到底处在什么 ROS2 环境
```

---

# 84. 不要同时修改太多变量

例如通信失败时，

不要一次：

```text
换 RMW
改 Domain
关闭 Firewall
改 DDS XML
改 QoS
```

然后发现：

```text
好了
```

因为你不知道：

```text
到底是什么原因
```

正确方式：

```text
一次只修改一个变量
↓
测试
↓
记录结果
```

---

# 85. 推荐记录排障过程

例如：

```text
问题：
ros2 node list 卡住

测试 1：
daemon restart
结果：无效

测试 2：
talker/listener
结果：正常

结论：
优先怀疑 CLI / daemon / Graph 查询
```

这样：

```text
问题会越来越明确
```

---

# 86. 一个标准 ROS2 排障模板

以后遇到任何问题，

可以先回答这几个问题：

```text
1. Node 在吗？
2. Topic 在吗？
3. Publisher 在吗？
4. Subscriber 在吗？
5. Type 对吗？
6. QoS 兼容吗？
7. 数据真的在发吗？
8. Discovery 正常吗？
9. 网络正常吗？
10. 环境变量正确吗？
```

---

# 87. 一个更完整的分层模板

```text
Application
│
├── 程序是否运行？
├── 参数是否正确？
├── Topic 名是否正确？
└── Remap 是否正确？
        ↓
Client Library
│
├── rclcpp / rclpy 是否正常？
        ↓
ROS Core
│
├── Node 是否进入 Graph？
        ↓
RMW
│
├── RMW 是否正确？
        ↓
DDS
│
├── Discovery 是否成功？
├── Endpoint 是否匹配？
├── QoS 是否兼容？
        ↓
Linux Network
│
├── IP 是否正常？
├── Route 是否正常？
├── UDP 是否正常？
        ↓
WSL2 / Windows
│
├── Virtual NIC
├── Firewall
├── VPN
└── Multicast
```

---

# 88. 最推荐的故障定位顺序

对于大部分 ROS2 问题，

建议：

```text
第 1 步
检查环境
```

```text
第 2 步
检查 Node
```

```text
第 3 步
检查 Topic
```

```text
第 4 步
检查 Publisher / Subscriber
```

```text
第 5 步
检查 QoS
```

```text
第 6 步
测试 demo_nodes
```

```text
第 7 步
检查 daemon
```

```text
第 8 步
检查 RMW / DDS
```

```text
第 9 步
检查 Linux Network
```

```text
第 10 步
检查 WSL2 / Windows
```

---

# 89. 为什么不是永远先查 daemon

因为：

```text
Topic 没数据
```

可能只是：

```text
Publisher 没启动
```

这种情况下重启：

```text
daemon
```

没有意义。

所以排障必须根据：

```text
实际现象
```

选择入口。

---

# 90. 不同现象应该从哪里开始

如果：

```text
ros2 命令不存在
```

查：

```text
环境 / 安装
```

如果：

```text
ros2 node list 卡住
```

查：

```text
CLI / daemon / Discovery
```

如果：

```text
Node 看不到
```

查：

```text
Node / Discovery
```

如果：

```text
Topic 看得到但没数据
```

查：

```text
Publisher / QoS / Transport
```

如果：

```text
本机正常跨机失败
```

查：

```text
DDS / Network / WSL2 / Firewall
```

---

# 91. 最终故障定位表

| 现象 | 优先检查 |
|---|---|
| `ros2` 命令不存在 | source / ROS2 安装 / PATH |
| `ros2 node list` 卡住 | daemon / RMW / DDS / Discovery |
| Node 进程在但 Graph 看不到 | Discovery / Domain / RMW |
| Topic 不存在 | Publisher / Remap / Node |
| Topic 存在但无 Publisher | 发布节点 |
| Topic 存在但 echo 无数据 | QoS / Publisher / Transport |
| talker/listener 本机失败 | ROS2 / RMW / DDS |
| 本机正常跨机失败 | Domain / DDS / Multicast / Firewall / WSL2 |
| FAST-LIO 无输入 | LiDAR Topic / Type / QoS |
| CLI 异常但数据正常 | daemon / Graph 查询 |

---

# 92. 当前架构与排障的关系

学习前面这些架构并不是为了：

```text
记名词
```

真正目的就是：

```text
出现问题时
知道下一步应该查哪里
```

例如：

```text
ros2 node list 卡住
```

现在已经可以想到：

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
Network
```

然后逐层验证。

---

# 93. 本篇核心总结

第一：

```text
ROS2 排障的核心
=
分层定位
```

第二：

```text
不要看到最终现象
就直接判断根因
```

第三：

```text
Node
↓
Topic
↓
Endpoint
↓
QoS
↓
DDS
↓
Network
```

是非常重要的排障链。

第四：

```text
talker / listener
```

是非常重要的：

```text
最小系统测试
```

第五：

```text
本机正常
跨机器失败
```

优先考虑：

```text
DDS Discovery
Domain
Multicast
Firewall
WSL2 Network
```

---

# 94. 最重要的一套命令

ROS2 环境：

```bash
echo $ROS_VERSION
echo $ROS_DISTRO
echo $RMW_IMPLEMENTATION
echo $ROS_DOMAIN_ID
echo $ROS_LOCALHOST_ONLY
```

Graph：

```bash
ros2 node list
ros2 topic list
```

Topic：

```bash
ros2 topic info /topic_name -v
ros2 topic echo /topic_name
```

daemon：

```bash
ros2 daemon status
ros2 daemon stop
ros2 daemon start
```

强制结束：

```bash
pkill -f _ros2_daemon
```

网络：

```bash
ip addr
ip route
ping <IP>
ss -u
```

Windows：

```powershell
ipconfig
wsl -l -v
wsl --shutdown
```

---

# 95. 整个 ROS2 学习链已经连接起来

现在可以把前面的所有内容串成：

```text
ROS1 与 ROS2
↓
ROS2 Architecture
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
Linux Network
↓
WSL2 Network
↓
Troubleshooting
```

这已经形成了一套完整的：

```text
ROS2 通信架构学习路径
```

---

# 96. 最终应该形成的思维

以后遇到 ROS2 问题时，

不要只想：

```text
ROS2 为什么坏了？
```

而应该想：

```text
到底是哪一层坏了？
```

然后：

```text
先验证这一层
↓
再检查下一层
↓
逐步缩小范围
```

这就是整个 ROS2 架构学习最重要的实际意义。
