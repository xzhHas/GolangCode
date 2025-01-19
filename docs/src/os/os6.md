---
title: Ubuntu24.04设置静态IP地址
shortTitle: 6.Ubuntu24.04设置静态IP地址
date: 2024-11-24
category:
  - 操作系统
tag:
  - 操作系统
---

## 前言：vm17.5的动态IP问题

![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182155892.png)

第一个是设置的静态IP我们可以看到是forever，第二个则是动态IP则是一天的时间。

如果我们不设置静态IP的话，那么可能在本地测试项目的时候，第二天发现一些服务不能用了，就是IP的问题。

## 简洁版快速解决问题

---
1. 编辑配置文件：进入路径`/etc/netplan`编辑文件`sudo vim /etc/netplan/01-netcfg.yaml
`,没有的话就新建即可。

2. 禁用动态IP：确保你的Netplan配置禁用了DHCP：

   ```yaml
   network:
     version: 2
     renderer: networkd
     ethernets:
       ens33:  #修改为你的实际接口名字
         dhcp4: no
         addresses:
           - 192.168.3.200/24
         routes:
           - to: default
             via: 192.168.3.1
         nameservers:
           addresses:
             - 8.8.8.8
             - 8.8.4.4
   ```


3. 应用Netplan配置：

   ```bash
   sudo netplan apply
   ```

4. 重启网络服务（可选）：

   ```bash
   sudo systemctl restart systemd-networkd
   ```

5. 验证配置：


   ```bash
   ip addr 
   ```

可以看到ens33就有这个静态IP地址了。


---
##  详细版解决问题教程

对设置静态IP地址的详细解释、扩展，以及为什么会出现动态IP导致问题的说明：
### 动态IP问题的原因
#### 动态IP的工作原理
动态IP是通过 **DHCP（Dynamic Host Configuration Protocol）** 自动分配的。它的优点是简化了IP地址的管理，但也有以下缺点：
1. 地址租约时间：
   DHCP分配的IP地址有一个租约时间（lease time）。在租约到期后，DHCP服务器可能会分配一个不同的IP地址给同一设备。
   - 例如：VM虚拟机启动时，DHCP服务器可能每次分配不同的IP地址。
   - 结果：本地测试服务依赖的IP地址可能发生变化，导致连接失败。

2. IP地址冲突：
   在某些情况下，如果有多个设备尝试连接，DHCP可能会分配不同的地址范围，这可能导致地址冲突或无法绑定固定IP。

3. 服务依赖：
   某些服务（如数据库、Web服务器等）通常需要固定的IP地址。如果使用动态IP，服务端配置会失效，影响系统的正常运行。
### 为什么需要静态IP？
1. 服务可靠性：
   本地测试或服务器环境通常依赖固定IP。静态IP能确保服务端地址不变，便于其他客户端访问。
2. 安全性：
   使用静态IP可以更精确地设置防火墙规则或访问控制规则。
3. 网络管理：
   静态IP可以帮助管理员更好地控制网络设备的分布和状态。

### 设置静态IP地址的详细步骤
#### 1. 定位Netplan配置文件
在Ubuntu 20.04及以上版本（包括24.04），Netplan用于管理网络配置。Netplan的配置文件通常位于 `/etc/netplan/` 目录中，文件名一般以 `.yaml` 结尾，例如 `01-netcfg.yaml` 或 `50-cloud-init.yaml`。  
如果文件不存在，可以创建一个新的配置文件：  
```bash
sudo vim /etc/netplan/01-netcfg.yaml
```
#### 2. 配置静态IP地址
以下是一个静态IP的Netplan配置示例：
```yaml
network:
  version: 2
  renderer: networkd  # 可选，适用于server版本；desktop版本可用NetworkManager
  ethernets:
    ens33:  # 替换为你的网络接口名称，可通过 `ip addr` 查看
      dhcp4: no  # 禁用动态IP
      addresses:
        - 192.168.3.200/24  # 静态IP地址和子网掩码
      routes:
        - to: default
          via: 192.168.3.1  # 网关地址
      nameservers:
        addresses:
          - 8.8.8.8  # Google公共DNS
          - 8.8.4.4  # Google备用DNS
```

---

#### 3. 应用Netplan配置
使用以下命令让新配置生效：
```bash
sudo netplan apply
```

#### 4. 重启网络服务（可选）
如果应用后发现网络未生效，可尝试重启相关服务：
```bash
sudo systemctl restart systemd-networkd
```

#### 5. 验证配置
运行以下命令查看网络接口状态，确认静态IP是否生效：
```bash
ip addr
```
输出示例：
```
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.3.200/24 brd 192.168.3.255 scope global ens33
       valid_lft forever preferred_lft forever
```
- `inet 192.168.3.200/24`：表示静态IP已经成功绑定。
- `valid_lft forever`：表示租约时间为永久（静态IP）。

---

### Netplan配置参数详解
1. **`addresses`**：配置IP地址，格式为 `IP地址/子网掩码`。
   - 例如：`192.168.3.200/24`，其中 `/24` 表示子网掩码为255.255.255.0。
2. **`routes`**：配置默认路由，用于访问外部网络。
   - `to: default`：指定默认路由。
   - `via: 192.168.3.1`：网关地址。
3. **`nameservers`**：配置DNS服务器。
   - 例如：`8.8.8.8` 和 `8.8.4.4` 是Google的公共DNS服务器。
4. **`dhcp4`**：用于启用或禁用IPv4的动态IP分配。
   - `yes`：启用DHCP。
   - `no`：禁用DHCP，手动配置静态IP。


### 动态IP与静态IP的比较

| 特性         | 动态IP                 | 静态IP                     |
| ------------ | ---------------------- | -------------------------- |
| **配置**     | 自动分配               | 需要手动配置               |
| **稳定性**   | 租约到期可能改变       | 固定不变                   |
| **适用场景** | 普通用户、临时设备     | 服务器、测试环境、本地服务 |
| **优点**     | 简单易用，节省地址资源 | 稳定可靠，便于管理         |
| **缺点**     | 可能因变更导致服务中断 | 配置复杂，易出错           |

![在这里插入图片描述](https://cdn.golangcode.cn/images/202501182155050.jpeg)
