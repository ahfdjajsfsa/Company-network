# MAIL 服务配置说明

## 1. 服务作用

MAIL 服务用于模拟校园内部邮件系统，实现不同用户之间的邮件收发测试。

## 2. 服务器地址

| 项目 | 配置 |
| --- | --- |
| 服务器名称 | MAIL Server |
| 所属 VLAN | VLAN 50 SERVER |
| IP 地址 | 192.168.50.40 |
| 子网掩码 | 255.255.255.0 |
| 默认网关 | 192.168.50.1 |
| DNS 地址 | 192.168.50.10 |
| 访问域名 | mail.school.local |
| 邮件域名 | school.local |

## 3. Packet Tracer 配置步骤

1. 点击 MAIL Server。
2. 进入 `Desktop -> IP Configuration`。
3. 配置 IP 地址、子网掩码、默认网关和 DNS 地址。
4. 进入 `Services -> EMAIL`。
5. 设置邮件域名为 `school.local`。
6. 将 SMTP 和 POP3 服务设置为 `On`。
7. 添加测试用户账号。

## 4. 邮件账号规划

| 用户名 | 密码 | 邮箱地址 | 说明 |
| --- | --- | --- | --- |
| user1 | 123456 | user1@school.local | 测试用户 1 |
| user2 | 123456 | user2@school.local | 测试用户 2 |
| teacher | 123456 | teacher@school.local | 教师测试用户 |
| student | 123456 | student@school.local | 学生测试用户 |

## 5. DNS 配置要求

需要在 DNS Server 中添加以下记录：

| 域名 | IP 地址 |
| --- | --- |
| mail.school.local | 192.168.50.40 |

## 6. PC 邮件客户端配置

在 PC 中进入：

```text
Desktop -> Email
```

示例配置：

| 项目 | user1 配置 |
| --- | --- |
| Your Name | user1 |
| Email Address | user1@school.local |
| Incoming Mail Server | mail.school.local |
| Outgoing Mail Server | mail.school.local |
| User Name | user1 |
| Password | 123456 |

另一台 PC 可配置为 user2，用于收发测试。

## 7. 测试方法

1. 在第一台 PC 上配置 user1 邮箱。
2. 在第二台 PC 上配置 user2 邮箱。
3. user1 给 user2 发送测试邮件。
4. user2 点击 Receive 接收邮件。
5. user2 回复 user1，验证双向通信。

如果邮件无法发送或接收，应检查 MAIL 服务是否开启、账号是否创建、DNS 是否能解析 `mail.school.local`，以及 PC 是否可以 ping 通 `192.168.50.40`。