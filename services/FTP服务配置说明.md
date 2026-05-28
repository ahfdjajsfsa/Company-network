# FTP 服务配置说明

## 1. 服务作用

FTP 服务用于模拟校园文件传输平台，可用于教学资料、实验文件、课程文件等资源的上传和下载。

## 2. 服务器地址

| 项目 | 配置 |
| --- | --- |
| 服务器名称 | FTP Server |
| 所属 VLAN | VLAN 50 SERVER |
| IP 地址 | 192.168.50.30 |
| 子网掩码 | 255.255.255.0 |
| 默认网关 | 192.168.50.1 |
| DNS 地址 | 192.168.50.10 |
| 访问域名 | ftp.school.local |

## 3. Packet Tracer 配置步骤

1. 点击 FTP Server。
2. 进入 `Desktop -> IP Configuration`。
3. 配置 IP 地址、子网掩码、默认网关和 DNS 地址。
4. 进入 `Services -> FTP`。
5. 将 FTP 服务设置为 `On`。
6. 添加测试用户并设置权限。

## 4. 用户账号规划

| 用户名 | 密码 | 权限 | 说明 |
| --- | --- | --- | --- |
| student | 123456 | Read / Write | 学生测试账号 |
| teacher | 123456 | Read / Write | 教师测试账号 |

## 5. DNS 配置要求

需要在 DNS Server 中添加以下记录：

| 域名 | IP 地址 |
| --- | --- |
| ftp.school.local | 192.168.50.30 |

## 6. 测试方法

在 PC 的 Command Prompt 中测试：

```text
ftp ftp.school.local
```

或使用服务器 IP 测试：

```text
ftp 192.168.50.30
```

登录时输入测试用户名和密码。登录成功后，可以使用简单的 FTP 命令进行测试，例如：

```text
dir
get 文件名
put 文件名
```

如果无法登录 FTP，应检查 FTP 服务是否开启、账号是否创建、PC 是否能 ping 通 `192.168.50.30`。