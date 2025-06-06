# k8s-start-up

Kubernetes 入门学习项目

## 项目介绍

本仓库用于记录 Kubernetes 学习过程中的示例代码和笔记。

## 环境配置

### WSL2 多实例 Ubuntu 22.04 虚拟机设置

本项目需要三台 Ubuntu 22.04 虚拟机来模拟 Kubernetes 集群环境。

#### 1. 下载 Ubuntu 22.04 WSL2 镜像

```powershell
# 下载 Ubuntu 22.04 LTS
wsl --install -d Ubuntu-22.04
```

#### 2. 创建三台命名的虚拟机实例

```powershell
# 方法一：使用 wsl --import 创建多个实例
# 首先导出已安装的 Ubuntu 22.04 为基础镜像
wsl --export Ubuntu-22.04 ubuntu-22.04-base.tar

# 创建三台虚拟机
wsl --import k8s-master C:\WSL\k8s-master ubuntu-22.04-base.tar
wsl --import k8s-worker1 C:\WSL\k8s-worker1 ubuntu-22.04-base.tar
wsl --import k8s-worker2 C:\WSL\k8s-worker2 ubuntu-22.04-base.tar
```

#### 3. 启动和配置虚拟机

```powershell
# 启动各个实例
wsl -d k8s-master -u seed --cd /home/seed
wsl -d k8s-worker1 -u seed --cd /home/seed
wsl -d k8s-worker2 -u seed --cd /home/seed

# 查看所有 WSL 实例
wsl --list --verbose
```
#### 4. 在每台虚拟机中设置主机名

```bash
# 在 k8s-master 中执行（现在以 seed 用户身份）
sudo hostnamectl set-hostname k8s-master

# 在 k8s-worker1 中执行
sudo hostnamectl set-hostname k8s-worker1

# 在 k8s-worker2 中执行
sudo hostnamectl set-hostname k8s-worker2
```

#### 6. 配置静态 IP（可选）

```bash
# 编辑网络配置文件
sudo nano /etc/netplan/01-netcfg.yaml

# 示例配置（根据实际网络调整）
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.10/24  # k8s-master
        # - 192.168.1.11/24  # k8s-worker1
        # - 192.168.1.12/24  # k8s-worker2
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]

# 应用配置
sudo netplan apply
```

#### 7. 虚拟机管理命令

```powershell
# 停止特定实例
wsl --terminate k8s-master

# 停止所有 WSL 实例
wsl --shutdown

# 删除实例（谨慎使用）
wsl --unregister k8s-master

# 设置默认实例
wsl --set-default k8s-master

# 查看实例状态
wsl --status
```

#### 集群规划

| 虚拟机名称 | 角色 | 推荐配置 |
|-----------|------|----------|
| k8s-master | Master 节点 | 2 CPU, 4GB RAM |
| k8s-worker1 | Worker 节点 | 2 CPU, 2GB RAM |
| k8s-worker2 | Worker 节点 | 2 CPU, 2GB RAM |

#### 注意事项

- 确保 WSL2 已启用并更新到最新版本
- 每台虚拟机建议至少分配 2GB 内存
- 可以通过 `.wslconfig` 文件调整 WSL2 的全局资源分配
- 创建实例前确保有足够的磁盘空间

#### powershell配置文件
```json
{
    "commandline": "C:\\WINDOWS\\system32\\wsl.exe -d k8s-worker2 -u seed --cd /home/seed",
    "guid": "{c62bd44e-24b5-5096-8afd-9fd58ff023c7}",
    "hidden": false,
    "icon": "C:\\WSL\\k8s.png", 
    "name": "k8s-worker2",
    "source": "Windows.Terminal.Wsl",
    "tabTitle": "k8s-worker2"
},
{
    "commandline": "C:\\WINDOWS\\system32\\wsl.exe -d k8s-master -u seed --cd /home/seed",
    "guid": "{997795ff-960d-5d19-adb6-0acb0c3c63bc}",
    "hidden": false,
    "icon": "C:\\WSL\\k8s.png",
    "name": "k8s-master", 
    "source": "Windows.Terminal.Wsl",
    "tabTitle": "k8s-master"
},
{
    "commandline": "C:\\WINDOWS\\system32\\wsl.exe -d k8s-worker1 -u seed --cd /home/seed",
    "guid": "{43add32a-8e93-54ca-ba51-dd88ed2f8745}",
    "hidden": false,
    "icon": "C:\\WSL\\k8s.png",
    "name": "k8s-worker1",
    "source": "Windows.Terminal.Wsl", 
    "tabTitle": "k8s-worker1"
}
```
k8s.png下载链接
```
data:image/png;base64,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
```