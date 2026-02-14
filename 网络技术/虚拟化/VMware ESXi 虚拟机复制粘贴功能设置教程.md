# VMware ESXi 虚拟机复制粘贴功能设置

> 在 ESXi Web 界面中直接启用虚拟机的复制粘贴功能

---

## 一、为什么默认禁用？

- **安全考虑**：防止敏感数据通过剪贴板泄露
- **合规要求**：某些安全标准要求禁用此功能
- **风险评估**：仅在受信任环境中启用

---

## 二、通过 Web 界面设置（最简单）

### 步骤 1：登录 ESXi Web 界面

打开浏览器访问：
```
https://<ESXi-IP>/ui
```

### 步骤 2：找到虚拟机并编辑设置

在左侧导航栏中找到目标虚拟机，**右键点击** → 选择「编辑设置」

### 步骤 3：配置参数

1. 点击「**虚拟机选项**」选项卡
2. 找到「**高级**」→「**配置参数**」→「**编辑配置**」
3. 点击「**添加参数**」，添加以下两行：

| 参数键                             | 参数值     |                   |     |
| ------------------------------- | ------- | ----------------- | --- |
| `isolation.tools.copy.disable`  | `FALSE` |                   |     |
| `isolation.tools.paste.disable` | `FALSE` |                   |     |
| isolation.tools.dnd.disable     | FALSE   | 禁用拖放(drag n drop) |     |
| isolation.tools.hgfs.disable    | FALSE   | 共享文件夹             |     |

### 步骤 4：保存并重启

1. 点击「**保存**」
2. **关闭虚拟机** → **重新开机**（必须完全重启）

---

## 三、更简单的方式：Guest Isolation

如果你的版本支持，这是最快的方法：

### 步骤：

1. 右键虚拟机 → 「编辑设置」
2. 点击「虚拟机选项」选项卡
3. 找到 **「Guest Isolation」**（访客隔离）
4. 勾选 **「启用复制和粘贴」**
5. 保存并重启虚拟机

> ⚠️ 此选项在某些版本中不可用，如果没有看到，请使用上面的方法二

---

## 四、VMware Tools 必须安装

VMware Tools 是复制粘贴工作的**必要条件**。

### 检查状态

在 Web 界面查看虚拟机摘要，确认 VMware Tools 状态为「正在运行」

### 安装方法

#### Windows
1. 右键虚拟机 → 「Guest OS」→「安装 VMware Tools」
2. 进入虚拟机，打开光驱运行安装程序
3. 安装完成后重启虚拟机

#### Linux (Ubuntu/Debian)
```bash
sudo apt install open-vm-tools-desktop -y
sudo reboot
```

#### Linux (CentOS/RHEL)
```bash
sudo dnf install open-vm-tools-desktop -y
sudo reboot
```

---

## 五、检查清单

配置完成后，确认以下事项：

- [ ] VMware Tools 已安装并运行
- [ ] 两个配置参数都已添加
- [ ] 虚拟机已完全重启（非暂停恢复）
- [ ] 测试复制粘贴功能（双向）

---

## 六、常见问题

| 问题        | 解决方案                              |
| --------- | --------------------------------- |
| 配置后仍无法使用  | 确认 VMware Tools 已安装               |
| 只能复制不能粘贴  | 两个参数都要设置为 FALSE                   |
| Web 界面不支持 | 尝试使用 VMware Remote Console (VMRC) |

---

#vmware #esxi #虚拟化
