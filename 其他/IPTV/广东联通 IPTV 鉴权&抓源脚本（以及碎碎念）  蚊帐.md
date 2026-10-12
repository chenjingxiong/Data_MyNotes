---
title: 广东联通 IPTV 鉴权&抓源脚本（以及碎碎念） | 蚊帐
source: https://mozz.ie/posts/gdcu-iptv-playlist/
author:
  - "[[蚊子]]"
published: 2024-10-27
created: 2025-09-04
description: 效果和用途同上一篇一样，只不过换成了联通的，两家的架构大差不差。至于我为什么从电信换成了联通，请看正文
tags:
  - clippings
profileName: 飞牛的wordpress
postId: "11"
postType: post
categories:
  - 1
---
## 广东联通 IPTV 鉴权&抓源脚本（以及碎碎念）

2024-10-27

4分钟阅读时长

> 好久不见，大家最近的网上冲浪生活过得如何？本来有很多次都想写博客，但是每当我准备动笔的时候我都不禁有，我即将要写的这个话题对读者有用吗？他们会喜欢看我的碎碎念吗？这么流水账的东西，还是不要写了吧……等种种念头，最终又导致我偷懒。那么总算是有一些可能对大家有用的东西，借着这个由头，我写下了这篇文章。

首先讲一下目前家里网络的配置吧。在下述事件发生的时间点，我家里有5个运营商的入户线，并且办理了5个宽带帐户，分别是：

- 广东电信（169元普通融合套餐 1000/100 公网+IPTV、290元CN2游戏宽带 无公网）
- 广东联通（180元商务极光1000/300 公网、38元三折冰淇淋 500/50 无公网）
- 广东移动（79元芒果卡 500/50 无公网）
- 广东广电（有入户线未安装）
- 长城宽带（有入户线未安装）

