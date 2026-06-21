# Packet Tracer 搭建步骤

## 1. 搭建目标

本仿真用于实现一个中小型企业网络，企业包含行政楼、销售部和生产厂区三个建筑，中心机房位于行政楼。仿真重点验证以下功能：

1. 行政楼 6 个部门、销售部 5 个团队、生产厂区 3 个车间分别接入独立 VLAN，彼此不能二层通信。
2. 三层核心交换机提供各 VLAN 网关，实现必要的跨 VLAN 三层通信。
3. 内部服务器区部署 DNS、数据库服务器和重要业务服务器，外部网络不能直接访问。
4. DMZ 对外服务区部署 WWW 和 MAIL，内部用户与外部用户均可访问。
5. 出口路由器连接外部网络，并通过 ACL 模拟边界防火墙。

## 2. 设备准备

在 Cisco Packet Tracer 中放置以下设备：

| 设备类型 | 推荐设备 | 数量 | 命名建议 | 用途 |
| --- | --- | ---: | --- | --- |
| 路由器 | Router 2911 / 1941 | 1 | EDGE | 企业出口、NAT、边界 ACL |
| 三层交换机 | Multilayer Switch 3560 | 1 | CORE | VLAN 网关、三层路由、服务器汇聚 |
| 二层交换机 | Switch 2960 | 3 | ADMIN-ASW、SALES-ASW、PROD-ASW | 三个建筑/区域的接入交换机 |
| 服务器 | Server | 5 | DNS、DB、APP、WWW、MAIL | 内部服务和对外服务 |
| 终端 | PC | 14-15 | GM-PC、SALES1-PC、WS1-PC 等 | 每个 VLAN 至少一台测试终端 |
| 外部终端 | PC 或 Server | 1 | External-PC | 模拟外部网络用户 |

如果设备数量较多，可以先搭建最小验证版：行政楼选 2 个部门、销售部选 2 个团队、生产厂区选 1 个车间，再补齐所有 VLAN。

## 3. 拓扑连接

推荐拓扑如下：

```text
External-PC
    |
  EDGE
    |
  CORE
  /  |   |       |         \
 /   |   |       |          \
ADMIN SALES PROD 内部服务器区  DMZ服务区
 ASW   ASW  ASW  DNS/DB/APP   WWW/MAIL
```

建议连接方式如下：

| 连接对象 | 接口示例 | 链路/端口类型 |
| --- | --- | --- |
| EDGE G0/0 ↔ CORE G0/1 | 192.168.100.1/30 ↔ 192.168.100.2/30 | 三层链路 |
| EDGE G0/1 ↔ External-PC | 200.1.1.1/24 ↔ 200.1.1.10/24 | 外部网络 |
| CORE F0/1 ↔ ADMIN-ASW F0/24 | Trunk | 允许 VLAN 10-15、99 |
| CORE F0/2 ↔ SALES-ASW F0/24 | Trunk | 允许 VLAN 20-24、99 |
| CORE F0/3 ↔ PROD-ASW F0/24 | Trunk | 允许 VLAN 30-32、99 |
| CORE F0/4 ↔ DNS Server | Access | VLAN 50 |
| CORE F0/5 ↔ DB Server | Access | VLAN 50 |
| CORE F0/6 ↔ APP Server | Access | VLAN 50 |
| CORE F0/7 ↔ WWW Server | Access | VLAN 60 |
| CORE F0/8 ↔ MAIL Server | Access | VLAN 60 |
| CORE F0/9 ↔ MGMT-PC | Access | VLAN 99 |

接入交换机终端连接建议：

| 接入交换机 | 端口 | VLAN | 连接对象 |
| --- | --- | ---: | --- |
| ADMIN-ASW | F0/1-F0/6 | 10-15 | 总经理、人力、财务、技术、工会、保卫后勤 PC |
| SALES-ASW | F0/1-F0/5 | 20-24 | 销售一队至销售五队 PC |
| PROD-ASW | F0/1-F0/3 | 30-32 | 一车间至三车间 PC |

## 4. VLAN 与地址规划

