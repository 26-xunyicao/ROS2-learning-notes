# rclcpp、rclpy 与 rcl

在上一篇中，我们已经知道：

```text
Node / Topic / Service / Action
↓
ROS Graph
↓
ros2 CLI
↓
daemon
```

同时也知道：

```text
真正的 ROS2 数据通信主链
```

更接近：

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

这里就出现了一个非常关键的问题：

```text
rclcpp 是什么？

rclpy 是什么？

rcl 又是什么？

为什么 C++ 和 Python
最终都能进入同一套 ROS2 通信系统？
```

本篇重点就是搞清楚：

```text
C++ / Python
↓
rclcpp / rclpy
↓
rcl
↓
RMW
```

这条链路到底是什么逻辑。

---

# 1. 为什么 ROS2 需要 Client Library

ROS2 并不是只支持一种编程语言。

我们可以使用：

```text
C++
Python
```

来编写 ROS2 程序。

例如：

```text
C++
↓
rclcpp
```

Python：

```text
Python
↓
rclpy
```

所以：

```text
rclcpp
rclpy
```

都属于 ROS2 的：

```text
Client Library
```

也就是：

```text
客户端库
```

它们主要负责：

```text
给不同编程语言
提供 ROS2 API
```

---

# 2. rclcpp 是什么

`rclcpp` 可以理解为：

```text
ROS2 的 C++ Client Library
```

也就是：

```text
C++
↓
rclcpp
↓
ROS2
```

当你用 C++ 编写 ROS2 Node 时，经常会看到：

```cpp
#include <rclcpp/rclcpp.hpp>
```

这说明：

```text
程序正在使用 rclcpp
```

---

# 3. 一个最简单的 rclcpp Node

例如：

```cpp
#include <rclcpp/rclcpp.hpp>

int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv);

    auto node = rclcpp::Node::make_shared("my_node");

    rclcpp::spin(node);

    rclcpp::shutdown();

    return 0;
}
```

这里最核心的几个部分是：

```text
rclcpp::init()
rclcpp::Node
rclcpp::spin()
rclcpp::shutdown()
```

它们都来自：

```text
rclcpp
```

---

# 4. rclcpp::init()

程序启动以后：

```cpp
rclcpp::init(argc, argv);
```

主要用于初始化 ROS2。

可以简单理解为：

```text
启动 ROS2 Client Library 环境
```

后续才能正常创建：

```text
Node
Publisher
Subscriber
Service
Action
```

等 ROS2 实体。

---

# 5. rclcpp::Node

例如：

```cpp
auto node = rclcpp::Node::make_shared("my_node");
```

这里创建：

```text
my_node
```

这个 ROS2 Node。

也就是说：

```text
C++ 程序
↓
调用 rclcpp
↓
创建 ROS2 Node
```

---

# 6. rclcpp::spin()

例如：

```cpp
rclcpp::spin(node);
```

它的作用可以先简单理解为：

```text
让 Node 持续运行
并处理 ROS2 事件
```

这些事件可能包括：

```text
收到 Topic 消息
收到 Service 请求
Timer 到期
Action 事件
```

所以：

```text
spin
```

是 ROS2 程序运行机制中的重要概念。

---

# 7. rclcpp::shutdown()

当程序准备退出时：

```cpp
rclcpp::shutdown();
```

用于：

```text
关闭 ROS2 Client Library
释放相关资源
```

---

# 8. rclpy 是什么

`rclpy` 可以理解为：

```text
ROS2 的 Python Client Library
```

结构：

```text
Python
↓
rclpy
↓
ROS2
```

如果使用 Python 编写 ROS2 Node，经常会看到：

```python
import rclpy
```

---

# 9. 一个最简单的 rclpy Node

例如：

```python
import rclpy
from rclpy.node import Node

def main(args=None):

    rclpy.init(args=args)

    node = Node('my_node')

    rclpy.spin(node)

    node.destroy_node()

    rclpy.shutdown()
```

可以看到 Python 版本和 C++ 版本非常相似。

---

# 10. C++ 与 Python 的对应关系

C++：

```cpp
rclcpp::init()
```

Python：

```python
rclpy.init()
```

C++：

```cpp
rclcpp::Node
```

Python：

```python
rclpy.node.Node
```

C++：

```cpp
rclcpp::spin()
```

Python：

