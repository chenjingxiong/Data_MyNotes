

### **Windows RDP 登录 Microsoft 账号，用户名通常填 `MicrosoftAccount\你的邮箱`，密码填微软账号密码，不是 PIN。**

在远程桌面连接时，如果目标电脑登录的是个人 Microsoft 账号，例如 Outlook、Hotmail、Live 或其他绑定微软账号的邮箱，最常用的 RDP 用户名格式是：`MicrosoftAccount\your_email@example.com`。密码要使用 Microsoft 账号的真实密码，不能使用 Windows Hello PIN、指纹或人脸识别。微软文档中也提到，在部分企业/组织账号场景下可使用 `user@domain.com` 这种 UPN 格式登录。 [Microsoft Learn](https://learn.microsoft.com/zh-cn/windows-server/remote/remote-desktop-services/remotepc/remote-desktop-connection-single-sign-on)

### **具体登录方式**

在你本地电脑打开“远程桌面连接”，输入远程电脑的 IP 地址或电脑名后，凭据窗口这样填：

```text
用户名：MicrosoftAccount\你的微软账号邮箱
密码：你的 Microsoft 账号密码
```

例如：

```text
用户名：MicrosoftAccount\abc@outlook.com
密码：微软账号密码
```

注意这里的反斜杠是 `\`，不是 `/`。

### **如果提示“你的凭据不起作用”**

最常见原因是目标 Windows 电脑平时只用 PIN 登录，RDP 不接受 PIN。你需要先在目标电脑上用 Microsoft 账号的“密码”本地登录一次，而不是用 PIN 登录。

可以在目标电脑上这样处理：

1. 打开“设置”。
2. 进入“账户”。
3. 进入“登录选项”。
4. 找到类似“为了提高安全性，仅允许对此设备上的 Microsoft 账户使用 Windows Hello 登录”的选项。
5. 将它关闭。
6. 注销或重启后，选择“密码”方式登录一次。

之后再尝试 RDP 连接。

### **也可以确认真实用户名**

有时 Windows 会给 Microsoft 账号生成一个本地用户名，比如邮箱是 `abc@outlook.com`，但本地用户目录可能是 `C:\Users\abc` 或其他名字。你可以在目标电脑上打开命令提示符，输入：

```cmd
whoami
```

它会显示类似：

```text
电脑名\用户名
```

如果你想用本地用户名格式登录，也可以试：

```text
远程电脑名\本地用户名
```

例如：

```text
DESKTOP-123456\abc
```

但如果你明确要用 Microsoft 账号登录，优先试：

```text
MicrosoftAccount\abc@outlook.com
```

### **还要检查远程桌面权限**

目标电脑必须满足这些条件：

- Windows 版本支持远程桌面主机功能，通常需要 Windows Pro、Enterprise 或 Education；Windows Home 一般不能作为被远程连接的主机。
- 已开启“远程桌面”。
- 当前 Microsoft 账号有密码。
- 该账号属于“管理员”或已被加入“远程桌面用户”。
- 两台设备网络可达，防火墙没有阻止远程桌面。
- 如果跨公网连接，建议通过 VPN，不建议直接暴露 3389 端口。

最推荐你先试这个组合：

```text
用户名：MicrosoftAccount\你的邮箱
密码：微软账号密码
```

如果失败，再到目标电脑上关闭 Windows Hello-only 登录限制，并用密码本地登录一次后重试。