| VLAN | 名称 | 区域 | 网段 | 网关 |
| --- | --- | --- | --- | --- |
| 10 | ADMIN_GM | 总经理办公室 | 192.168.10.0/24 | 192.168.10.1 |
| 11 | ADMIN_HR | 人力资源部 | 192.168.11.0/24 | 192.168.11.1 |
| 12 | ADMIN_FINANCE | 财务部 | 192.168.12.0/24 | 192.168.12.1 |
| 13 | ADMIN_TECH | 技术部 | 192.168.13.0/24 | 192.168.13.1 |
| 14 | ADMIN_UNION | 工会 | 192.168.14.0/24 | 192.168.14.1 |
| 15 | ADMIN_SEC_LOG | 保卫和后勤 | 192.168.15.0/24 | 192.168.15.1 |
| 20 | SALES_T1 | 销售一队 | 192.168.20.0/24 | 192.168.20.1 |
| 21 | SALES_T2 | 销售二队 | 192.168.21.0/24 | 192.168.21.1 |
| 22 | SALES_T3 | 销售三队 | 192.168.22.0/24 | 192.168.22.1 |
| 23 | SALES_T4 | 销售四队 | 192.168.23.0/24 | 192.168.23.1 |
| 24 | SALES_T5 | 销售五队 | 192.168.24.0/24 | 192.168.24.1 |
| 30 | PROD_WS1 | 一车间 | 192.168.30.0/24 | 192.168.30.1 |
| 31 | PROD_WS2 | 二车间 | 192.168.31.0/24 | 192.168.31.1 |
| 32 | PROD_WS3 | 三车间 | 192.168.32.0/24 | 192.168.32.1 |
| 50 | SERVER_IN | 内部服务器区 | 192.168.50.0/24 | 192.168.50.1 |
| 60 | DMZ_SERVICE | 对外服务区 | 192.168.60.0/24 | 192.168.60.1 |
| 99 | MANAGEMENT | 网络管理区 | 192.168.99.0/24 | 192.168.99.1 |

## 5. 搭建流程

### 5.1 放置并重命名设备

1. 放置 1 台 Router，命名为 `EDGE`。
2. 放置 1 台 Multilayer Switch，命名为 `CORE`。
3. 放置 3 台 2960 交换机，命名为 `ADMIN-ASW`、`SALES-ASW`、`PROD-ASW`。
4. 放置 5 台服务器，命名为 `DNS Server`、`DB Server`、`APP Server`、`WWW Server`、`MAIL Server`。
5. 每个部门、团队和车间至少放置 1 台 PC，用于代表该 VLAN 的用户。
6. 放置 1 台 `External-PC`，模拟外部网络访问者。

### 5.2 连接设备

按第 3 节拓扑表连接设备。交换机之间使用 Copper Straight-Through 通常也可正常工作，Packet Tracer 会自动适配；如果链路不亮，可尝试交叉线或等待端口转发状态恢复。

### 5.3 配置核心交换机

在 CORE 中执行 `configs/设备配置说明.md` 的核心交换机配置，主要包括：

1. 开启 `ip routing`。
2. 创建 VLAN 10-15、20-24、30-32、50、60、99。
3. 为每个 VLAN 配置 SVI 网关。
4. 将 F0/1、F0/2、F0/3 配置为 Trunk。
5. 将服务器端口划入 VLAN 50 或 VLAN 60。
6. 配置 G0/1 到 EDGE 的三层链路和默认路由。

核心交换机完成后，可使用以下命令检查：

```cisco
show vlan brief
show ip interface brief
show ip route
```

### 5.4 配置接入交换机

分别在 ADMIN-ASW、SALES-ASW、PROD-ASW 中执行对应配置。

ADMIN-ASW：

```cisco
enable
configure terminal
hostname ADMIN-ASW
vlan 10
 name ADMIN_GM
vlan 11
 name ADMIN_HR
vlan 12
 name ADMIN_FINANCE
vlan 13
 name ADMIN_TECH
vlan 14
 name ADMIN_UNION
vlan 15
 name ADMIN_SEC_LOG
vlan 99
 name MANAGEMENT

interface range fastEthernet0/1 - 6
 switchport mode access

interface fastEthernet0/1
 switchport access vlan 10
interface fastEthernet0/2
 switchport access vlan 11
interface fastEthernet0/3
 switchport access vlan 12
interface fastEthernet0/4
 switchport access vlan 13
interface fastEthernet0/5
 switchport access vlan 14
interface fastEthernet0/6
 switchport access vlan 15

interface fastEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10-15,99

end
write
```