```python
rclpy.spin()
```

C++：

```cpp
rclcpp::shutdown()
```

Python：

```python
rclpy.shutdown()
```

所以：

```text
rclcpp
```

和：

```text
rclpy
```

虽然语言不同，

但提供的 ROS2 核心能力非常相似。

---

# 11. rclcpp / rclpy 提供什么

它们主要向开发者提供：

```text
Node
Publisher
Subscriber
Service
Client
Action
Timer
Parameter
Executor
QoS
Logging
```

等高级 ROS2 API。

所以对于普通 ROS2 开发者来说：

```text
rclcpp
```

或者：

```text
rclpy
```

就是我们最直接接触的 ROS2 编程层。

---

# 12. 创建 Publisher

以 C++ 为例：

```cpp
publisher_ = this->create_publisher<std_msgs::msg::String>(
    "chatter",
    10
);
```

这句话表示：

```text
当前 Node
↓
创建 Publisher
↓
Topic = chatter
↓
Message Type = String
```

这里开发者操作的是：

```text
rclcpp API
```

---

# 13. 创建 Subscriber

例如：

```cpp
subscription_ = this->create_subscription<std_msgs::msg::String>(
    "chatter",
    10,
    callback
);
```

表示：

```text
当前 Node
↓
创建 Subscriber
↓
订阅 chatter
↓
收到消息后调用 callback
```

---

# 14. Python 中也是类似逻辑

例如：

```python
self.publisher_ = self.create_publisher(
    String,
    'chatter',
    10
)
```

Subscriber：

```python
self.subscription = self.create_subscription(
    String,
    'chatter',
    self.listener_callback,
    10
)
```

虽然语法不同，

但是背后的 ROS2 逻辑相同：

```text
Node
↓
Publisher / Subscriber
↓
Topic
```

---

# 15. 为什么不能让 C++ 和 Python 直接操作 DDS

这里就进入 ROS2 分层设计的核心。

假设没有 Client Library。

那么开发者可能需要直接写：

```text
DDS Participant
DDS Publisher
DDS DataWriter
DDS Subscriber
DDS DataReader
```

甚至还要处理：

```text
DDS QoS
DDS Discovery
序列化
类型支持
```

这对普通机器人开发来说非常复杂。

因此 ROS2 提供：

```text
rclcpp
rclpy
```

把底层复杂性封装起来。

---

# 16. rclcpp / rclpy 是“高级 API 层”

可以理解为：

```text
机器人开发者
↓
rclcpp / rclpy
↓
ROS2 高级 API
```

开发者只需要说：

```text
我要创建一个 Publisher
```

而不需要自己处理：

```text
DDS DataWriter
DDS Participant
网络 Socket
```

这些底层细节。

---

# 17. 那为什么还需要 rcl

如果已经有：

```text
rclcpp
rclpy
```

为什么下面还要有：

```text
rcl
```

？

这是 ROS2 架构中非常重要的一层。

因为 ROS2 不希望：

```text
每种编程语言
都重新实现一整套 ROS2 底层逻辑
```

否则就可能变成：

```text
rclcpp
↓
自己实现 ROS2 核心

rclpy
↓
再实现一次 ROS2 核心
```

这样会造成大量重复代码。

---

# 18. rcl 是什么

`rcl` 可以理解为：

```text
ROS Client Library 的公共核心层
```

它主要使用：

```text
C
```

实现。

所以架构可以表示为：

```text
C++
↓
rclcpp
↓
rcl
```

Python：

```text
Python
↓
rclpy
↓
rcl
```

然后继续：

```text
rcl
↓
RMW
```

---

# 19. 为什么 rcl 使用 C

C 有一个非常重要的优势：

```text
容易被其他语言调用
```

所以 ROS2 可以把很多核心逻辑放在：

```text
rcl
```

这一层。

然后：

```text
C++ → rclcpp
Python → rclpy
```

再去使用这些公共能力。

---

# 20. rcl 的核心作用

可以简单理解成：

```text
把不同语言的 Client Library
统一到 ROS2 公共核心
```

比如：

```text
创建 Node
创建 Publisher
创建 Subscriber
处理 ROS Graph
处理参数
处理 Guard Condition
处理 Wait Set
```

等底层公共能力。

---

# 21. rclcpp 和 rcl 的关系

假设 C++ 程序调用：