广东电信最近在疯狂的以打击PCDN之名关停大流量宽带的公网IP，并且将这些用户都拉到深圳出口，NAT类型为Symmetric。坊间称为广东电信NAT4竞技场事件。意料之中的是，我也被牵连了——因为我确实有跑PCDN，具体的表现是：MTU 变成 1430，IPv4 分到 100.64 的 CGNAT IP，IPv6 不能入站+半小时变一次。具体的事件可以看 [这个](https://www.v2ex.com/t/1079783) 和 [这个](https://www.v2ex.com/t/1082313#reply110) 帖子，在文章里就不展开讲了。与选择硬刚电信到底的网友不同，我对我家这条 169/月 1000/100 带公网的电信的态度一直是能让我跑我就跑，给我掐了我也愿赌服输。所以我麻利地带上光猫去营业厅办理了销户，然后转头就到隔壁的联通营业厅加开了IPTV业务。为什么是联通？因为电信和移动单独加IPTV都要20元一个月，而联通只要10元一个月，我也不知道为什么他们那么贵。

师傅非常敬业地快速来安装了，盒子型号是数码视讯TR100。由于广东联通一直以来的摆烂态度，IPTV这种业务他们肯定是完全不上心的（不然也不会必须到营业厅办理），所以机顶盒中一个广告也没有，甚至连电信/移动标配的增值业务包也没有，非常干净。而相对的，就是节目源的缺乏——我扫了一眼频道列表，就发现了没有本地台、高清卫视台只有7个、各种频道的回看缺失（在后面把源提取出来之后，竟然实际上能看）等种种问题。当然这些问题也无伤大雅，反正真正看的就那么几个台。

抓源的过程非常简单，我也做了一个小脚本方便进行鉴权和频道整理，以下是流程：

1. 按照 [前文的方法](https://mozz.ie/posts/extracting-iptv-live-streams/) 抓取完整过程的包，然后这次搜索 分组字节流-字符串 输入 `grant_type=EncryToken` ![image-20241030215024704](https://mozz.ie/assets/gdcu-iptv-playlist/image-20241030215024704.png)
2. 在数据包详情里面滚动到最下面，有一个Full request URI，右键 复制-值，就可以获得完整的鉴权 URL ![image-20241030215209212](https://mozz.ie/assets/gdcu-iptv-playlist/image-20241030215209212.png)
3. 替换以下脚本中的鉴权URL，并把脚本保存为 iptv.py
	```python
	import requests
	import json
	import os
	import sys
	# 设置工作目录为脚本所在目录
	os.chdir(os.path.dirname(os.path.abspath(__file__)))
	s = requests.Session()
	s.headers['User-Agent'] = 'okhttp/3.10.0'
	# 获取所有频道信息
	all_channels_response = s.get("http://120.87.12.38:8083/epg/api/custom/getAllChannel.json")
	all_channels_data = all_channels_response.json()
	if all_channels_data['status'] != '200' or 'channels' not in all_channels_data:
	    print("错误：无法获取频道列表或频道列表格式错误。")
	    sys.exit(1)
	# 提取频道代码
	channel_codes = [channel['params']['hwcode'] for channel in all_channels_data['channels'] if 'params' in channel and 'hwcode' in channel['params']]
	channel_codes_str = ','.join(channel_codes)
	# 获取访问令牌
	token_response = s.get("这里替换鉴权URL，可能IP地址不同，请确保同时修改下面的IP")
	access_token = token_response.json().get('access_token')
	if not access_token:
	    print("错误：未能获取有效的 access token。")
	    sys.exit(1)
	# 使用访问令牌请求数据
	headers = {
	    'Authorization': access_token,
	    'User-Agent': 'okhttp/3.10.0',
	    'Connection': 'Keep-Alive',
	    'Content-Type': 'application/json;charset=utf-8'
	}
	data = json.dumps({"channelcodes": channel_codes_str})
	# 请修改此处IP，确保与鉴权URL的IP一致
	response = s.post("http://120.87.11.25:33200/EPG/interEpg/channellist/batch", headers=headers, data=data)
	if not response.json():
	    print("错误：返回的数据为空。")
	    sys.exit(1)
	# 保存响应数据到文件
	with open('iptv.json', 'w',encoding='UTF-8') as f:
	    json.dump(response.json(), f)
	# 复制 base.txt 到 iptv.txt
	with open('base.txt', 'r',encoding='UTF-8') as f_base, open('iptv.txt', 'w' ,encoding='UTF-8') as f_iptv:
	    f_iptv.write(f_base.read())
	# 读取 iptv.json 并处理数据
	with open('iptv.json', 'r',encoding='UTF-8') as f:
	    data = json.load(f)
	    if not data.get('channellist'):
	        print("错误：'channellist' 数据为空。")
	        sys.exit(1)
	    channels = {}
	    for item in data['channellist']:
	        hwcode = item['channelcode']
	        channel_info = next((ch for ch in all_channels_data['channels'] if ch['params'].get('hwcode') == hwcode), None)
	        if not channel_info:
	            continue 
	        channel_name = channel_info['title']
	        timeshift_url = item['timeshifturl']
	        group = "其它"
	        if "CCTV" in channel_name or "CGTN" in channel_name:
	            group = "央视"
	        elif any(x in channel_name for x in ["广东", "广州", "大湾区", "南方", "岭南"]):
	            group = "广东"
	        elif "卫视" in channel_name:
	            group = "卫视"
	        if group:
	            if group not in channels:
	                channels[group] = []
	            channels[group].append(f"{channel_name},{timeshift_url}")
	    # 写入处理后的数据到 iptv.txt
	    with open('iptv.txt', 'a',encoding='UTF-8') as f:
	        for group, ch_list in channels.items():
	            f.write(f"{group},#genre#\n")
	            for channel in ch_list:
	                f.write(f"{channel}\n")
	```
4. 在同目录新建一个 base.txt，放你需要的其他频道列表（比如我放了一些其他节目源），如果没有就新建一个 base.txt 然后里面什么都不写即可。
5. 运行 iptv.py，它会在同目录下生成符合DIYP影音格式的 iptv.txt。
6. 如果需要修改分组规则或者排除非高清频道，请自行问 AI 让他帮你改脚本里的分类代码。

那么通过以上的脚本我们就稳定实现了电视节目源的自动更新，并且可以将它提取出来在任何有广东联通出口的网络上看啦（联通仍未像电信那样禁止在公网看IPTV，好！）

上一页 [双方无公网状态下 RouterOS 基于 WireGuard 的组网方案](https://mozz.ie/posts/dual-cgnat-routeros-wireguard-tunnel/)

下一页 [广东电信 IPTV 自助鉴权 + 获取播放列表](https://mozz.ie/posts/gdct-iptv-auth-and-fetch-playlist/)