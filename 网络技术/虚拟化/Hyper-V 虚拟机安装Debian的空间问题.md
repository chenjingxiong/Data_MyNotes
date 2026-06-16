## 在动态 VHDX 文件上优化 Linux 文件系统

有些 Linux 文件系统可能会消耗大量的实际磁盘空间，即使文件系统大部分是空的。 要减少动态 VHDX 文件的实际磁盘空间使用量，请考虑以下建议：

- 创建 VHDX 时，请在 PowerShell 中将 BlockSizeBytes 设置为 1 MB（从默认的 32 MB），例如：

Powershell

```
PS > New-VHD -Path C:\MyVHDs\test.vhdx -SizeBytes 127GB -Dynamic -BlockSizeBytes 1MB
```

- ext4 格式优先于 ext3，因为与动态 VHDX 文件一起使用时，ext4 比 ext3 更节省空间。
    
- 创建文件系统时，将组数指定为 4096，例如：
    

Bash

```
# mkfs.ext4 -G 4096 /dev/sdX1
```