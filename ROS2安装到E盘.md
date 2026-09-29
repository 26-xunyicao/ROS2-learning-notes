# Windows 11 + WSL2：在 E 盘安装 Ubuntu 22.04 + ROS2 Humble

本文记录如何在 Windows 11 下，通过 WSL2 将 Ubuntu 22.04 安装到 E 盘，并在其中安装 ROS2 Humble。

最终结构：

```text
Windows 11
└── WSL2
    └── Ubuntu 22.04
        └── ROS2 Humble
```

目标安装位置：

```text
E:\WSL\Ubuntu22.04
```

---

## 1. 查看当前 WSL 状态

打开 Windows PowerShell：

```powershell
wsl --status
```

查看已经安装的发行版：

```powershell
wsl -l -v
```

查看可以安装的 Linux 发行版：

```powershell
wsl -l -o
```

找到：

```text
Ubuntu-22.04
```

---

## 2. 在 E 盘创建 WSL 目录

执行：

```powershell
mkdir E:\WSL
```

然后：

```powershell
mkdir E:\WSL\Ubuntu22.04
```

最终目录：

```text
E:
└── WSL
    └── Ubuntu22.04
```

---

## 3. 将 Ubuntu 22.04 直接安装到 E 盘

执行：

```powershell
wsl --install -d Ubuntu-22.04 --location E:\WSL\Ubuntu22.04
```

参数含义：

```text
-d Ubuntu-22.04
```

表示安装 Ubuntu 22.04。

```text
--location E:\WSL\Ubuntu22.04
```

表示将该 WSL 发行版直接安装到 E 盘。

等待安装完成。

---

## 4. 检查是否安装成功

执行：

```powershell
wsl -l -v
```

应该可以看到类似：

```text
NAME            STATE           VERSION
Ubuntu-22.04    Stopped         2
```

---

## 5. 启动 Ubuntu 22.04

执行：

```powershell
wsl -d Ubuntu-22.04
```

第一次启动时，会要求创建 Linux 用户。

例如：

```text
Enter new UNIX username:
```

输入用户名。

然后：

```text
New password:
```

输入密码。

注意：

Linux 输入密码时不会显示：

```text
*
```

也不会显示其他字符。

这是正常现象。

---

## 6. 检查 Ubuntu 版本

进入 Ubuntu 后执行：

```bash
lsb_release -a
```

应该看到：

```text
Ubuntu 22.04
```

也可以：

```bash
cat /etc/os-release
```

---

## 7. 更新 Ubuntu

执行：

```bash
sudo apt update
```

然后：

```bash
sudo apt upgrade -y
```

---

## 8. 配置 UTF-8 Locale

执行：

```bash
sudo apt install locales -y
```

生成语言环境：

```bash
sudo locale-gen en_US en_US.UTF-8
```

设置默认语言环境：

```bash
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
```

当前终端设置：

```bash
export LANG=en_US.UTF-8
```

检查：

```bash
locale
```

---

## 9. 启用 Universe 软件源

执行：

```bash
sudo apt install software-properties-common -y
```

启用：

```bash
sudo add-apt-repository universe
```

更新：

```bash
sudo apt update
```

---

## 10. 安装 curl

执行：

```bash
sudo apt install curl -y
```

---

## 11. 添加 ROS2 密钥

执行：

```bash
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
-o /usr/share/keyrings/ros-archive-keyring.gpg
```

---

## 12. 添加 ROS2 软件源

执行：

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

然后更新：

```bash
sudo apt update
```

---

## 13. 安装 ROS2 Humble Desktop

执行：

```bash
sudo apt install ros-humble-desktop -y
```

这个版本包含：

```text
ROS2 基础组件
rviz2
常用工具
示例节点
常用消息包
```

---

## 14. 安装 ROS2 开发工具

执行：

```bash
sudo apt install ros-dev-tools -y
```

---

## 15. 加载 ROS2 环境

执行：

```bash
source /opt/ros/humble/setup.bash
```

检查：

```bash
echo $ROS_DISTRO
```

应该输出：

```text
humble
```

再检查：

```bash
echo $ROS_VERSION
```

应该输出：

```text
2
```

---

## 16. 设置自动加载 ROS2 环境

执行：

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

然后：

```bash
source ~/.bashrc
```

再次检查：

```bash
echo $ROS_DISTRO
```

应该输出：

```text
humble
```

---

## 17. 检查 ros2 命令

执行：

```bash
which ros2
```

通常应该显示：

```text
/opt/ros/humble/bin/ros2
```

检查：

```bash
ros2 --help
```

如果能正常显示帮助信息，说明 ROS2 CLI 已安装成功。

---

## 18. 测试 talker

打开一个 Ubuntu 终端：

```bash
ros2 run demo_nodes_cpp talker
```

正常会持续输出：

```text
Publishing: 'Hello World: 1'
Publishing: 'Hello World: 2'
Publishing: 'Hello World: 3'
```

保持这个终端运行。

---

## 19. 测试 listener

再打开一个 Ubuntu 22.04 终端。

执行：

```bash
ros2 run demo_nodes_py listener
```

正常应该看到：

```text
I heard: [Hello World: 1]
I heard: [Hello World: 2]
I heard: [Hello World: 3]
```

这说明 ROS2 的基本发布订阅通信已经正常。

---

## 20. 最后检查 WSL

回到 Windows PowerShell：

```powershell
wsl -l -v
```

确认：

```text
Ubuntu-22.04
```

存在，并且：

```text
VERSION
2
```

---

# 最终结构

```text
Windows 11
└── E:
    └── WSL
        └── Ubuntu22.04
            └── Ubuntu 22.04
                └── /opt/ros/humble
```

ROS2 在 Linux 中的位置是：

```text
/opt/ros/humble
```

而 Ubuntu 22.04 本身位于：

```text
E:\WSL\Ubuntu22.04
```

因此 ROS2 的主要磁盘占用也会落在 E 盘。

---

# 核心命令汇总

## PowerShell

```powershell
wsl -l -o
mkdir E:\WSL
mkdir E:\WSL\Ubuntu22.04
wsl --install -d Ubuntu-22.04 --location E:\WSL\Ubuntu22.04
wsl -l -v
wsl -d Ubuntu-22.04
```

## Ubuntu

```bash
sudo apt update
sudo apt upgrade -y
```

```bash
sudo apt install locales software-properties-common curl -y
```

```bash
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

```bash
sudo add-apt-repository universe
sudo apt update
```

```bash
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
-o /usr/share/keyrings/ros-archive-keyring.gpg
```

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

```bash
sudo apt update
sudo apt install ros-humble-desktop ros-dev-tools -y
```

```bash
source /opt/ros/humble/setup.bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

```bash
echo $ROS_DISTRO
echo $ROS_VERSION
```

```bash
ros2 run demo_nodes_cpp talker
```

```bash
ros2 run demo_nodes_py listener
```
