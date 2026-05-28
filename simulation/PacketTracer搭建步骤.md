# Packet Tracer 搭建步骤

## 1. 搭建目标

本仿真用于实现一个基础校园网络拓扑，主要完成以下功能：

1. 教学区、办公区、宿舍区、图书馆等不同区域终端接入。
2. 使用 VLAN 对不同区域进行逻辑隔离。
3. 通过三层核心交换机实现跨 VLAN 通信。
4. 通过服务器区提供 DNS、WWW、FTP、MAIL 等基础服务。
5. 通过出口路由器预留外部网络访问能力。

## 2. 设备准备

在 Cisco Packet Tracer 中放置以下设备：

| 设备类型 | 推荐设备 | 数量 | 用途 |
| --- | --- | ---: | --- |
| 路由器 | Router 2911 / 1941 | 1 | 作为校园网出口路由器 |
| 三层交换机 | Multilayer Switch 3560 | 1 | 作为核心交换机，负责 VLAN 网关和三层转发 |
| 二层交换机 | Switch 2960 | 2 | 作为接入交换机，连接不同区域终端 |
| 服务器 | Server | 4 | 分别模拟 DNS、WWW、FTP、MAIL 服务 |
| 终端 | PC | 4-8 | 模拟教学区、办公区、宿舍区、图书馆用户 |
| 外部服务器 | Server 或 Cloud | 1，可选 | 模拟外部网络或 Internet 服务 |

## 3. 拓扑连接

按照以下结构连接设备：

```text
Internet/外部服务器
        |
      Router
        |
   Core Switch
    /   |    \
ASW1  ASW2  Server区
 / \    / \
教学 办公 宿舍 图书馆
```

建议连接方式如下：

| 连接对象 | 接口示例 | 说明 |
| --- | --- | --- |
| Router ↔ Core Switch | Router G0/0 ↔ Core G0/1 | 出口三层链路 |
| Core Switch ↔ ASW1 | Core F0/1 ↔ ASW1 F0/24 | Trunk 链路 |
| Core Switch ↔ ASW2 | Core F0/2 ↔ ASW2 F0/24 | Trunk 链路 |
| Core Switch ↔ 服务器 | Core F0/3-F0/6 | 服务器区接入 VLAN 50 |
| ASW1 ↔ 教学区 PC | ASW1 F0/1 | Access，VLAN 10 |
| ASW1 ↔ 办公区 PC | ASW1 F0/2 | Access，VLAN 20 |
| ASW2 ↔ 宿舍区 PC | ASW2 F0/1 | Access，VLAN 30 |
| ASW2 ↔ 图书馆 PC | ASW2 F0/2 | Access，VLAN 40 |

## 4. VLAN 规划

| VLAN ID | VLAN 名称 | 区域 | 网段 | 网关 |
| --- | --- | --- | --- | --- |
| VLAN 10 | TEACHING | 教学区 | 192.168.10.0/24 | 192.168.10.1 |
| VLAN 20 | OFFICE | 办公区 | 192.168.20.0/24 | 192.168.20.1 |
| VLAN 30 | DORMITORY | 宿舍区 | 192.168.30.0/24 | 192.168.30.1 |
| VLAN 40 | LIBRARY | 图书馆 | 192.168.40.0/24 | 192.168.40.1 |
| VLAN 50 | SERVER | 服务器区 | 192.168.50.0/24 | 192.168.50.1 |
| VLAN 99 | MANAGEMENT | 管理区 | 192.168.99.0/24 | 192.168.99.1 |

## 5. 配置接入交换机

### 5.1 ASW1 配置

```cisco
enable
configure terminal

vlan 10
 name TEACHING
vlan 20
 name OFFICE

interface fastEthernet0/1
 switchport mode access
 switchport access vlan 10

interface fastEthernet0/2
 switchport mode access
 switchport access vlan 20

interface fastEthernet0/24
 switchport mode trunk

end
write
```

### 5.2 ASW2 配置

```cisco
enable
configure terminal

vlan 30
 name DORMITORY
vlan 40
 name LIBRARY

interface fastEthernet0/1
 switchport mode access
 switchport access vlan 30

interface fastEthernet0/2
 switchport mode access
 switchport access vlan 40

interface fastEthernet0/24
 switchport mode trunk

end
write
```

## 6. 配置核心交换机

核心交换机使用三层交换机，负责创建 VLAN、配置 VLAN 网关并进行跨 VLAN 路由。

```cisco
enable
configure terminal

ip routing

vlan 10
 name TEACHING
vlan 20
 name OFFICE
vlan 30
 name DORMITORY
vlan 40
 name LIBRARY
vlan 50
 name SERVER
vlan 99
 name MANAGEMENT

interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface vlan 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown

interface vlan 30
 ip address 192.168.30.1 255.255.255.0
 no shutdown

interface vlan 40
 ip address 192.168.40.1 255.255.255.0
 no shutdown

interface vlan 50
 ip address 192.168.50.1 255.255.255.0
 no shutdown

interface vlan 99
 ip address 192.168.99.1 255.255.255.0
 no shutdown
```

配置核心交换机连接接入交换机的 Trunk 端口：

```cisco
interface fastEthernet0/1
 switchport mode trunk

interface fastEthernet0/2
 switchport mode trunk
```

配置服务器区端口：

```cisco
interface range fastEthernet0/3 - 6
 switchport mode access
 switchport access vlan 50
```

