# Linux Lvm
>操作系统: Ubuntu 18.04

以/dev/sdb做演示。
## 创建PV
```shell script
pvcreate <physical_disk_path>
pvcreate /dev/sdb
```
## 创建VG
使用 **/dev/sdb** 创建卷组
```shell script
vgcreate <vg_name> <pv_name>
vgcreate vg_data /dev/sdb
```
## 创建LV
以刚建的卷组创建一个名为lv_test的逻辑卷
```shell script
lvcreate -l <lv_size>  -n <lv_name> <vg_name>
lvcreate -l 100%VG -n lv_test vg_data
```
## 格式化与挂载
逻辑卷建好后就可以格式化和挂载逻辑卷了
```shell script
mkfs.ext4 /dev/mapper/vg_data-lv_test
mkdir -p /data/test
mount /dev/mapper/vg_data-lv_test
```

最后记得将其加入/etc/fstab，服务器重启后自动挂载。

## 扩容
以加入/dev/sdc做演示。
```shell script
pvcreate /dev/sdc
vgextend vg_data /dev/sdc
lvextend -l 100%FREE /dev/mapper/vg_data-lv_test
resize2fs /dev/mapper/vg_data-lv_test
```