```cpp
create_publisher()
```

大致逻辑可以理解成：

```text
C++ 用户代码
↓
rclcpp
↓
rcl
↓
RMW
↓
DDS
```

所以：

```text
rclcpp
```

主要负责：

```text
C++ 风格 API
对象封装
模板
Executor
Callback
```

而：

```text
rcl
```

负责更公共、更底层的 ROS2 Client 功能。

---

# 22. rclpy 和 rcl 的关系

Python 同样可以理解成：

```text
Python 用户代码
↓
rclpy
↓
rcl
↓
RMW
↓
DDS
```

所以：

```text
Python
```

和：

```text
C++
```

最终能够进入同一个 ROS2 网络，

是因为它们最终都会走向：

```text
rcl
↓
RMW
↓
DDS
```

这条统一的底层链路。

---

# 23. 为什么 C++ Node 能和 Python Node 通信

这是一个很好的问题。

例如：

C++：

```text
talker
```

Python：

```text
listener
```

为什么能互相通信？

因为：

```text
C++ talker
↓
rclcpp
↓
rcl
↓
RMW
↓
DDS
```

Python listener：

```text
Python listener
↓
rclpy
↓
rcl
↓
RMW
↓
DDS
```

最终：

```text
都进入 DDS 通信体系
```

所以编程语言不同并不影响：

```text
ROS2 通信
```

---

# 24. 一个完整例子

假设：

```text
C++ Node
```

发布：

```text
/chatter
```

Python Node：

```text
订阅 /chatter
```

完整链路：

```text
C++ Application
↓
rclcpp
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
rcl
↓
rclpy
↓
Python Application
```

所以真正让：

```text
C++
```

和：

```text
Python
```

互通的不是：

```text
语言本身
```

而是：

```text
统一 ROS2 中间件体系
```

---

# 25. rclcpp / rclpy 与 ROS Graph

当你创建：

```text
Node
Publisher
Subscriber
```

这些实体以后，

它们最终会进入：

```text
ROS Graph
```

例如：

```text
rclcpp
↓
创建 Node
↓
rcl
↓
RMW
↓
DDS
↓
Discovery
↓
其他 Node 发现它
```

于是：

```bash
ros2 node list
```

就可能看到这个 Node。

---

# 26. Node 创建到底发生了什么

表面上你可能只写：

```cpp
rclcpp::Node::make_shared("lidar_node");
```

但底层并不是只：

```text
创建一个 C++ 对象
```

而是会逐步建立 ROS2 所需的：

```text
Node 信息
RMW 实体
DDS Participant / Endpoint
Graph 信息
```

所以：

```text
创建一个 Node
```

背后实际上会跨越多个 ROS2 层。

---

# 27. Publisher 创建也是一样

你写：

```cpp
create_publisher(...)
```

表面是：

```text
创建 ROS2 Publisher
```

底层大致会继续：

```text
rclcpp
↓
rcl
↓
RMW
↓
DDS Publisher / DataWriter
```

最终才进入 DDS 通信体系。

---

# 28. Subscriber 创建过程

同样：

```text
rclcpp / rclpy
↓
创建 Subscriber
↓
rcl
↓
RMW
↓
DDS Subscriber / DataReader
```

然后：

```text
DDS Discovery
```

发现可以匹配的 Publisher。

---

# 29. rcl 与 RMW 的边界

这里是非常重要的架构边界。

上面：

```text
rcl
```

仍然属于：

```text
ROS2 Client Library 核心
```

下面：

```text
RMW
```

开始进入：

```text
Middleware 抽象层
```

所以：

```text
rcl
↓
RMW
```

可以看成：

```text
ROS2 核心
↓
中间件接口
```

之间的重要分界线。

---

# 30. rcl 不直接绑定 DDS

这是 ROS2 设计中的关键点。

如果：

```text
rcl
```

直接写死某一种 DDS，

结构就会变成：

```text
rcl
↓
Fast DDS
```

那 ROS2 就很难更换中间件。

因此设计成：

```text
rcl
↓
RMW
↓
具体 DDS 实现
```

这样：

```text
rcl
```

只需要调用：

```text
RMW 接口
```

不需要关心：

```text
下面到底是哪一种 DDS
```

---

# 31. 这也是为什么下一层是 RMW

目前我们已经走到了：

```text
Application
↓
rclcpp / rclpy
↓
rcl
```

