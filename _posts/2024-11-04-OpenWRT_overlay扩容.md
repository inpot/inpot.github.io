OpenWRT overlay扩容

这里是直接扩容原分区, 参考https://www.youtube.com/watch?v=i1_cmHxyQVo

1. 需要的插件lsblk fdisk losetup resize2fs f2fs-tools

2. 查看分区情况

   ```bash
   fdisk -l
   ```

3. 修改分区

   ```bash
   fdisk /dev/当前磁盘
   #再次确认查看情况
   p
   #查看第二个分区的Start,删除第二个分区
   d
   2
   #创建新分区
   n
   #设置为primary
   p
   #设置上边分区开始的偏移量
   #设置结束偏移量,默认全部, 可以用+3g
   #remove the signature, 一定要选择No
   #写入磁盘
   w
   #再次查看分区结果
   fdisk -l
   ```

4. 挂载扩容后的分区

   ```bash
   #查看文件系统
   df -h
   #查看循环设备情况,记下offset
   losetup
   #losetup -f  -o 上边的offset 上述第二个分区
   losetup -f  -o 333333 /dev/mmcblk02
   #再次查看循环设备
   losetup
   #挂载新的loop设备
   mount /dev/loop1  /mnt/loop1
   unmout /mnt/loop1
   #查看循环设备的文件格式
   lsblk -f
   #确认文件格式后
   #ext文件格式用以下
   resize2fs -f /dev/loop1
   #f2fs用以下命令
   resize.f2fs -f /dev/loop1
   
   ```

5. 重启生效

```bash
reboot
```