SALES-ASW：

```cisco
enable
configure terminal
hostname SALES-ASW
vlan 20
 name SALES_T1
vlan 21
 name SALES_T2
vlan 22
 name SALES_T3
vlan 23
 name SALES_T4
vlan 24
 name SALES_T5
vlan 99
 name MANAGEMENT

interface range fastEthernet0/1 - 5
 switchport mode access

interface fastEthernet0/1
 switchport access vlan 20
interface fastEthernet0/2
 switchport access vlan 21
interface fastEthernet0/3
 switchport access vlan 22
interface fastEthernet0/4
 switchport access vlan 23
interface fastEthernet0/5
 switchport access vlan 24

interface fastEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 20-24,99

end
write
```

PROD-ASW：

```cisco
enable
configure terminal
hostname PROD-ASW
vlan 30
 name PROD_WS1
vlan 31
 name PROD_WS2
vlan 32
 name PROD_WS3
vlan 99
 name MANAGEMENT

interface fastEthernet0/1
 switchport mode access
 switchport access vlan 30

interface fastEthernet0/2
 switchport mode access
 switchport access vlan 31

interface fastEthernet0/3
 switchport mode access
 switchport access vlan 32

interface fastEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 30-32,99

end
write
```

### 5.5 配置出口路由器

EDGE 需要配置内侧接口、外侧接口、回程路由、NAT 和外部访问 ACL。完整配置见 `configs/设备配置说明.md`。

配置完成后，外部网络访问策略为：

| 外部访问目标 | 预期结果 |
| --- | --- |
| `192.168.60.20` 的 HTTP/HTTPS | 允许访问 WWW |
| `192.168.60.40` 的 SMTP/POP3/IMAP | 允许访问 MAIL |
| `192.168.50.30` DB Server | 禁止访问 |
| `192.168.50.40` APP Server | 禁止访问 |
| 其他企业内部网段 | 禁止直接访问 |

## 6. 配置终端和服务器地址

### 6.1 服务器地址

| 设备 | IP 地址 | 子网掩码 | 默认网关 | DNS |
| --- | --- | --- | --- | --- |
| DNS Server | 192.168.50.10 | 255.255.255.0 | 192.168.50.1 | 192.168.50.10 |
| DB Server | 192.168.50.30 | 255.255.255.0 | 192.168.50.1 | 192.168.50.10 |
| APP Server | 192.168.50.40 | 255.255.255.0 | 192.168.50.1 | 192.168.50.10 |
| WWW Server | 192.168.60.20 | 255.255.255.0 | 192.168.60.1 | 192.168.50.10 |
| MAIL Server | 192.168.60.40 | 255.255.255.0 | 192.168.60.1 | 192.168.50.10 |

### 6.2 用户终端地址

| 终端 | IP 地址 | 子网掩码 | 默认网关 | DNS |
| --- | --- | --- | --- | --- |
| GM-PC | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 | 192.168.50.10 |
| HR-PC | 192.168.11.10 | 255.255.255.0 | 192.168.11.1 | 192.168.50.10 |
| FIN-PC | 192.168.12.10 | 255.255.255.0 | 192.168.12.1 | 192.168.50.10 |
| TECH-PC | 192.168.13.10 | 255.255.255.0 | 192.168.13.1 | 192.168.50.10 |
| UNION-PC | 192.168.14.10 | 255.255.255.0 | 192.168.14.1 | 192.168.50.10 |
| SECLOG-PC | 192.168.15.10 | 255.255.255.0 | 192.168.15.1 | 192.168.50.10 |
| SALES1-PC | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 | 192.168.50.10 |
| SALES2-PC | 192.168.21.10 | 255.255.255.0 | 192.168.21.1 | 192.168.50.10 |
| SALES3-PC | 192.168.22.10 | 255.255.255.0 | 192.168.22.1 | 192.168.50.10 |
| SALES4-PC | 192.168.23.10 | 255.255.255.0 | 192.168.23.1 | 192.168.50.10 |
| SALES5-PC | 192.168.24.10 | 255.255.255.0 | 192.168.24.1 | 192.168.50.10 |
| WS1-PC | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 | 192.168.50.10 |
| WS2-PC | 192.168.31.10 | 255.255.255.0 | 192.168.31.1 | 192.168.50.10 |
| WS3-PC | 192.168.32.10 | 255.255.255.0 | 192.168.32.1 | 192.168.50.10 |
| External-PC | 200.1.1.10 | 255.255.255.0 | 200.1.1.1 | 可不填 |