下一层：

```text
RMW
```

就是专门解决：

```text
ROS2 怎么和不同 Middleware 对接
```

的问题。

---

# 32. Executor 是什么

在 rclcpp / rclpy 中还有一个非常重要的概念：

```text
Executor
```

它负责处理：

```text
Callback
```

例如：

```text
Subscriber Callback
Timer Callback
Service Callback
Action Callback
```

可以简单理解：

```text
Executor
=
负责调度 ROS2 回调执行
```

---

# 33. spin 与 Executor 的关系

之前看到：

```cpp
rclcpp::spin(node);
```

或者：

```python
rclpy.spin(node)
```

本质上和：

```text
Executor 持续等待事件
↓
事件到来
↓
执行 Callback
```

有关。

例如：

```text
Topic 消息到达
↓
Executor 发现事件
↓
调用 Subscriber Callback
```

---

# 34. Subscriber 为什么需要 callback

例如：

```text
/imu/data
```

来了新消息。

ROS2 需要知道：

```text
收到消息以后
应该执行哪个函数？
```

所以开发者会提供：

```text
callback
```

例如：

```cpp
void imu_callback(...)
{
    ...
}
```

Executor 会负责：

```text
调度这个 callback
```

---

# 35. Timer 也是类似

例如：

```text
每 100 ms
发布一次消息
```

可以创建：

```text
Timer
```

Timer 到期以后：

```text
Executor
↓
执行 Timer Callback
↓
Publisher 发布消息
```

---

# 36. Executor 为什么重要

在简单 demo 中：

```text
Executor
```

可能感觉不明显。

但在复杂无人机系统中：

```text
IMU Callback
LiDAR Callback
Timer Callback
Service Callback
Action Callback
```

可能同时存在。

这时：

```text
谁先执行
谁在哪个线程执行
是否会互相阻塞
```

就非常重要。

这些都与：

```text
Executor
```

有关。

---

# 37. rclcpp 比 rcl 多了什么

可以粗略理解为：

```text
rcl
```

更底层、更通用。

而：

```text
rclcpp
```

在它上面增加：

```text
C++ 对象封装
模板
智能指针
Executor
Callback
QoS 类
Node 类
Publisher 类
Subscriber 类
```

所以开发体验更友好。

---

# 38. rclpy 又做了什么

`rclpy` 则把 ROS2 能力包装成：

```text
Python 风格 API
```

例如：

```python
Node(...)
```

```python
create_publisher(...)
```

```python
create_subscription(...)
```

这样 Python 开发者不需要直接使用：

```text
C 接口
```

---

# 39. 为什么不是所有东西都直接放在 rcl

因为：

```text
不同语言有不同的编程习惯
```

例如 C++：

```text
class
template
shared_ptr
```

Python：

```text
dynamic typing
Python object
Python callback
```

如果全部塞进：

```text
rcl
```

就无法很好地提供语言原生体验。

所以 ROS2 采用：

```text
语言层
↓
公共核心层
```

的设计。

---

# 40. 整体结构

现在可以画成：

```text
                   ROS2 Application
                          │
              ┌───────────┴───────────┐
              │                       │
              │                       │
             C++                    Python
              │                       │
           rclcpp                   rclpy
              │                       │
              └───────────┬───────────┘
                          │
                         rcl
                          │
                         RMW
                          │
                         DDS
                          │
                       Network
```

这张图非常重要。

---

# 41. 用无人机系统理解

例如：

```text
FAST-LIO
```

可能是 C++ 编写。

那么：

```text
FAST-LIO
↓
rclcpp
↓
rcl
↓
RMW
↓
DDS
```

而某个简单控制脚本：

```text
Python Control Node
```

可能使用：

```text
rclpy
```

于是：

```text
Python Control Node
↓
rclpy
↓
rcl
↓
RMW
↓
DDS
```

两者最终可以通过：

```text
Topic
Service
Action
```

正常通信。

---

# 42. C++ 和 Python 的性能区别

通常：

```text
C++
```

在：

```text
性能
实时性
资源控制
```

方面更有优势。

而：

```text
Python
```

通常在：

```text
开发速度
测试
脚本
算法验证
```

方面更方便。

所以无人机和机器人系统中经常出现：

```text
底层高性能节点
→ C++

上层逻辑 / 测试工具
→ Python
```

