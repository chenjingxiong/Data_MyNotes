---
profileName: 飞牛的wordpress
postId: "8"
postType: post
categories:
  - 1
---
最后发现不用改配置文件，就在esxi中把显卡转为直通，重启，虚拟机添加[pci设备](https://zhida.zhihu.com/search?content_id=441774378&content_type=Answer&match_order=1&q=pci%E8%AE%BE%E5%A4%87&zhida_source=entity)，开机，打驱动，重启后关机，高级里面添加一个参数，完事。

![](https://pic1.zhimg.com/v2-8272be10b45df84a8bf6fd45304dde0d_r.jpg?source=2c26e567)

出了这个还有什么无法注册设备的错误；

————————————————————————————————————————

成功的图

![](https://pic1.zhimg.com/v2-c62c666100dc827fde4dfedbe5bd3fb9_r.jpg?source=2c26e567)

![](https://pica.zhimg.com/v2-e28cc3a5943d175976e112bdfac8cbce_r.jpg?source=2c26e567)

![](https://picx.zhimg.com/v2-3fc42baa8b8c7a9b27f7ca537c37397c_r.jpg?source=2c26e567)