ubuntu下格式化U盘

ubuntu下格式化U盘

有分区，要将分区先取消挂载
```shell
sudo umount /dev/sdb
# 清除U盘所有数据，可以使用下面的方式格式化。
# sudo dd if=/dev/zero of=/dev/sdb bs=4k && sync
```

1. 查看分区
```shell
sudo fdisks -l
```
2. 卸载U盘
```shell
sudo umount /dev/sdb1
```
3. 格式化
```shell
sudo apt-get install exfat-utils
sudo mkfs.exfat /dev/sdb1
sudo mkfs.ntfs -F /dev/sdc # 能被anroid识别。加-F是强制执行
```

格式化的时候出现错误：`/dev/sdc is entire device, not just one partition.`
```shell
sudo mkfs.ntfs -F /dev/sdc 
```