## 7. 配置服务器服务

### 7.1 DNS 服务

进入 DNS Server：

```text
Services -> DNS
```

开启 DNS 服务，并添加记录：

| 域名 | IP 地址 | 用途 |
| --- | --- | --- |
| www.company.local | 192.168.60.20 | 企业 WWW 服务 |
| mail.company.local | 192.168.60.40 | 企业 MAIL 服务 |
| db.company.local | 192.168.50.30 | 内部数据库服务 |
| app.company.local | 192.168.50.40 | 内部业务服务 |

### 7.2 WWW 服务

进入 WWW Server：

```text
Services -> HTTP
```

开启 HTTP 服务，并把默认网页内容改为企业对外网站说明，例如“Company Web Service”。

### 7.3 MAIL 服务

进入 MAIL Server：

```text
Services -> EMAIL
```

设置邮件域名：

```text
company.local
```

添加测试账号：

| 用户名 | 密码 |
| --- | --- |
| user1 | 123456 |
| user2 | 123456 |
| sales1 | 123456 |

## 8. 测试步骤

建议按以下顺序测试，便于定位问题：

1. 每台 PC ping 本 VLAN 网关，例如 GM-PC ping `192.168.10.1`。
2. 不同 VLAN 之间互 ping，例如 GM-PC ping HR-PC，验证已通过三层通信互通，而不是二层同网段通信。
3. 用户 PC ping `192.168.50.10`，验证能访问 DNS Server。
4. 用户 PC 访问 `http://www.company.local`，验证 DNS 和 WWW 服务。
5. 用户 PC 配置 `mail.company.local`，验证内部邮件收发。
6. 用户 PC ping `200.1.1.10`，验证内部用户访问外部网络。
7. External-PC 使用浏览器访问 `http://192.168.60.20`，验证外部用户访问企业 WWW。
8. External-PC 尝试访问或 ping `192.168.50.30`、`192.168.50.40`，预期失败，验证内部数据库和重要业务服务器禁止外部访问。
9. 在 EDGE 上执行 `show access-lists`，查看 ACL 命中计数。
10. 在 EDGE 上执行 `show ip nat translations`，查看内部用户访问外部网络时是否产生 NAT 记录。

## 9. 常见问题检查

| 问题 | 可能原因 | 检查方法 |
| --- | --- | --- |
| PC ping 不通网关 | Access 端口 VLAN 错误或 SVI 未开启 | `show vlan brief`、`show ip interface brief` |
| 不同 VLAN 不能通信 | CORE 未开启 `ip routing` 或 Trunk 未放行 VLAN | `show ip route`、`show interfaces trunk` |
| 服务器不能访问 | CORE 服务器端口 VLAN 错误或服务器网关配置错误 | 检查服务器 IP、网关和端口 VLAN |
| 域名不能解析 | DNS 服务未开启或 PC DNS 地址错误 | 检查 DNS Server 和 PC IP Configuration |
| 外部不能访问 WWW | EDGE ACL、路由或 WWW 服务配置错误 | `show access-lists`、检查 HTTP 服务 |
| 外部能访问内部服务器 | EDGE 外侧 ACL 未应用或规则顺序错误 | `show running-config`、`show access-lists` |
| 内部不能访问外部网络 | 默认路由、回程路由或 NAT 配置错误 | `show ip route`、`show ip nat translations` |

## 10. 最小实现建议

如果时间有限，先完成以下最小版本：

1. CORE、EDGE、ADMIN-ASW、SALES-ASW、PROD-ASW 各 1 台。
2. 每个接入交换机先接 1-2 台 PC。
3. 先创建 VLAN 10、11、20、30、50、60。
4. 先部署 DNS、WWW、DB 三台服务器。
5. 先验证内部用户访问 WWW、外部用户访问 WWW、外部用户不能访问 DB。

最小版本跑通后，再补齐所有部门、团队、车间和 MAIL 服务。
