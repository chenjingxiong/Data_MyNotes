---
title:
source: "https://www.cnblogs.com/shanren/p/19685473"
author:
  - "[[雨梦山人]]"
published: 2026-03-08
created: 2026-03-15
description:
tags:
---
![](https://pica.zhimg.com/v2-61b7fb90be7e3ab9c75b0a6d71ebd190_1440w.jpg)

这篇文章会带你从零开始，把 OpenClaw 装进你的电脑。

读完你将拥有一个 24/7 在线的 AI 助手——能聊天，更能干活。

![](https://pic2.zhimg.com/v2-2df8b3e7102d28c0851ee8f7cc1af96f_1440w.jpg)

## 🎯 先搞清楚三件事

### 第一件：OpenClaw 是什么？

![](https://pic1.zhimg.com/v2-31b66d59fa9184372d869c8fe1984e8e_1440w.jpg)

当大家还在把 AI 当成 “聊天工具” 时，一种 **能真正帮你干活** 的智能体，已经来了。

普通的 AI 更像一个 **问答助手** 。你提问，它回答；你要文案，它给文字 —— 始终停留在 “对话” 层面。

而 OpenClaw 不一样。它是一位 **会动手、能落地、直接接管你电脑工作** 的专属助理。

让它整理文件，它就真的去整理；让它发邮件、查数据、管日程、控制智能家居，只要是你用电脑能完成的事，它都能替你做。

更重要的是，它 **开源免费、本地部署** 。所有操作都在你自己的电脑上完成，数据不经过第三方服务器，安全、可控、只属于你。

这，才是 AI 助理该有的样子。

### 第二件：你需要什么？

![](https://pic4.zhimg.com/v2-2f87981d837b83b328af071308d09b25_1440w.jpg)

| 准备项 | 说明 |  
| 一台电脑 | Mac、Windows 或 Linux 都行 |  
| 网络 | 安装时需要从 GitHub 下载东西 |  
| 30 分钟 | 第一次装会慢点，熟了就 5 分钟搞定 |  
| 一点点耐心 | 会用到"终端"，但都是复制粘贴，不需要会编程 |

**不需要** ：编程基础、懂代码、计算机专业背景。

### 第三件：不同系统走不同的路

因为系统差异，安装流程分成三条线：

- **Mac 用户** → 跳到「Mac 安装路线」
- **Windows 用户** → 跳到「Windows 安装路线」
- **Linux/Ubuntu 用户** → 跳到「Linux 安装路线」

配置部分（选 AI 模型、输入密钥等）所有系统一样，统一讲。

## 🍎 Mac 安装路线

![](https://pic4.zhimg.com/v2-fd45758d056bea0198b595fcb2f699ff_1440w.jpg)

### Step 1：打开终端

终端就是你跟电脑"用文字对话"的窗口。

**打开方法** ：

1. 按下 `Command + 空格` ，弹出搜索框
2. 输入 `Terminal` 或中文"终端"
3. 回车，搞定

终端长这样：一个窗口，里面有行文字和闪烁的光标，结尾是 `$` 或 `%` 符号。

**怎么粘贴命令？**

- 在网页上复制命令（ `Command + C` ）
- 切到终端窗口，按 `Command + V` 粘贴
- 回车执行

### Step 2：检查 Homebrew（可能已经有了）

Homebrew 是 Mac 上的"软件管家"，用来安装开发工具。

**好消息** ：OpenClaw 安装时会自动帮你装 Homebrew 和 Node.js。

所以你可以 **先跳到 Step 5** 试试直接装 OpenClaw。如果失败了，再回来从 Step 3 开始。

想先确认一下？输入：

- 显示 `Homebrew 5.x.x` → 有了，跳到 Step 4
- 显示 `command not found` → 没有，继续 Step 3

### Step 3：安装 Homebrew（如果没有）

**需要 3-10 分钟** ，中间会让你输入电脑密码。

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**安装过程中会遇到** ：

1. 让你按回车确认 → 直接按
2. 让你输入密码 → 输入开机密码（屏幕不会显示字符，这是正常的，盲打完回车就行）
3. 滚动一堆英文 → 正常，等它跑完
4. 显示 `Installation successful!` → 成功

**Apple Silicon 用户注意** （M1/M2/M3/M4 芯片）：

安装完可能会提示你运行两行命令配置路径。把终端里提示的那两行原样复制粘贴执行就行。

怎么判断是不是 Apple Silicon？点左上角苹果图标 → "关于本机"，看芯片那一栏。

**验证安装** ：

看到版本号就成功了。

如果Homebrew始终安装不上（网络源的问题），直接跳到Step6。

### Step 4：安装 Node.js

Node.js 是 OpenClaw 运行的环境。 **版本要求 ≥ 22** 。

先检查：

- 显示 `v22.x.x` 或更高 → 够了，跳到 Step 5
- 显示低于 v22 或 `command not found` → 需要装/升级

**安装命令** ：

如果装出来的版本还是低于 22，试试：

然后按终端提示链接到系统路径。

**验证** ：

看到 `v22` 或更高就对了。

### Step 5：安装 OpenClaw（一键安装）

主角登场。

```
curl -fsSL https://openclaw.ai/install.sh | bash
```

这行命令会：

1. 下载 OpenClaw 程序文件
2. 顺带安装 Homebrew 和 Node.js（如果没有）
3. 把它装到正确位置
4. 自动启动配置向导

**需要 5-10 分钟** ，中间不要关终端窗口。

### Step 6：NPM安装 OpenClaw（推荐）

**⚠️ 如果遇到权限问题** ：

```
# 确保 npm 全局目录有权限mkdir -p ~/.npm-globalnpm config set prefix '~/.npm-global'echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.zshrcsource ~/.zshrc
```

然后重新运行以下安装命令（推荐）。

在终端里输入：

这行命令会：

1. 下载 OpenClaw 程序文件
2. 把它装到正确位置

**需要 2-5 分钟** ，中间不要关窗口。

### Step 7：配置 OpenClaw

跳到「 **配置向导** 」部分继续。

## 🪟 Windows 安装路线

![](https://pic4.zhimg.com/v2-15386e7887eecb95260bdb19dbda13b5_1440w.jpg)

### Step 1：安装Node.js（已安装忽略）

访问： [https://nodejs.org/en/download](https://link.zhihu.com/?target=https%3A//nodejs.org/en/download) ，下载node.js安装文件。

![](https://pic4.zhimg.com/v2-7bed9abc1d465b328954f70d6b280331_1440w.jpg)

安装node.js，一直点Next下一步就好：

![](https://pic2.zhimg.com/v2-5ddaef713b4a2ddd4f5b2e046b77ee5f_1440w.jpg)

### Step 2：安装Git（已安装忽略）

访问： [https://git-scm.com/](https://link.zhihu.com/?target=https%3A//git-scm.com/) ，下载git安装文件。

![](https://pic2.zhimg.com/v2-650af7d56a64b9b7f4afb5f319b9f6b5_1440w.jpg)

![](https://picx.zhimg.com/v2-3a4488c20265f949377886d6f7dbd26f_1440w.jpg)

安装git：

![](https://pic2.zhimg.com/v2-f02ac96b5aaec0e3044b557f9da0cc01_1440w.jpg)

注意其中选择组件一步，勾选“Add a Git Bash Profile to Windows Terminal”，其余点击Next下一步

![](https://pica.zhimg.com/v2-ada994f22052f6dc93744710baf39e6e_1440w.jpg)

### Step 3：以管理员身份打开 CMD

OpenClaw 现在支持原生 Windows 安装， **不需要 WSL** 。

**打开方法** ：

1. 按键盘上的 `Win` 键
2. 搜索 CMD
3. **右键** 点击"CMD"
4. 选择" **以管理员身份运行** "

![](https://pic3.zhimg.com/v2-997485b83f0e26272a244b4e595dfc46_1440w.jpg)

1. 弹出提示"要允许此应用更改设备吗？" → 点"是"

### Step 4：安装 OpenClaw

在 CMD里输入：

```
#安装OpenClaw
npm i -g openclaw
```

这行命令会：

1. 下载 OpenClaw 程序文件
2. 把它装到正确位置

**需要 2-5 分钟** ，中间不要关窗口。

![](https://pic2.zhimg.com/v2-1db3ec64b5c8a11a6908913611da0f21_1440w.jpg)

**看到以上窗口** ，那么恭喜你安装成功了！

**⚠️ 如果安装失败** ：

- 确保网络能访问 GitHub（不过按照我的步骤99%成功！）

### Step 5：配置 OpenClaw

跳到「 **配置向导** 」部分继续。

## 🐧 Linux（Ubuntu）安装路线（适用云端服务器）

![](https://pic2.zhimg.com/v2-1e2cda7698984c0f103c819d7cf1de57_1440w.jpg)

### Step 1：打开终端

**打开方法** ：

- 按下 `Ctrl + Alt + T` （大多数 Linux 发行版）
- 或者在应用菜单里搜索"终端"或"Terminal"

### Step 2：更新系统软件包

在终端里输入：

```
sudo apt updatesudo apt upgrade -y
```

如果要输密码，输入你的 Linux 用户密码（输入时不显示字符，盲打后回车）。

### Step 3：安装 Node.js

**方法一：使用 NodeSource（推荐）**

```
# 安装 Node.js 22.xcurl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -sudo apt install -y nodejs
```

**方法二：使用 apt（可能版本较低）**

```
sudo apt install -y nodejs npm
```

**验证安装** ：

```
node --versionnpm --version
```

看到 `v22.x.x` 或更高就对了。

如果版本太低，使用方法一重新安装。

### Step 4：一键安装 OpenClaw

在终端里输入：

```
curl -fsSL https://openclaw.ai/install.sh | bash
```

这行命令会：

1. 下载 OpenClaw 程序文件
2. 把它装到正确位置
3. 自动启动配置向导

**需要 2-5 分钟** ，中间不要关窗口。

安装完成会自动进入配置向导 → 继续看下面的配置向导部分。

### Step 5：NPM安装 OpenClaw（推荐）

**⚠️ 如果遇到权限问题** ：

```
# 确保 npm 全局目录有权限mkdir -p ~/.npm-globalnpm config set prefix '~/.npm-global'echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrcsource ~/.bashrc
```

然后重新运行以下安装命令（推荐）。

在终端里输入：

这行命令会：

1. 下载 OpenClaw 程序文件
2. 把它装到正确位置

**需要 2-5 分钟** ，中间不要关窗口。

### Step 6：配置 OpenClaw

跳到「 **配置向导** 」部分继续。

## ⚙️ 配置向导（所有系统通用）

安装完成后，会自动进入一个交互式的配置向导。它会一步一步问你问题，你按提示选择就行。

**如果安装后没有自动进入配置向导** ，手动输入：

```
openclaw onboard --install-daemon
```

`--install-daemon` 会让 OpenClaw 注册为系统服务，开机自动启动。

### 配置 Step 1：安全提醒

第一个画面会提醒你 OpenClaw 是实验性软件，会访问你的文件和网络，问你是否了解风险。

![](https://pic2.zhimg.com/v2-5941c73b74fe37cf5cd010a4ba267495_1440w.jpg)

**操作** ：用键盘 **左箭头 `←`** 选择 `Yes` ，按回车继续。

**关于安全** ：OpenClaw 会读取你电脑上的文件来帮你干活，这也意味着它能看到你的私人文件。建议不要在存有敏感文件（银行信息、密码文档等）的电脑上使用，或者在使用时注意限制它的访问范围。

### 配置 Step 2：选择安装模式

会问你要 `QuickStart` （快速开始）还是自定义安装。

![](https://pica.zhimg.com/v2-d9ce05c6badc3a72e3a488860b6be43e_1440w.jpg)

**操作** ：选 `QuickStart` ，按回车。

选 QuickStart 就对了，它会帮你设置好最基本的东西。等你熟悉了以后，随时可以用 `openclaw onboard` 重新配置。

### 配置 Step 3：选择 AI 模型提供商（重要！）

![](https://pic3.zhimg.com/v2-594f8c7bef3126f4181fe4d324fa661c_1440w.jpg)

这一步是选"AI 的大脑"用谁家的。会列出几个选项，用键盘上下箭头选择，按回车确认。

| 提供商 | 推荐度 | 说明 | 价格参考 |  
| ChatGPT | ⭐⭐⭐ 首选 | 不封号，授权登录即可 | 免费额度大 |  
| Claude | ⭐⭐ 备选 | 功能最全，推理能力最强（不要授权登录会被封号） | 用API很贵 |  
| Gemini | ⭐⭐ 备选 | 性价比高，中文不错（授权也封号） | 推荐用API |  
| MiniMax | ⭐⭐ ⭐省钱之选 | 中文能力强，价格便宜 | 注册送免费额度 |  
| Kimi/Zhipu | ⭐⭐ ⭐国内首选 | 国内直连不需要梯子，便宜 | 注册送免费额度 |  
| Ollama（本地模型） | ⭐ 免费之选(不推荐) | 完全免费，不需要联网，但需要电脑配置好 | 免费 |

**省钱建议** ：

- 只是想试试看 → 选 **MiniMax** 或 **Kimi/Zhipu** ，注册就有免费额度
- 在国内，网络不方便 → 选他们就行，国内直连，不需要梯子
- 电脑配置好（至少 32GB 内存）→ 可以试试 **Ollama** ，完全免费，但需要另外安装 Ollama 软件
- 追求最好的效果 → 选 **Anthropic（Claude）** 但是要用API

### 配置 Step 4：连接 AI 模型（两种方式）

选完提供商后，你需要让 OpenClaw 连接上 AI 服务。有两种方式：

### 方式一：API Key（推荐，更稳定）

![](https://pic1.zhimg.com/v2-4023cfd4538cfa75509cd1b229ef5f9a_1440w.jpg)

API Key 是一串类似密码的文字，用来连接 AI 服务。

以 Zhipu（GLM-5）为例，获取 API Key 的步骤：

1. 打开浏览器，访问智谱AI开放平台 [https://open.bigmodel.cn/](https://link.zhihu.com/?target=https%3A//open.bigmodel.cn/)

![](https://pic3.zhimg.com/v2-a05982dc0c51c1cf9e2297a77d53e786_1440w.jpg)

1. 注册一个账号（需要手机号）
2. 登录后，点击右侧菜单的 **API Keys**
3. 点击 **添加新的API Key** （创建密钥）

![](https://pic2.zhimg.com/v2-0c5bbdda7d40a659a9f2ae2d37afe653_1440w.jpg)

1. 给密钥取个名字（随便写，比如 `openclaw` ），点创建
2. **复制生成的那串密钥**

**其他提供商获取 API Key 的地方** ：

| 提供商 | 注册地址 | 备注 |  
| OpenAI | [https://platform.openai.com/api-keys](https://link.zhihu.com/?target=https%3A//platform.openai.com/api-keys) | 需要绑定信用卡 |  
| Google Gemini | [https://aistudio.google.com/apikey](https://link.zhihu.com/?target=https%3A//aistudio.google.com/apikey) | 有免费额度 |  
| MiniMax | [https://platform.minimaxi.com/](https://link.zhihu.com/?target=https%3A//platform.minimaxi.com/) | Coding Plan |  
| DeepSeek | [https://platform.deepseek.com/api\_keys](https://link.zhihu.com/?target=https%3A//platform.deepseek.com/api_keys) | 国内直连 |  
| Kimi（月之暗面） | [https://platform.moonshot.cn/console/api-keys](https://link.zhihu.com/?target=https%3A//platform.moonshot.cn/console/api-keys) | 注册送 ¥15 额度 |  
| 阿里千问 | [https://www.aliyun.com/benefit/scene/codingplan?tlog=yuekan\_2](https://link.zhihu.com/?target=https%3A//www.aliyun.com/benefit/scene/codingplan%3Ftlog%3Dyuekan_2) | Coding Plan |

**提醒** ：第三方服务商可能更便宜，可以多问问。

### 方式二：OAuth 授权登录（方便，但有风险）

如果你有 ChatGPT Plus 或 Claude Pro 的付费订阅，OpenClaw 可以直接用你的账号登录，不需要单独买 API。

**⚠️ 重要警告：OAuth 有封号风险！**

2026 年 2 月，有多位用户报告用 Claude Pro/Max 和 Google 的 OAuth 账户接入 OpenClaw 后 **账号被封禁** 。多家媒体都报道了这个问题。

**建议** ：用独立的 API Key，不要用你的个人付费订阅账号的 OAuth。一定要用 OAuth 的话，用一个不重要的账号。

### 配置 Step 5：选默认模型

会让你选具体用哪个模型。

![](https://pic3.zhimg.com/v2-59a7ae456be6ababf23d1dc764fcdd90_1440w.jpg)

| 提供商 | 推荐模型 | 理由 |  
| Anthropic | Claude Sonnet 4.5 | 性价比最高 |  
| OpenAI | GPT-4o | 速度快、效果好 |  
| Google | Gemini 2.0 Flash | 快又便宜 |  
| MiniMax | M2.1 | 中文最好 |  
| DeepSeek | deepseek-chat | 国内直连、便宜 |  
| Kimi | kimi-k2.5 | 中文好、有免费额度 |  
| BigModel | glm-5 | 国内直连、便宜 |

**操作** ：用上下箭头选好模型，回车确认。

### 配置 Step 6：配置聊天渠道（可跳过）

这一步决定你以后怎么跟 OpenClaw 说话。可选：

- **Telegram** ：海外用户推荐，设置简单
- **WhatsApp** ：扫二维码就行
- **飞书** ：国内用户推荐，但配置稍复杂
- **网页面板** ：不用额外配置，装好就能用

![](https://pic4.zhimg.com/v2-55cc5a9a6997b3d8c0a59a576dabc2a1_1440w.jpg)

**操作** ：这一步建议 `Skip` （ **跳过** ）。网页面板默认就有，不用设。以后玩熟了，用 `openclaw onboard` 重新配置。

### 配置 Step 7：技能和其他配置

会问你要不要启用预装技能（网页浏览、文件管理、日程管理等）。

![](https://picx.zhimg.com/v2-11bcdaa2b5eb92e7d0181a17977db713_1440w.jpg)

**操作** ：先选 `Skip` （跳过）。后续可以通过聊天方式安装，更方便。

然后后面的所有选项都先选 `No` 。

![](https://pic1.zhimg.com/v2-354832e679ec46bf60b5ecd79968f076_1440w.jpg)

这一步是设置龙虾个性、记忆和配置的地方，后续通过聊天设置更方便。

### 配置 Step 8：安装完成！

看到类似 `OpenClaw is ready!` 或 `Your AI assistant is now running` 的提示，说明安装配置全部完成了。

它可能会问你要不要打开网页面板，选 `Open the web UI` 会在浏览器里打开。

![](https://picx.zhimg.com/v2-2adc2e2ef30bfe91caf0eaab0bf781c7_1440w.jpg)

**但我建议先选在终端里聊天** ，慢慢你会习惯这种操作方式，对以后有帮助。

![](https://pic4.zhimg.com/v2-651958b5500e3e3760d8188333050bd7_1440w.jpg)

最后你会看到这样的画面，就可以和它对话了！

## ✅ 验证安装成功

装完了，确认一下一切正常。

![](https://pic1.zhimg.com/v2-2d6bc3ca0a7181a7d195d615702595e6_1440w.jpg)

### 检查运行状态

- 显示 `Running` → 正常
- 显示没在运行 → 输入 `openclaw gateway start` 启动

### 打开网页面板

或者直接在浏览器地址栏输入：

`127.0.0.1` 是"你自己的电脑"的地址， `18789` 是 OpenClaw 用的端口。

如果看到 OpenClaw 的网页面板，恭喜你，安装成功！

### 发一条消息测试

在面板的聊天框里输入：

如果收到正常回复，说明 AI 大脑连接正常！

### 健康检查（可选）

它会自动检查各项配置，有问题会给出修复建议。

## 📋 常用命令速查

![](https://pic3.zhimg.com/v2-c7b341c43999df112d812e391b5eaee4_1440w.jpg)

装好之后，这些命令你会经常用到：

| 命令 | 作用 | 什么时候用？ |  
| openclaw gateway start | 启动 OpenClaw | 电脑重启后没自动启动时 |  
| openclaw gateway stop | 停止 OpenClaw | 不用了想关掉 |  
| openclaw gateway restart | 重启 OpenClaw | 改了配置之后 |  
| openclaw status | 查看运行状态 | 想确认它在不在运行 |  
| openclaw dashboard | 打开网页面板 | 想用网页跟它聊天 |  
| openclaw doctor | 自动检查问题 | 出问题了不知道哪错了 |  
| openclaw update | 更新到最新版 | 想体验新功能 |  
| openclaw onboard | 重新运行配置向导 | 想换 AI 模型或重新配置 |  
| openclaw logs | 查看运行日志 | 出问题想看详细错误 |

**小技巧** ：如果你在安装时加了 `--install-daemon` ，OpenClaw 会开机自动启动，不用每次手动输 `openclaw gateway start` 。

如果没有，下次打开终端输入：

## 🔧 常见问题排查

**原因** ：终端找不到 OpenClaw 程序，通常是环境变量没配好。

**解决** ：

1. 先试试 **关掉终端，重新打开** ，再输入 `openclaw status`
2. 如果还是不行，检查 npm 全局路径：
```
npm config get prefix把输出的路径加到 PATH 里：echo 'export PATH=$(npm config get prefix)/bin:$PATH' >> ~/.zshrcsource ~/.zshrc
```
1. 以上都不行 → 重新运行安装命令

### ❓ 网页面板打不开

**解决** ：

1. 确认 OpenClaw 在运行： `openclaw status`
2. 没在运行： `openclaw gateway start`
3. 再试访问 `http://127.0.0.1:18789/`
4. 如果端口被占用，换个端口：
```
openclaw gateway --port 18790
```
1. 然后访问 `http://127.0.0.1:18790/`

### ❓ AI 不回复消息

**排查** ：

1. 检查 API Key 是否正确： `openclaw doctor`
2. 检查 API 账户是否有余额：去对应提供商网站查看
3. 检查网络：用国外的 AI 服务（Claude、GPT）需要能访问国际网络
4. 检查模型名称：确保配置文件里的模型名跟提供商一致

### ❓ Mac 上 Homebrew 安装失败

**原因** ：网络问题，下载超时。

**解决** ：用国内镜像：

```
/bin/bash -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

### ❓ Node.js 安装失败或版本太低

**解决 1** ：从官网下载安装包

- 访问 [https://nodejs.org/zh-cn/download](https://link.zhihu.com/?target=https%3A//nodejs.org/zh-cn/download)
- Mac 用户下载 `.pkg` 安装包，双击安装
- 安装完关掉终端重新打开，输入 `node --version` 检查

**解决 2（Mac）** ：用 npm 的版本管理工具：

```
npm install -g nsudo n 22
```

### ❓ Windows 上安装失败

**解决** ：

- 确保网络能访问 GitHub
- 以管理员身份运行 CMD
- 检查 Windows 防火墙是否阻止了安装

### ❓ Linux 上权限问题

**解决** ：

```
mkdir -p ~/.npm-globalnpm config set prefix '~/.npm-global'echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrcsource ~/.bashrc
```

### ❓ 想重新配置

如果之前配置选错了，不用重新安装，直接运行：

会重新进入配置向导，重新选择 AI 模型、输入密钥等。

### ❓ 想完全卸载重装

```
openclaw gateway stopnpm uninstall -g openclawrm -rf ~/.openclaw
```

然后重新从安装命令开始。

## 🚀 装好之后可以做什么？

![](https://picx.zhimg.com/v2-476532f6107ca51f3b7a424e10c0d349_1440w.jpg)

OpenClaw 装好只是第一步。它真正强大的地方在于你可以给它装"技能"，让它会做更多事情。

更多技能可以在 ClawHub 上浏览和安装。目前 ClawHub 上已经有超过 16,878 个社区技能，但刚上手不要贪多， **先装几个最实用的就够了** 。

![](https://pic2.zhimg.com/v2-e6156a47b409228ad1104b39c2690edd_1440w.jpg)

### 新手必装（装完立刻有用）

安装方法很简单，在聊天框里对 OpenClaw 说「安装 xxx 技能」就行。

举例：安装ClawHub上最火爆技能：self-improving-agent

![](https://pic2.zhimg.com/v2-efc263ca99189d41cde1202210d588a3_1440w.jpg)

![](https://pic1.zhimg.com/v2-cc9f5e2a336d6e0c206b5c11f1725eea_1440w.jpg)

看到以上界面表示技能安装成功！

熟悉指令的同学也可以通过以下命令安装：

```
npx clawhub@latest install self-improving-agent
```

![](https://pic2.zhimg.com/v2-13c2f4940b1639f1d88b9114184b1295_1440w.jpg)

或者（已安装clawhub）：

```
clawdhub install self-improving-agent
```

也可以手动安装：

```
git clone https://github.com/peterskoett/self-improving-agent.git ~/.openclaw/skills/self-improving-agent
```

**第一天我们可以装几个评分最高的skill** ，用顺了再加。

![](https://pic4.zhimg.com/v2-87466c62f1350b58f39d313237b251d9_1440w.jpg)

| 技能 | 干什么用 | 一句话推荐理由 |  
| Tavily Search | 联网搜索最新资讯 | 让 AI 能查实时信息，不再只靠训练数据。零配置，装上就用 |  
| Summarize | 长文章/长邮件压缩成摘要 | 每天刷资讯省一半时间，知识工作者必备 |  
| Firecrawl | 抓取网页内容、提取正文 | 给 AI 一个链接，它就能读懂整个网页并总结 |  
| Find Skills | 帮助用户发现并安装技能 | 帮助你查找互联网上已有技能，避免重复造轮子 |

### 进阶推荐（玩熟了再装）

Openclaw和Claude code的skill是可以通用的，所以只要是skill都可以安装给Openclaw使用，推荐一些资源网址。

| 仓库 / 网站 | 网址 (URL) | 核心定位与主要内容 | 特色适用场景 / 备注 | 特色适用场景 / 备注 |  
| SkillsMP 技能市场 | [http://skillsmp.com](https://link.zhihu.com/?target=http%3A//skillsmp.com) | 最全面的 Claude 技能商店。包含 UI 设计、SEO 优化、品牌规范、Shopify 开发等各类现成技能组件。 | 直接使用：寻找高频业务场景（营销、前端、安全审计）的现成指令集，支持一键复制代码。 | 直接使用：寻找高频业务场景（营销、前端、安全审计）的现成指令集，支持一键复制代码或链接。 |  
| Agent Skills Directory | skills.sh | 模块化技能托管平台。由 Vercel 团队构建，托管了超过 2 万个遵循 SKILL.md 规范的技能模块。 | 标准分发：配合 CLI 工具 (npx skills add) 使用，支持将技能快速同步到 Cursor、Claude Code 或本地项目。 | 标准分发：配合 CLI 工具 (npx skills add) 使用，支持将技能快速同步到 Cursor、Claude Code 或本地项目。 |  
| Anthropic 官方示例库 | [http://github.com/anthropics/skills](https://link.zhihu.com/?target=http%3A//github.com/anthropics/skills) | 提供创意设计、MCP 服务器、Web 测试及各类文档处理（Office/PDF）的官方技能快照与模板。 | 权威参考：学习官方 skill-creator 开发规范，是自定义复杂技能的最佳起点。 | 权威参考：学习官方 skill-creator 开发规范，是自定义复杂技能的最佳起点。 |  
| Composio 精选集 | [http://github.com/ComposioHQ/awesome-claude-skills](https://link.zhihu.com/?target=http%3A//github.com/ComposioHQ/awesome-claude-skills) | 聚合优质第三方资源，侧重于 Claude 与 1000+ 外部应用（Slack, GitHub 等）的授权集成方案。 | 连接外部：当需要 Claude 操作特定 SaaS 工具或进行跨平台自动化流转时首选。 | 连接外部：当需要 Claude 操作特定 SaaS 工具或进行跨平台自动化流转时首选。 |  
| OpenClaw 精选集 | [http://github.com/VoltAgent/awesome-openclaw-skills](https://link.zhihu.com/?target=http%3A//github.com/VoltAgent/awesome-openclaw-skills) | 围绕 OpenClaw 开源框架整理的技能资源，聚焦开源生态下的技能收集与分类。 | 开源适配：适用于在非官方客户端或开源 Claude 环境下进行技能开发。 | 开源适配：适用于在非官方客户端或开源 Claude 环境下进行技能开发。 |  
| Claude Code 精选资源 | [http://github.com/hesreallyhim/awesome-claude-code](https://link.zhihu.com/?target=http%3A//github.com/hesreallyhim/awesome-claude-code) | 专注于 Anthropic 命令行代码助手 (Claude Code) 的使用技巧、插件和最佳工程实践。 | 编程增强：提升本地终端环境下的代码编写、调试及全栈开发效率。 | 编程增强：提升本地终端环境下的代码编写、调试及全栈开发效率。 |  
| 基于文件的规划工具 | [http://github.com/OthmanAdi/planning-with-files](https://link.zhihu.com/?target=http%3A//github.com/OthmanAdi/planning-with-files) | 围绕文件操作构建的能力集，通过文件交互实现复杂的任务规划与流程管理。 | 流程管理：适用于以文件/文档为驱动的自动化任务规划开发场景。 | 流程管理：适用于以文件/文档为驱动的自动化任务规划开发场景。 |  
| Google Workspace CLI | [http://github.com/googleworkspace/cli](https://link.zhihu.com/?target=http%3A//github.com/googleworkspace/cli) | 提供 Google 文档、表格、幻灯片等办公套件的 Node.js 命令行操作能力。 | 办公自动化：可集成至 Claude 技能中，实现 AI 对 Google 办公资源的程序化管理。 | 办公自动化：可集成至 Claude 技能中，实现 AI 对 Google 办公资源的程序化管理。 |  
| Claude 科学计算技能 | [http://github.com/K-Dense-AI/claude-scientific-skills](https://link.zhihu.com/?target=http%3A//github.com/K-Dense-AI/claude-scientific-skills) | 覆盖科研数据分析、科学计算流程、学术文档撰写等方向的垂直行业技能集。 | 科研学术：适用于科研人员利用 Claude 完成专业数据处理、公式推导及学术写作。 | 科研学术：适用于科研人员利用 Claude 完成专业数据处理、公式推导及学术写作。 |

## 🎭 进阶：给你的 AI 助手塑造灵魂

OpenClaw 的工作目录（默认在 ~/.openclaw/workspace/）里有一组 `.md` 文件，它们就像 AI 的"性格档案"。每次 AI 醒来，第一件事就是读这些文件，搞清楚自己是谁、在帮谁、该怎么做。

你不需要一次全配好， **从 `SOUL.md` 和 `USER.md` 开始就够了** ，其他的用到再加。

md格式的文件，使用文本编辑器，比如记事本都可以打开直接修改编辑。

### SOUL.md：它是谁

![](https://picx.zhimg.com/v2-c59693632d33d6265d66810985cbcb67_1440w.jpg)

这是最核心的文件，决定了你的 AI 助手的 **性格、说话方式和行为准则** 。

```
#mac电脑直接使用nano打开nano ~/.openclaw/workspace/SOUL.md
```

往里面写什么？举个例子：

```
# SOUL.md - 你是谁你是小白，一个温柔、直接、有主见的 AI 助手。核心信条：- 要提供真正的帮助，不是表演式的客套- 跳过"这是一个好问题！"这类废话，直接解决问题- 可以有不同意见，有偏好，甚至觉得某些事情无聊- 提问前先自己想办法，真的卡住了再问- 做一个你自己都想与之交谈的助手边界：- 隐私就是隐私，没得商量- 对外操作（发邮件、发推文）先问再做- 对内操作（读文件、整理笔记）大胆做
```

写完按 `Ctrl + X` ，然后按 `Y` 保存，按回车退出。

**重点** ：这个文件可以随时改。用了几天觉得它太啰嗦？改。觉得它不够主动？改。它会按照你最新写的来。

### IDENTITY.md：它叫什么

比 SOUL.md 更简单，就是给 AI 一张"名片"：

```
# IDENTITY.md- 名字：小白- 身份：你的 AI 私人助手- 风格：温柔、俏皮、支持感拉满- Emoji：💫名字随便起，叫"贾维斯"、"星期五"、"小爱"都行。
```

### USER.md：你是谁

![](https://pic3.zhimg.com/v2-ed617f89fc00c8e3baedb53c278f0ec6_1440w.jpg)

这个文件是告诉 AI"你在帮谁"，让它了解你的习惯和偏好：

```
# USER.md- 名字：老王Bingo- 时区：Asia/Shanghai- 偏好：沟通直接、步骤清晰、可以直接帮我执行- 当前重点：做自媒体内容、学习编程- 互动风格：可以轻松，但执行要稳- 高绩效认知：目标导向、结果优先、主动闭环、要事第一、持续迭代、高效复盘、极致落地
```

写了这个之后，AI 不会每次都问"你好，请问有什么可以帮你的？"，而是直接进入状态，知道你是谁、你在做什么。

### AGENTS.md：工作守则

这个文件定义了 AI 的 **工作流程和规矩** ，比如：

- 每次醒来先读哪些文件
- 什么操作可以自己做，什么必须先问你
- 群聊里该怎么表现（不要什么都回，像个正常人一样）
- 怎么管理记忆文件

你可以简单写，也可以写得很详细。OpenClaw 安装时会自动生成一个默认版本，后面慢慢按自己的需求改就好。

### HEARTBEAT.md：定时巡逻

这个文件让 AI **主动帮你干活** ，而不是等你发消息才动。

```
# HEARTBEAT.md1. 检查有没有新邮件，重要的通知我2. 看看日历，提前 2 小时提醒我有会议3. 检查系统运行状态
如果什么都没有，回复 HEARTBEAT_OK（别打扰我）。
```

配合 OpenClaw 的心跳机制（默认每 30 分钟执行一次），它就会像一个尽职的助理一样定时巡查，有事才叫你。

### MEMORY.md：长期记忆

AI 每次启动都是"失忆"的，但 MEMORY.md 让它能记住重要的事：

- 你做过的重要决策
- 你的偏好和习惯
- 之前踩过的坑

它会自动维护这个文件——把每天的对话日志（ `memory/YYYY-MM-DD.md` ）里值得记住的内容提炼进来，就像一个人整理日记、更新自己的认知一样。

| 文件 | 一句话说明 | 必须配吗 |  
| SOUL.md | AI 的性格和行为准则 | ⭐⭐⭐ 强烈建议 |  
| IDENTITY.md | AI 的名字和身份 | ⭐⭐ 建议 |  
| USER.md | 你的个人信息和偏好 | ⭐⭐⭐ 强烈建议 |  
| AGENTS.md | 工作流程和安全规矩 | ⭐ 可选（有默认值） |  
| HEARTBEAT.md | 定时自动执行的任务 | ⭐ 可选 |  
| MEMORY.md | AI 的长期记忆 | 自动生成 |

**最低配置** ：只写 `SOUL.md` + `USER.md` ，花 5 分钟，你的 AI 助手就能从"通用聊天机器人"变成"了解你的私人助手"。

![](https://pic4.zhimg.com/v2-26e8c336e1f7079241364c56238960bd_1440w.jpg)

如果你有任何地方卡住了，评论区告诉我，我帮你看看。

posted @ [雨梦山人](https://www.cnblogs.com/shanren) 阅读(3933)  评论(1) [收藏](https://www.cnblogs.com/shanren/p/) [举报](https://www.cnblogs.com/shanren/p/)

<table><tbody><tr><td colspan="7"><table><tbody><tr><td><a href="https://www.cnblogs.com/shanren/p/">&lt;</a></td><td align="center">2026年3月</td><td align="right"><a href="https://www.cnblogs.com/shanren/p/">&gt;</a></td></tr></tbody></table></td></tr><tr><th align="center">日</th><th align="center">一</th><th align="center">二</th><th align="center">三</th><th align="center">四</th><th align="center">五</th><th align="center">六</th></tr><tr><td align="center">1</td><td align="center">2</td><td align="center">3</td><td align="center">4</td><td align="center">5</td><td align="center">6</td><td align="center">7</td></tr><tr><td align="center"><a href="https://www.cnblogs.com/shanren/p/archive/2026/03/08"><u>8</u></a></td><td align="center">9</td><td align="center">10</td><td align="center">11</td><td align="center">12</td><td align="center">13</td><td align="center">14</td></tr><tr><td align="center">15</td><td align="center">16</td><td align="center">17</td><td align="center">18</td><td align="center">19</td><td align="center">20</td><td align="center">21</td></tr><tr><td align="center">22</td><td align="center">23</td><td align="center">24</td><td align="center">25</td><td align="center">26</td><td align="center">27</td><td align="center">28</td></tr><tr><td align="center">29</td><td align="center">30</td><td align="center">31</td><td align="center">1</td><td align="center">2</td><td align="center">3</td><td align="center">4</td></tr><tr><td align="center">5</td><td align="center">6</td><td align="center">7</td><td align="center">8</td><td align="center">9</td><td align="center">10</td><td align="center">11</td></tr></tbody></table>