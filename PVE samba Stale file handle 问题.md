
在 Proxmox VE (PVE) 中遇到 Samba/CIFS 挂载点出现 `Stale file handle`（无效的文件句柄）错误时，通常是因为==网络共享短时间断开、服务端重启或 inode 缓存失效导致的==
添加 `noserverino` 挂载参数 pvesm set your-storage-id --options noserverino

或者也可以直接修改 `/etc/pve/storage.cfg` 配置文件，在对应的存储条目下加入：
cifs: sata12-137XXXX6391
        path /mnt/pve/sata12-137XXXX6391
        server 192.168.9.6
        share sata12-137XXXX6391
        content iso,vztmpl,images,rootdir,backup,snippets,import
        prune-backups keep-all=1
        username 13761056391
        options noserverino