但它们都可以进入同一个：

```text
ROS Graph
```

---

# 43. rclcpp 和 rclpy 是否能直接互相调用

通常不是：

```text
rclcpp
直接调用
rclpy
```

而是：

```text
rclcpp
↓
ROS2 通信体系
↑
rclpy
```

它们通过：

```text
Topic
Service
Action
```

等 ROS2 通信机制协作。

---

# 44. 一个常见误区

不要认为：

```text
rclcpp = ROS2 C++ 版

rclpy = ROS2 Python 版

它们是两套独立 ROS2
```

不是。

更准确地说：

```text
rclcpp
```

和：

```text
rclpy
```

只是：

```text
不同语言进入同一个 ROS2 核心体系的入口
```

---

# 45. 另一个常见误区

也不要认为：

```text
rcl
=
DDS
```

不是。

层级是：

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
ROS2 Client Library 核心层
```

```text
RMW
=
Middleware 抽象接口
```

```text
DDS
=
底层通信中间件体系
```

---

# 46. 再看完整的数据发送流程

假设：

```text
C++ talker
```

发布：

```text
/chatter
```

完整逻辑：

```text
用户代码
↓
rclcpp Publisher
↓
rcl
↓
RMW
↓
DDS DataWriter
↓
Network
```

接收端：

```text
Network
↓
DDS DataReader
↓
RMW
↓
rcl
↓
rclpy / rclcpp Subscriber
↓
Callback
↓
用户代码
```

---

# 47. ros2 CLI 和这一层有什么关系

前面已经知道：

```text
ros2 CLI
↓
daemon
↓
ROS Graph
```

但是 daemon 本身最终也需要：

```text
ROS2 Client Library
RMW
DDS
```

参与 ROS2 网络。

所以：

```text
CLI / daemon
```

和：

```text
rcl / RMW / DDS
```

并不是完全割裂的。

---

# 48. 为什么理解这一层对排障有帮助

例如：

```text
Python Node 启动失败
```

那么问题可能发生在：

```text
Python 代码
↓
rclpy
↓
ROS2 环境
```

而：

```text
多个语言的 Node 都无法通信
```

则可能需要继续向下检查：

```text
rcl
RMW
DDS
网络
```

---

# 49. 可以按照层次判断问题

例如：

```text
只有一个 Python Node 异常
```

优先考虑：

```text
Python 代码
rclpy
依赖
```

如果：

```text
C++ 和 Python Node 都异常
```

则应该考虑：

```text
更公共的底层
```

例如：

```text
rcl
RMW
DDS
Network
```

---

# 50. rclcpp / rclpy / rcl 在架构中的位置

现在放回整个 ROS2 架构：

```text
ROS2 Application
│
├── C++ Node
│      ↓
│    rclcpp
│
├── Python Node
│      ↓
│    rclpy
│
└────────┬────────
         ↓
        rcl
         ↓
        RMW
         ↓
        DDS
         ↓
 Linux Network Stack
```

---

# 51. 三者的核心区别

可以直接记成：

```text
rclcpp
=
C++ ROS2 API
```

```text
rclpy
=
Python ROS2 API
```

```text
rcl
=
ROS2 公共 Client Library 核心
```

---

# 52. 最重要的关系

最重要的一条链：

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

最后统一：

```text
rcl
↓
RMW
↓
DDS
```

---

# 53. 本篇核心总结

第一：

```text
rclcpp
=
ROS2 的 C++ Client Library
```

第二：

```text
rclpy
=
ROS2 的 Python Client Library
```

第三：

```text
rcl
=
它们下面共享的 ROS2 Client Library 核心层
```

因此：

```text
C++ Node
```

和：

```text
Python Node
```

虽然使用不同语言，

但最终都会进入：

```text
rcl
↓
RMW
↓
DDS
```

同一套 ROS2 通信体系。

---

# 54. 下一篇

现在我们已经把通信链继续向下拆到：

```text
Application
↓
rclcpp / rclpy
↓
rcl
```

接下来马上会遇到：

```text
RMW
```

那么新的问题就是：

```text
为什么 rcl 不直接连接 DDS？

为什么还需要中间再加一层 RMW？

RMW 到底做了什么？

为什么 ROS2 可以更换不同 DDS 实现？
```

下一篇继续拆解：

```text
07-RMW.md
```
