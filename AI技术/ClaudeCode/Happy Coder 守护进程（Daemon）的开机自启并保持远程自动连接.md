













### **要实现 Happy Coder 守护进程（Daemon）的开机自启并保持远程自动连接，最稳妥的做法是利用 Linux 系统的 Systemd 用户服务或使用 Node.js 的进程管理器 PM2 来托管 `happy daemon start` 命令。**

### **一、 认识 Happy Daemon 及其作用**

Happy Coder（目前主要以 `happy` 或 `happy-coder` 命令行工具存在）是一款专为 Claude Code 和 Codex 打造的移动端和 Web 远程客户端。 [GitHub - slopus/happy](https://github.com/slopus/happy) 它的远程连接和多会话管理高度依赖于后台运行的守护进程（Daemon）。该进程负责在本地后台监听，并通过端到端加密通道安全传输控制指令，从而允许你随时通过手机或浏览器恢复、监控或创建新的 AI 编程会话。 [Happy CLI Docs](https://slopus-happy-9.mintlify.app/cli/daemon) 

在配置开机自启之前，建议在终端运行以下命令，确定你的 `happy` 可执行程序的真实路径：

```bash
which happy
```

通常，全局安装的路径为 `/usr/local/bin/happy`、`/usr/bin/happy` 或当前用户 Node 环境下的 `~/.npm-global/bin/happy`。

---

### **二、 方案 1：使用 PM2 托管自启（最推荐，跨平台）**

作为 Node.js 生态中最成熟的进程管理器，PM2 可以非常轻松地实现 `happy daemon` 的开机自启、崩溃自动重启以及日志流监控。 [Happy Daemon Guide](https://slopus-happy-9.mintlify.app/guides/daemon-management)

#### **1. 安装 PM2**
如果你的系统尚未安装 PM2，请通过 npm 全局安装：
```bash
npm install -g pm2
```

#### **2. 使用 PM2 启动守护进程**
由于 `happy daemon start` 本身会以分离模式（detached）运行，我们直接让 PM2 运行并托管此启动指令：
```bash
pm2 start "happy daemon start" --name "happy-daemon"
```

#### **3. 锁定 PM2 进程状态并配置开机自启**
运行以下命令来保存当前的进程列表，并生成对应操作系统的开机自启配置：
```bash
pm2 save
pm2 startup
```
执行 `pm2 startup` 后，终端会输出一行类似于 `sudo env PATH=...` 的配置指令。请完整复制这行输出，并在终端中粘贴并执行。这样在系统重启时，PM2 就会自动拉起 Happy Daemon。

---

### **三、 方案 2：配置 Linux Systemd 用户服务（Linux 环境）**

由于 `happy` 需要读取当前用户的配置文件和加密密钥（存放在 `~/.happy/` 目录下），将其配置为 Systemd 的**用户级服务（User Service）**是最佳实践。这不仅能避免使用 `root` 权限导致的路径和环境变量丢失，还能确保安全隔离。 [Happy CLI Docs](https://slopus-happy-9.mintlify.app/cli/daemon)

#### **1. 创建 Service 描述文件**
在用户的配置目录下创建服务文件（如果目录不存在，请先创建）：
```bash
mkdir -p ~/.config/systemd/user/
nano ~/.config/systemd/user/happy-daemon.service
```

在文件中写入以下内容：
```ini
[Unit]
Description=Happy Coder Daemon Service
After=network.target

[Service]
Type=forking
# 请将 /usr/local/bin/happy 替换为 `which happy` 得到的实际绝对路径
ExecStart=/usr/local/bin/happy daemon start
ExecStop=/usr/local/bin/happy daemon stop
Restart=on-failure
RestartSec=5
# 自动导入基本环境变量
Environment=PATH=/usr/bin:/usr/local/bin:%h/.npm-global/bin
Environment=HOME=%h

[Install]
WantedBy=default.target
```

#### **2. 载入并启动服务**
运行以下命令让 Systemd 载入新配置，并启用开机自启：
```bash
# 重载用户级 Systemd 配置
systemctl --user daemon-reload

# 启用并立即启动服务
systemctl --user enable happy-daemon.service
systemctl --user start happy-daemon.service
```

#### **3. 开启用户残留（Linger）**
默认情况下，Systemd 用户服务在用户注销登录（SSH 断开）后会被终止。为了让守护进程在服务器刚开机且没有用户物理登录时就能够在后台运行，必须开启用户残留：
```bash
sudo loginctl enable-linger 用户名
```
*(请将 `用户名` 替换为你当前的系统登录用户名)*

---

### **四、 方案 3：配置 macOS Launchd（Mac 环境）**

如果你在使用一台不关机的 Mac（例如 Mac Mini）作为你的 AI 运行主机，可以通过 macOS 的 `launchd` 服务来实现开机自动运行。 [Happy Daemon Guide](https://slopus-happy-9.mintlify.app/guides/daemon-management)

#### **1. 创建 LaunchAgent 描述文件**
在用户的 LaunchAgents 目录下新建 `plist` 文件：
```bash
nano ~/Library/LaunchAgents/com.happy.daemon.plist
```

在文件中写入以下 XML 配置：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.happy.daemon</string>
    <key>ProgramArguments</key>
    <array>
        <!-- 请替换为 `which happy` 得到的真实绝对路径 -->
        <string>/usr/local/bin/happy</string>
        <string>daemon</string>
        <string>start</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <dict>
        <key>SuccessfulExit</key>
        <false/>
    </dict>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin</string>
    </dict>
</dict>
</plist>
```

#### **2. 载入服务**
使用 `launchctl` 将配置注册并立即启动：
```bash
launchctl bootstrap gui/用户UID ~/Library/LaunchAgents/com.happy.daemon.plist
```
*(注：用户UID 可通过在终端运行 `id -u` 获得，通常为 `501`)*

---

### **五、 验证与绑定远程客户端**

完成自启配置后，请按照以下步骤验证并配对你的设备，以确保可以实现全天候的自动远程连接：

#### **1. 检查守护进程状态**
```bash
happy daemon status
```
你应当能看到 Daemon 状态输出为运行中（Running），表明自启服务已顺利在后台接管了进程。 [Happy CLI Docs](https://slopus-happy-9.mintlify.app/cli/daemon)

#### **2. 进行远程客户端安全配对**
在终端运行配对命令：
```bash
happy --auth
```
此时终端会渲染出一个配对二维码以及一串认证 Key。 [Happy Engineering](https://happy.engineering/docs/quick-start) 你只需在手机上打开已安装的 Happy 客户端（可通过 Google Play 或 App Store 下载），或者访问 Web 端，扫描此二维码进行端到端加密的安全绑定。 [GitHub - slopus/happy](https://github.com/slopus/happy) 绑定完成后，即使你关闭终端或重启电脑，移动端也能通过云端中继自动寻找并无缝接入你本地不间断运行的 Claude Code。

---

### **六、 参考与学习资源**

如果你想更直观地了解其远程连接的工作原理和实际操作效果，可以参考以下资源：
- **演示与教程视频**: 推荐观看由社区制作的演示视频：[Happy Coder - Claude Code Mobile Client Tutorial](https://www.youtube.com/watch?v=UkDOO6N9Ju0)，它生动展示了如何在手机端操作并接管本地的 AI 进程。
- **项目官方仓库**: 如果在不同系统平台遇到特定路径或节点兼容问题，可以前往 [GitHub - slopus/happy](https://github.com/slopus/happy) 查看最新的 Issue 讨论。

你可以根据你当前服务器的实际操作系统选择最适合的方案。如果在配置路径或加载环境变量时遇到阻碍，请随时告诉我，我们可以针对特定的系统报错信息进行进一步的调试排查。