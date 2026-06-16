



### **可以通过“组策略 + 用户权限分配 + 电源策略 + Windows Update 策略”组合实现：禁止自动锁屏、隐藏/限制注销入口、禁止普通用户重启或关机。管理员仍建议保留应急权限，避免服务器无法维护。**

下面按 Windows Server 常用做法分步骤说明，适用于 Windows Server 2016 / 2019 / 2022 / 2025 的大多数场景。建议优先用“本地组策略”或“域组策略”配置，而不是单纯改注册表。

### **一、禁止自动锁屏**

如果你的目标是“远程桌面或控制台长时间不操作也不要自动锁屏”，重点要配置两类策略：会话空闲锁定和电源显示关闭。

#### **步骤 1：关闭系统空闲自动锁定**

打开：

```text
gpedit.msc
```

进入：

```text
计算机配置
→ Windows 设置
→ 安全设置
→ 本地策略
→ 安全选项
```

找到：

```text
交互式登录: 计算机不活动限制
```

设置为：

```text
0 秒
```

含义是禁用因不活动而自动锁定。

对应注册表位置通常为：

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System
```

键值：

```text
InactivityTimeoutSecs = 0
```

也可以用命令设置：

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v InactivityTimeoutSecs /t REG_DWORD /d 0 /f
```

#### **步骤 2：禁止屏幕保护程序锁定**

进入：

```text
用户配置
→ 管理模板
→ 控制面板
→ 个性化
```

建议配置：

| 策略项 | 建议设置 |
|---|---|
| 启用屏幕保护程序 | 已禁用 |
| 屏幕保护程序超时 | 已禁用或未配置 |
| 密码保护屏幕保护程序 | 已禁用 |

如果是域环境，需要在对应用户 OU 上配置，因为这是“用户配置”。

#### **步骤 3：禁止显示器关闭、睡眠、休眠**

服务器建议关闭睡眠和休眠：

```cmd
powercfg /change monitor-timeout-ac 0
powercfg /change standby-timeout-ac 0
powercfg /change hibernate-timeout-ac 0
powercfg /hibernate off
```

如果是域策略，路径为：

```text
计算机配置
→ 管理模板
→ 系统
→ 电源管理
```

可以配置“关闭显示器”“睡眠”“休眠”等策略为从不。

### **二、禁止手动锁定**

如果你不希望用户按 `Win + L` 或从开始菜单锁定服务器，可以通过组策略隐藏“锁定计算机”。

进入：

```text
用户配置
→ 管理模板
→ 系统
→ Ctrl+Alt+Del 选项
```

启用：

```text
删除“锁定计算机”
```

也可以配置：

```text
用户配置
→ 管理模板
→ 开始菜单和任务栏
```

查找并启用与“删除锁定”相关的策略项。

需要注意：这类策略主要是“移除入口”或“限制普通用户操作”，并不等于从系统底层完全取消所有锁定能力。管理员、系统服务、远程会话策略仍可能触发锁定或断开。

### **三、禁止注销**

“禁止注销”通常分两种需求：

第一种是隐藏开始菜单、Ctrl+Alt+Del 中的注销入口；第二种是防止 RDP 空闲### **可以通过“组策略 + 用户权限分配 + RDS 会话策略 + Windows Update 重启策略”来实现大部分限制；但不建议也很难对管理员账户做到绝对禁止，尤其是注销和重启。**