配置核心交换机到出口路由器的三层链路：

```cisco
interface gigabitEthernet0/1
 no switchport
 ip address 192.168.100.2 255.255.255.252
 no shutdown

ip route 0.0.0.0 0.0.0.0 192.168.100.1

end
write
```

如果 Packet Tracer 中所选交换机端口不支持 `no switchport`，可以改用单独的出口 VLAN，例如 VLAN 100，作为核心交换机与路由器之间的三层连接。

## 7. 配置出口路由器

```cisco
enable
configure terminal

interface gigabitEthernet0/0
 ip address 192.168.100.1 255.255.255.252
 no shutdown

ip route 192.168.0.0 255.255.0.0 192.168.100.2

end
write
```

如需模拟外部网络，可以在路由器另一个接口连接外部服务器：

```cisco
configure terminal

interface gigabitEthernet0/1
 ip address 200.1.1.1 255.255.255.0
 no shutdown

end
write
```

外部服务器可配置为：

| 项目 | 配置 |
| --- | --- |
| IP 地址 | 200.1.1.10 |
| 子网掩码 | 255.255.255.0 |
| 默认网关 | 200.1.1.1 |

## 8. 配置终端和服务器地址

| 设备 | IP 地址 | 子网掩码 | 默认网关 | DNS |
| --- | --- | --- | --- | --- |
| 教学区 PC | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 | 192.168.50.10 |
| 办公区 PC | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 | 192.168.50.10 |
| 宿舍区 PC | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 | 192.168.50.10 |
| 图书馆 PC | 192.168.40.10 | 255.255.255.0 | 192.168.40.1 | 192.168.50.10 |
| DNS Server | 192.168.50.10 | 255.255.255.0 | 192.168.50.1 | 192.168.50.10 |
| WWW Server | 192.168.50.20 | 255.255.255.0 | 192.168.50.1 | 192.168.50.10 |
| FTP Server | 192.168.50.30 | 255.255.255.0 | 192.168.50.1 | 192.168.50.10 |
| MAIL Server | 192.168.50.40 | 255.255.255.0 | 192.168.50.1 | 192.168.50.10 |

## 9. 配置服务器服务

### 9.1 DNS 服务

进入 DNS Server：

```text
Services -> DNS
```

开启 DNS 服务，并添加记录：

| 域名 | IP 地址 |
| --- | --- |
| www.school.local | 192.168.50.20 |
| ftp.school.local | 192.168.50.30 |
| mail.school.local | 192.168.50.40 |

### 9.2 WWW 服务

进入 WWW Server：

```text
Services -> HTTP
```

开启 HTTP 服务，可以修改默认网页内容，例如显示“校园门户网站”。

### 9.3 FTP 服务

进入 FTP Server：

```text
Services -> FTP
```

开启 FTP 服务，并添加测试用户，例如：

| 用户名 | 密码 | 权限 |
| --- | --- | --- |
| student | 123456 | Read / Write |

### 9.4 MAIL 服务

进入 MAIL Server：

```text
Services -> EMAIL
```

设置邮件域名：

```text
school.local
```

添加测试账号，例如：

| 用户名 | 密码 |
| --- | --- |
| user1 | 123456 |
| user2 | 123456 |

## 10. 测试步骤

建议按照以下顺序测试，便于定位问题：

1. 在每台 PC 上 ping 自己所在 VLAN 的网关。
2. 教学区 PC ping 办公区、宿舍区、图书馆 PC，测试跨 VLAN 通信。
3. 各区域 PC ping 服务器区地址，例如 `192.168.50.20`。
4. PC 浏览器访问 `http://192.168.50.20`，测试 WWW 服务。
5. PC 浏览器访问 `http://www.school.local`，测试 DNS + WWW 服务。
6. PC 使用 FTP 客户端访问 `ftp.school.local`，测试 FTP 服务。
7. 使用邮件客户端配置 `mail.school.local`，测试邮件收发。
8. PC ping `192.168.100.1`，测试到出口路由器的连通性。
9. 如果配置外部服务器，PC ping `200.1.1.10`，测试外部网络访问。

## 11. 常见问题检查

| 问题 | 可能原因 | 检查方法 |
| --- | --- | --- |
| PC ping 不通网关 | VLAN 未创建、端口 VLAN 错误、网关未开启 | `show vlan brief`、`show ip interface brief` |
| 不同 VLAN 不能通信 | 核心交换机未开启三层路由 | 检查是否配置 `ip routing` |
| 接入交换机下 PC 不能跨 VLAN | 上联端口不是 Trunk | 检查 `show interfaces trunk` |
| 不能访问服务器 | 服务器 IP、网关或 VLAN 配置错误 | 检查服务器地址和核心交换机端口 VLAN |
| 域名不能访问 | DNS 未开启或记录错误 | 检查 DNS 服务和 PC 的 DNS 地址 |
| 不能访问外部网络 | 默认路由或回程路由缺失 | 检查核心交换机和路由器静态路由 |

## 12. 最小实现建议

如果时间有限，可以先完成最小可行版本：

1. Router 1 台。
2. Core Switch 1 台。
3. Access Switch 2 台。
4. VLAN 10、20、30、50。
5. PC 3 台。
6. DNS/WWW Server 1 台。

先保证不同 VLAN 能互通，PC 能通过域名访问校园门户网站，再逐步补充 FTP、MAIL、图书馆区、管理区和外部网络模拟。
