# WWW 服务配置说明

## 1. 服务作用

WWW 服务用于模拟校园门户网站，可以展示校园通知、教学资源入口、网络服务入口等内容。用户终端可以通过 IP 地址或域名访问该服务。

## 2. 服务器地址

| 项目 | 配置 |
| --- | --- |
| 服务器名称 | WWW Server |
| 所属 VLAN | VLAN 50 SERVER |
| IP 地址 | 192.168.50.20 |
| 子网掩码 | 255.255.255.0 |
| 默认网关 | 192.168.50.1 |
| DNS 地址 | 192.168.50.10 |
| 访问域名 | www.school.local |

## 3. Packet Tracer 配置步骤

1. 点击 WWW Server。
2. 进入 `Desktop -> IP Configuration`。
3. 配置 IP 地址、子网掩码、默认网关和 DNS 地址。
4. 进入 `Services -> HTTP`。
5. 将 HTTP 服务设置为 `On`。
6. 可根据需要修改默认 `index.html` 内容，例如显示“校园门户网站”。

## 4. DNS 配置要求

需要在 DNS Server 中添加以下记录：

| 域名 | IP 地址 |
| --- | --- |
| www.school.local | 192.168.50.20 |

## 5. 访问方式

用户终端可以通过以下方式访问：

```text
http://192.168.50.20
http://www.school.local
```

## 6. 测试方法

1. 在任意用户 PC 上执行 `ping 192.168.50.20`。
2. 打开 PC 的 Web Browser。
3. 输入 `http://192.168.50.20`，测试 IP 访问。
4. 输入 `http://www.school.local`，测试域名访问。

如果 IP 可以访问但域名不能访问，应检查 DNS 服务是否开启、DNS 记录是否正确、PC 的 DNS 地址是否为 `192.168.50.10`。