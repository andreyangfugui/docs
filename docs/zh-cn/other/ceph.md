
## `cephadm` 安装`ceph v15`准备

参考安装文档, 有问题可以直接留言，大家一起交流，以下都是部署的比较基础的东西

```
https://docs.ceph.com/en/latest/cephadm/
http://www.dtmao.cc/news_show_1027422.shtml
```


> 用的阿里云的三台机器进行测试，其中每台机器挂载了一块40G的数据盘，系统盘也为40G，系统为centos7.8

* 三台服务器

| hostname | IP |
|------|------------|
| node1  | 172.16.2.186 |
| node2  | 172.16.2.184 |
| node3  | 172.16.2.184 |

* 安装`docker`

参考https://www.runoob.com/docker/centos-docker-install.html
```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```
* 设置hostname和修改hosts（每台服务器都需要设置hostname和修改hosts文件）

设置hostname
``` 
vim /etc/hostname 然后写入hostname即可

hostnamectl set-hostname node1
```
修改hosts(如下图所示)
```
vim /etc/hosts
```
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4771adca05e342b0a92ef2cc348e8a21~tplv-k3u1fbpfcp-watermark.image)

* 禁用防火墙

```shell
systemctl stop firewalld
systemctl disable firewalld
```

**接下来就开始我们的安装之旅把！**

## 获取`cephadm`

>`cephadm`其实就个`python3`的脚本而已

连接到`node1`

```shell
wget https://raw.githubusercontent.com/ceph/ceph/octopus/src/cephadm/cephadm
# 如果wget不到该文件 直接浏览器访问复制文件内容到新建文件cephadm即可

# 赋予执行权限
chmod +x cephadm
```

## 获取软件包
* 尽管独立脚本足以启动群集，但是cephadm在主机上安装命令很方便。要安装提供cephadm八达通发行版命令的软件包，请运行以下命令

```shell
./cephadm add-repo --release octopus
./cephadm install
```

* cephadm通过运行which以下命令来确认该路径现在位于您的PATH中：

```
which cephadm
```
* 成功的命令将返回以下内容：which cephadm

```
/usr/sbin/cephadm
```

## 引导新的集群

```
# 172.16.2.186为node1的IP
cephadm bootstrap --mon-ip 172.16.2.186
```
>安装完成后会返回一个dashboard页面 直接使用公网IP访问即可，主要确保安全组的端口畅通

## 将主机添加到集群中

要将每个新主机添加到群集，请执行两个步骤：

在新主机的根用户authorized_keys文件中安装群集的公共SSH密钥：

```
ssh-copy-id -f -i /etc/ceph/ceph.pub root@*<new-host>*
```
例如：
```
ssh-copy-id -f -i /etc/ceph/ceph.pub root@node1
ssh-copy-id -f -i /etc/ceph/ceph.pub root@node2
```
告诉Ceph，新节点是集群的一部分：
```
ceph orch host add *newhost*
```
例如：
```
ceph orch host add node1
ceph orch host add node1
```

5. 将磁盘转化为osd

有几种创建新OSD的方法：

告诉Ceph使用任何可用和未使用的存储设备：
```
ceph orch apply osd --all-available-devices
```
从特定主机上的特定设备创建OSD：
```
ceph orch daemon add osd *<host>*:*<device-path>*
```
例如：

```
ceph orch daemon add osd node1:/dev/vdb
```

>注意该磁盘必须是干净的，未被分区的，如果已经使用，可以参考下面的方法

部署完`ceph`之后
`cephadm shell` 进入容器内部， 然后执行擦除磁盘
```
# Zap (erase!) a device so it can be re-used
orch device zap <hostname> <path> [--force]
```
`example`
```
ceph orch device zap node1 /dev/sdb --force
```

## 部署rgw

**确保`ceph -s`的状态为`HEALTH_OK`**

`Cephadm`将`radosgw`部署为管理特定领域和区域的守护程序的集合。（有关领域和区域的更多信息，请参见Multi-Site。）

请注意，使用`cephadm`时，`radosgw`守护程序是通过监视器配置数据库而不是通过`ceph.conf`或命令行来配置的。如果该配置尚未就绪（通常在本
`client.rgw.<realmname>.<zonename>`
节中），那么radosgw守护程序将使用默认设置（例如，绑定到端口80）启动。

例如，要在node1、node2和node3上部署3个服务于myorg领域和us-east-1区域的rgw守护程序：

```shell
# 如果尚未创建领域，请首先创建一个领域：
radosgw-admin realm create --rgw-realm=myorg --default

# 接下来创建一个新的区域组：
radosgw-admin zonegroup create --rgw-zonegroup=default --master --default

# 接下来创建一个区域：
radosgw-admin zone create --rgw-zonegroup=default --rgw-zone=cn-east-1 --master --default

# 为特定领域和区域部署一组radosgw守护程序：
ceph orch apply rgw myorg cn-east-1 --placement="3 node1 node2 node3"
```

这一步完成之后，我们可以`ceph -s`看到

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f26b5432dc4c4795af9930ee00ee206a~tplv-k3u1fbpfcp-watermark.image)

接下来我们，为rgw创建一个用户和开启rgw的dashboard

> 参考文档 https://blog.csdn.net/qq_40017427/article/details/106235456

```
# 创建用户
radosgw-admin user create --uid=rgw --display-name=rgw --system

# 提供Dashboard证书 set-rgw-api-access-key set-rgw-api-secret-key是创建用户返回的

# 也可以使用 radosgw-admin user info --uid=rgw 查看
ceph dashboard set-rgw-api-access-key ZW1Y5IWDTB7K85P32H2A
ceph dashboard set-rgw-api-secret-key aehFzwMypF4V8Bm8A3baevHonEmu4E9a4oLZ1umh

# 禁用SSL 可禁用可不禁用
ceph dashboard set-rgw-api-ssl-verify False

# 启用rgw
ceph dashboard set-rgw-api-host node
ceph dashboard set-rgw-api-port 80
ceph dashboard set-rgw-api-scheme http
ceph dashboard set-rgw-api-user-id rgw

radosgw-admin period update --commit
```

验证见参考文档

做完之一🙅步之后可以重启一下三台服务器

使用`python`访问`ceph`对象存储， 如果要前端访问存储资源时候可以用HTML标签来访问，不能浏览器直接访问（前提可以把`bucket`的`acl`设置成`public-read`或者把存储对象设置成`public-read`）

参考文档

https://blog.csdn.net/nslogheyang/article/details/100115336

## 移除osd

```shell script
# 先将该osd reweight 到0，也就是将权重降低到0，让数据副本分散到其它osd上
ceph osd reweight 2 0.0
# 待集群重新恢复为ok后执行以下命令将osd踢出集群
service ceph stop osd.2
ceph osd out 2
ceph osd crush remove osd.2
ceph auth del osd.2
ceph osd rm osd.2
```

## RGW

设置rgw端口

```shell script
ceph config set client.rgw.myorg.cn-east-1 rgw_frontends 'beast port=7480'
```