你的目标可以拆成三类：禁止锁屏、隐藏/限制注销、禁止或延后重启。Windows Server 上最稳妥的做法是用 GPO 统一下发，而不是只改注册表。微软文档说明，组策略可集中管理 Windows Server/Client 的用户与计算机配置；Windows Update 的自动重启也可通过组策略控制；开始菜单策略里也包含隐藏锁定、注销、重启、电源按钮等入口。 [Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy/group-policy-overview) [Microsoft Learn](https://learn.microsoft.com/en-us/windows/deployment/update/waas-restart) [Microsoft Learn](https://learn.microsoft.com/en-us/windows/configuration/start/policy-settings)

### **一、禁止锁屏**

#### **1. 禁止用户手动锁定**

打开本地组策略或域 GPO：

```text
用户配置
  └─ 管理模板
      └─ 系统
          └─ Ctrl+Alt+Del 选项
              └─ 删除“锁定计算机”
```

设置为：

```text
已启用
```

作用是移除 Ctrl+Alt+Del 界面中的“锁定计算机”入口，也会影响部分锁定入口。微软社区文档中也明确提到该策略路径：`User Configuration > Administrative Templates > System > Ctrl+Alt+Del Options > Remove Lock Computer`。 [Microsoft Learn](https://learn.microsoft.com/en-us/answers/questions/5776856/how-to-disable-lock-icon)

也可以用注册表方式下发：

```cmd
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System" /v DisableLockWorkstation /t REG_DWORD /d 1 /f
```

如果是域环境，建议使用 GPO Preferences 或 ADMX 策略，不建议逐台手工改。

#### **2. 隐藏开始菜单里的“锁定”**

策略路径：

```text
用户配置
  └─ 管理模板
      └─ 开始菜单和任务栏
          └─ 隐藏“锁定”
```

设置为：

```text
已启用
```

微软的开始菜单策略文档中包含 `Hide Lock`、`Hide Sign out`、`Hide Restart`、`Hide Shut down`、`Hide Power button` 等策略项，可用于隐藏开始菜单中的对应入口。 [Microsoft Learn](https://learn.microsoft.com/en-us/windows/configuration/start/policy-settings)

#### **3. 防止空闲后自动锁屏**

如果是因为屏保或空闲策略导致自动锁屏，检查这些策略：

```text
用户配置
  └─ 管理模板
      └─ 控制面板
          └─ 个性化
              ├─ 启用屏幕保护程序
              ├─ 密码保护屏幕保护程序
              └─ 屏幕保护程序超时
```

建议配置为：

| 策略 | 建议值 |
|---|---|
| 启用屏幕保护程序 | 已禁用 |
| 密码保护屏幕保护程序 | 已禁用 |
| 屏幕保护程序超时 | 未配置或设置较大值 |

也可在本机上检查电源设置，避免关闭显示器后触发锁定：

```cmd
powercfg /change monitor-timeout-ac 0
powercfg /change standby-timeout-ac 0
```

注意：`powercfg` 只控制电源行为，不等同于完整的锁屏策略。

### **二、禁止或隐藏注销**

#### **1. 隐藏开始菜单中的“注销 / Sign out”**

策略路径：

```text
用户配置
  └─ 管理模板
      └─ 开始菜单和任务栏
          └─ 隐藏“注销”
```

设置为：

```text
已启用
```

对应策略在微软开始菜单策略文档中属于 `Hide Sign out`。 [Microsoft Learn](https://learn.microsoft.com/en-us/windows/configuration/start/policy-settings)

#### **2. 禁止通过 Ctrl+Alt+Del 显示部分入口**

可同时检查：

```text
用户配置
  └─ 管理模板
      └─ 系统
          └─ Ctrl+Alt+Del 选项
```

这里可移除：

```text
删除“锁定计算机”
删除“任务管理器”
更改密码等入口
```

但要注意，Windows 并没有一个简单、官方、全局的“绝对禁止用户注销”开关。注销是用户会话生命周期的一部分，系统、RDS、策略超时、管理员操作、应用程序调用 API 都可能触发注销。因此实际工程上通常是“隐藏入口 + 避免 RDS 自动注销 + 限制用户权限”。

#### **3. 如果是远程桌面 RDS 自动注销，需要改会话超时策略**

路径：

```text
计算机配置
  └─ 管理模板
      └─ Windows 组件
          └─ 远程桌面服务
              └─ 远程桌面会话主机
                  └─ 会话时间限制
```

重点检查这些项：

```text
设置已断开会话的时间限制
设置活动但空闲的远程桌面服务会话的时间限制
达到时间限制时结束会话
设置 RemoteApp 会话注销的时间限制
```

建议配置：

| 策略 | 建议值 |
|---|---|
| 设置已断开会话的时间限制 | 从不 |
| 设置活动但空闲的远程桌面服务会话的时间限制 | 从不 |
| 达到时间限制时结束会话 | 已禁用 |
| 设置 RemoteApp 会话注销的时间限制 | 按需设为较长或从不 |

微软 RDS 文档说明，RDS 会话可能因为 `MaxIdleTime`、`MaxConnectionTime`、断开会话限制等策略进入锁定、断开或结束状态；RemoteApp 也有专门的“Set time limit for logoff of RemoteApp sessions”策略。 [Microsoft Learn](https://learn.microsoft.com/en-us/troubleshoot/windows-server/remote/troubleshoot-unexpected-rds-session-locks-or-disconnections) [Microsoft Learn](https://learn.microsoft.com/en-us/troubleshoot/windows-server/remote/remoteapp-sessions-disconnected)

### **三、禁止重启 / 关机**

这里要区分三种情况：

| 场景 | 处理方式 |
|---|---|
| 用户从开始菜单点重启 | 隐藏电源、重启、关机按钮 |
| 普通用户执行 `shutdown /r` | 移除“关闭系统”权限 |
| Windows Update 自动重启 | 配置 Windows Update 自动重启策略 |

#### **1. 隐藏开始菜单中的重启、关机、电源按钮**

策略路径：

```text
用户配置
  └─ 管理模板
      └─ 开始菜单和任务栏
          ├─ 隐藏“电源”按钮
          ├─ 隐藏“重启”
          ├─ 隐藏“关机”
          ├─ 隐藏“睡眠”
          └─ 隐藏“休眠”
```

建议至少启用：

```text
隐藏“电源”按钮
隐藏“重启”
隐藏“关机”
```

这能防止普通用户通过 UI 操作重启服务器。微软开始菜单策略文档列出了 `Hide Power button`、`Hide Restart`、`Hide Shut down` 等策略项。 [Microsoft Learn](https://learn.microsoft.com/en-us/windows/configuration/start/policy-settings)

#### **2. 移除普通用户的“关闭系统”权限**

打开：

```text
secpol.msc
```

路径：

```text
本地策略
  └─ 用户权限分配
      ├─ 关闭系统
      └─ 从远程系统强制关机
```

建议配置：

```text
关闭系统：只保留 Administrators，移除 Users、Remote Desktop Users、Power Users 等非必要组
从远程系统强制关机：只保留 Administrators
```

域环境中用 GPO：

```text
计算机配置
  └─ Windows 设置
      └─ 安全设置
          └─ 本地策略
              └─ 用户权限分配
                  ├─ 关闭系统
                  └─ 从远程系统强制关机
```

微软关于用户权限的文档说明，用户权限可分配给用户账户或组，决定用户/组能在系统上执行哪些操作；Active Directory 安全组也可用于分配这些权限。 [Microsoft Learn](https://learn.microsoft.com/en-us/windows/client-management/mdm/policy-csp-userrights) [Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups)

#### **3. 禁止 Windows Update 自动重启**

策略路径：

```text
计算机配置
  └─ 管理模板
      └─ Windows 组件
          └─ Windows 更新
```

重点配置：

```text
没有自动重启已登录用户的计划自动更新安装
```

设置为：

```text
已启用
```

英文策略名通常是：

```text
No auto-restart with logged on users for scheduled automatic updates installations
```

如果系统版本支持，还可以配置：

```text
Turn off auto-restart for updates during active hours
```

也就是在活动时间内关闭自动重启。

微软文档说明，Windows Update 安装后是否重启，可以通过组策略、MDM 或注册表控制；其中 `No auto-restart with logged on users for scheduled automatic updates installations` 可防止在用户已登录时自动重启。 [Microsoft Learn](https://learn.microsoft.com/en-us/windows/deployment/update/waas-restart)

注册表方式：

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoRebootWithLoggedOnUsers /t REG_DWORD /d 1 /f
```

如果你的服务器由 WSUS、SCCM、Intune、第三方补丁平台管理，还需要同步检查这些平台是否设置了强制重启 deadline。否则本机策略可能被上级补丁策略覆盖。

### **四、推荐的完整 GPO 配置清单**

如果你的目标是“服务器长期保持登录会话、不被普通用户锁屏/注销/重启”，建议建一个专用 GPO，例如：

```text
GPO 名称：Server-NoLock-NoSignout-NoReboot
链接位置：服务器所在 OU
安全筛选：只应用到目标服务器或目标用户组
```

配置如下：

| 目标 | 策略路径 | 策略值 |
|---|---|---|
| 禁止锁定 | 用户配置 → 管理模板 → 系统 → Ctrl+Alt+Del 选项 → 删除“锁定计算机” | 已启用 |
| 隐藏锁定 | 用户配置 → 管理模板 → 开始菜单和任务栏 → 隐藏“锁定” | 已启用 |
| 隐藏注销 | 用户配置 → 管理模板 → 开始菜单和任务栏 → 隐藏“注销” | 已启用 |
| 隐藏重启 | 用户配置 → 管理模板 → 开始菜单和任务栏 → 隐藏“重启” | 已启用 |
| 隐藏关机 | 用户配置 → 管理模板 → 开始菜单和任务栏 → 隐藏“关机” | 已启用 |
| 隐藏电源按钮 | 用户配置 → 管理模板 → 开始菜单和任务栏 → 隐藏“电源”按钮 | 已启用 |
| 禁止普通用户关机 | 计算机配置 → Windows 设置 → 安全设置 → 本地策略 → 用户权限分配 → 关闭系统 | 仅保留管理员 |
| 禁止远程强制关机 | 计算机配置 → Windows 设置 → 安全设置 → 本地策略 → 用户权限分配 → 从远程系统强制关机 | 仅保留管理员 |
| 禁止更新后自动重启 | 计算机配置 → 管理模板 → Windows 组件 → Windows 更新 → 没有自动重启已登录用户的计划自动更新安装 | 已启用 |
| 禁止 RDS 空闲注销 | 计算机配置 → 管理模板 → Windows 组件 → 远程桌面服务 → 远程桌面会话主机 → 会话时间限制 | 设置为从不或禁用结束会话 |

### **五、应用和验证**

在目标服务器上执行：

```cmd
gpupdate /force
```

查看策略结果：

```cmd
gpresult /h C:\gpresult.html
```

检查本机生效策略：

```cmd
rsop.msc
```

检查用户权限分配：

```cmd
secedit /export /cfg C:\secpol.cfg
notepad C:\secpol.cfg
```

检查 Windows Update 自动重启策略注册表：

```cmd
reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU"
```

检查当前 RDS 会话状态：

```cmd
query user
```

### **六、重要注意事项**

不要对生产服务器的管理员账户做“完全禁止重启/注销”的封死配置。Windows Server 维护、补丁、驱动、角色服务变更都可能需要重启；如果把管理员也限制住，可能会造成维护窗口无法操作，甚至需要通过带外管理、云控制台或物理机房介入。

更合理的做法是：普通运维/业务用户禁止锁屏、注销、重启；管理员保留紧急重启权限；Windows Update 自动重启改为维护窗口；RDS 会话超时策略改为从不自动结束。这样既能满足业务连续性，也不会把服务器管理能力完全锁死。