### NFS
> NFS就是Network File System的缩写，它最大的功能就是可以通过网络，让不同的机器、不同的操作系统可以共享彼此的文件。

Samba是在Linux系统上实现SMB（Session MessageBlock）协议的一个免费软件，以实现文件共享和打印机服务共享。 

> 系统:Ubuntu 18.04

* 安装软件

```shell script
apt install samba
```

* 配置
vim /etc/samba/smb.conf

在末尾追加

```shell script
[share]
    comment = share folder
    browseable = yes
    path = /srv/data
    create mask = 0700
    directory mask = 0700
    valid users = root
    public = yes
    available = yes
    writeable = yes
```

其中path=后面跟的是服务器共享目录，可根据需求更改

* 添加密码:
```shell script
smbpasswd -a root
```
输入访问用密码

* 重启服务
```shell script
systemctl restart smbd
```

### 定时备份
> 通过rsync和crontab配合，每天将A服务器上共享文件夹的内容备份到B服务器指定目录下。
* 安装软件
```shell script
apt install rsync
```

* 配置
    * A服务器作为服务端

        vim /etc/rsyncd.conf
        ```shell script
        #global settings 
        pid file=/var/rsync/rsync.pid
        port=873
        lock file=/var/rsync/lock.log
        log file=/var/rsync/rsync.log
        use chroot=no
        max connections=100
        uid=root
        gid=root
        timeout=3000

        [nfs]
        path=/srv/data/
        list=yes
        read only=false
        ignore errors=yes
        use chroot=yes
        auth users=root
        secrets file=/etc/rsyncd.secrets
        ```
    * B服务器作为客户端，采用默认配置即可。
    
    * A服务器配置密码
    vim /etc/rsyncd.secrets
    ```shell script
    root:$password # $password替换为A服务器配置的密码
    ```
    * 文件赋权
    ```shell script
    chmod 600 /etc/rsyncd.secrets
    ```

    * B服务器配置
    vim /etc/rsyncd.pass
    ```shell script
    $password # $password替换为A服务器配置的密码
    ```
    * 文件赋权

    ```shell script
    chmod 600 /etc/rsyncd.pass
    ```


* 启动服务
```shell script
systemctl restart rsync
```

* 测试

从B服务器拉取A服务器上的文件
```shell script
rsync -avH --password-file /etc/rsync.pass --delete root@A::nfs /nfs-backup/
# A替换为A服务器ip
```
    * 参数解释:
    * -a, --archive 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD。
    * -v, --verbose 详细模式输出。
    * -H, --hard-links 保留硬链结。
    * –delete 删除那些B服务器中有，A服务器上没有的文件。


* 定时备份
```shell script
crontab -e
00 00 * * * rsync -avH --password-file /etc/rsync.pass --delete root@A::nfs /nfs-backup/
# A替换为A服务器ip
```
保存退出即可。