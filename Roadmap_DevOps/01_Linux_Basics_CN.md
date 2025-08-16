# DevOps 的 Linux 基础（第一部分）

当人们谈论 **DevOps** 时，话题往往会很快跳到一些炫酷的工具，比如 Docker、Kubernetes 或 Terraform。
但在这些工具之下，有一个每个 DevOps 工程师都必须掌握的基石：**Linux**。

大多数云端服务器运行在 Linux 上。容器是基于 Linux 的。CI/CD 流水线通常在 Linux 环境中运行。
无论你是在部署应用、排查生产问题，还是在自动化工作流，极有可能都会用到 Linux。

如果你一直在关注我之前的博客，你会知道到目前为止我主要分享的内容是 **后端编程、API 和系统设计**。这是我成长路程中的重要部分。

现在，是时候开启一个**新篇章：DevOps**。
这将是一个新的技术博客系列，我会从基础讲起，逐步深入到 DevOps 领域更高级的主题。

就像任何一段旅程一样，我们要从地基开始。
这篇文章将介绍 Linux 基础 —— 踏入 DevOps 世界的必备前提。

---

## Shell：Bash vs. PowerShell

如果你在 Windows 上工作过，你应该对 **命令提示符（Command Prompt）** 或 **PowerShell** 很熟悉。
在 Linux 中，对应的是 **Bash**（Bourne Again Shell）。它们都是**命令解释器**，让你能够与操作系统交互。

作为一名 DevOps 工程师，你经常会：

* SSH 进入远程 Linux 服务器
* 在容器中运行基于 Linux 的命令
* 使用运行 shell 脚本的 CI/CD 流水线

因此，掌握 Bash 是必备技能。

---

## 常用 Linux 命令

以下是一些在 DevOps 工作中几乎每天都会用到的 **高频命令**。把它们当作你的生存工具包：

### 路径导航

```bash
pwd        # 显示当前工作目录（我在哪？）
ls         # 列出当前目录下的文件
cd /path   # 切换目录
```

### 文件操作

```bash
cat file.txt        # 打印文件内容
less file.txt       # 分页查看文件内容
cp file.txt backup/ # 复制文件
mv file.txt data/   # 移动/重命名文件
rm file.txt         # 删除文件
```

### 进程管理

```bash
ps aux        # 显示正在运行的进程
top           # 实时查看 CPU/内存使用情况
kill <PID>    # 按进程 ID 杀死进程
```

### 权限管理

```bash
chmod +x script.sh      # 赋予脚本可执行权限
chown user:user file    # 修改文件所有权
```

### 网络基础

```bash
ping google.com          # 检查主机是否可达
curl http://localhost:80 # 发起一个网络请求
ss -tulpn                # 显示监听端口及其进程
```

---

## 软件包管理器

在 Linux 服务器上你最先需要做的事情之一就是 **安装软件**。这就是 **软件包管理器** 的作用所在。
不同于 Windows 上常见的去网站下载安装程序，Linux 使用命令行软件包管理器，从可信的仓库中获取并安装软件。

常见的有：

* **APT**（Debian/Ubuntu）

```bash
sudo apt update
sudo apt install nginx
```

* **YUM / DNF**（RHEL, CentOS, Fedora）

```bash
sudo yum install httpd
# 或者在较新的系统上
sudo dnf install httpd
```

* **Homebrew**（macOS，本地开发环境加分项）

```bash
brew install wget
```

💡 **为什么对 DevOps 很重要**：
在搭建 CI/CD 流水线时，你可能需要安装 `dotnet-sdk`、`python3` 或 `docker` 等依赖。掌握包管理器的使用能确保你能够可靠地脚本化安装过程。

---

## Linux 服务

在 DevOps 中，我们部署的很多应用都作为 **服务（service）** 运行 —— 即由操作系统管理的后台进程。可以把服务理解为**常驻守护进程**（如 Web 服务器、数据库、消息队列）。

大多数现代 Linux 系统使用 **systemd** 作为服务管理器。常用命令如下：

```bash
# 启动服务
sudo systemctl start nginx

# 停止服务
sudo systemctl stop nginx

# 重启服务
sudo systemctl restart nginx

# 查看状态
sudo systemctl status nginx
```

💡 **为什么对 DevOps 很重要**：
如果你的 Web API “挂了”，你首先可能会检查服务是否在运行。重启服务、查看日志、验证配置，都是 DevOps 工程师的日常工作。

```bash
curl http://localhost:5000/health
```

---

## VI 编辑器（快速生存指南）

在某些时候，你会进入一台 Linux 服务器，需要快速修改一个配置文件 —— 可能是 Docker 配置、Nginx 设置，或环境变量文件。
而此时唯一可用的编辑器可能就是 **VI（或 Vim）**。

这里有一个**最小生存指南**，帮你不慌：

* 打开文件：

```bash
vi file.txt
```

* 切换到 **插入模式**（开始输入）：

```
i
```

* 保存并退出：

```
:wq
```

* 不保存退出：

```
:q!
```

这就足以应对 95% 的真实场景。如果你更喜欢简单的，可以安装 **nano**，但每个 DevOps 工程师至少应该掌握 VI 的基本操作。

💡 **专业提示**：当你凌晨 2 点 SSH 到生产服务器时，掌握这些 VI 命令能帮你避免很多压力。

---

## 学习资源

学习 Linux 可能让人觉得无从下手，因为它的范围非常广。好消息是，作为 DevOps 工程师，你不需要掌握 *所有* 知识 —— 只要学会在服务器、容器和自动化中必备的核心部分即可。

以下是对我有帮助的一些资源：

* **Roadmap.sh – Linux 学习路线图**
  [https://roadmap.sh/linux](https://roadmap.sh/linux)
  一个可视化的路线图，展示了开发者和 DevOps 最相关的 Linux 知识点。

* **YouTube 速成课**

  * [Linux Operating System - Crash Course for Beginners](https://youtu.be/ROjZy1WbCIA?si=Xd1DlSPPGxWzMBY6) （很棒的完整入门教程）。

    * 安装 Linux
    * 桌面环境概览
    * 终端基础
    * 目录操作
    * 文件操作
    * 查看和编辑文件内容
    * 理解 Linux 文件结构
    * 系统信息命令
    * 网络基础
    * 包管理器
    * 文本编辑器

👉 我的建议是：**只专注于对 DevOps 有用的部分**。不要陷入内核编程或写驱动程序的泥沼。
把重点放在命令、服务、权限以及基础的网络知识上。

---

## 总结：Linux 是地基

在 DevOps 学习旅程的第一部分，我介绍了每个 DevOps 工程师都应该掌握的 **Linux 基础**：

* 使用 Bash 导航
* 运行常用 Linux 命令
* 通过包管理器安装软件
* 使用 `systemctl` 管理服务
* 熟悉 VI 编辑器的基本操作

这些不仅仅是理论练习 —— 它们是你在部署应用、管理服务器或调试流水线时的日常工具。

💡 核心要点：*你不需要精通整个 Linux，但你必须对基础操作驾轻就熟*。

### 下一步

在接下来的文章里，我会进入 DevOps 基础的下一层：

* 虚拟化（VirtualBox, Vagrant）
* 网络基础（DNS、IP、端口）
* YAML 配置（DevOps 中无处不在）

敬请期待！🚀

---