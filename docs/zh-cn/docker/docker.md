* 安装

```shell script
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/debian buster stable" > /etc/apt/sources.list.d/docker.list
```

* 加速器

```shell script
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
sudo systemctl enable docker
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://2r522sjx.mirror.aliyuncs.com"],
  "log-driver":"json-file",
  "log-opts": {"max-size":"1024m"}
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

* 迁移方案

两种方法迁移 Docker 的默认安装(存储)目录
如果是通过 Ubuntu 的 apt-get 安装的 Docker，默认的安装目录应该是：/var/lib/docker。

为了完全确定，可以使用以下的命令查询真正的安装路径：
```shell script
sudo docker info | grep "Docker Root Dir"
```
前置工作
下文以 /store/software/docker 这个路径作为要迁移的新 Docker 安装(存储)目录

在开始迁移之前，首先复制原 Docker 安装(存储)目录到新的路径下：
```shell script
cp -a /var/lib/docker /store/software/
```
然后备份原目录数据：
```shell script
mv -u /var/lib/docker /var/lib/docker.bak
```
### 方法一：软链接

通过把 Docker 的原安装(存储)目录移动到自定义的其他路径下，然后在原位置下建立软链接是最简单快捷的方式。

首先，关停 Docker 服务：
```shell script
sudo systemctl stop docker
# or
sudo service docker stop
```
接着，新建一个 /var/lib/docker 的软链：
```shell script
sudo ln -fs /store/software/docker /var/lib/docker
```
最后，重启 Docker 服务：
```shell script
sudo systemctl start docker
# or
sudo service docker start
```
启动 Docker 之后，以后你 pull image 和 run container，Docker 写入的路径依然是 /var/lib/docker，但是因为软链的设置，实际已经是往新的目录写入了。至此，完成了 Docker 安装(存储)目录的迁移。

### 方法二：直接修改 Docker 配置文件

Docker 版本 < v17.05.0
因为 dockerd 运行 Docker 服务的时候可以通过参数 graph 指定镜像和容器存放路径，比如：–graph=/var/lib/docker，我们只需要修改配置文件指定启动参数即可。

Docker 的配置文件可以设置大部分的后台进程参数，在各个操作系统中的存放位置不一致，在 Ubuntu 中的位置是：/etc/default/docker，在 CentOS 中的位置是：/etc/sysconfig/docker。

* 如果是 CentOS 则添加下面这行：
OPTIONS=--graph="/store/software/docker" --selinux-enabled -H fd://

* 如果是 Ubuntu 则添加下面这行（因为 Ubuntu 默认没开启 selinux）：
OPTIONS=--graph="/store/software/docker" -H fd://
* 或者
DOCKER_OPTS="-g /store/software/docker"
Docker 版本 >= v17.05.0
因为 Docker 官方在这个发行版本就 deprecated 了 graph 这个 feature，所以如果你机器上安装的 Docker 版本 >= v17.05.0，则无法通过在 /etc/default/docker 配置文件中指定 graph 参数来修改 Docker 的默认安装(存储)目录了，具体参见官网文档：Docker Docs。

好在天无绝人之路，新版本的 Docker 还有其他方式可以达到我们修改安装(存储)目录的目的：通过修改(新建)/etc/docker/daemon.json，指定 data-root 参数的值。

按如下操作：
```shell script
vim /etc/docker/daemon.json
```
加入
```shell script
{
    "data-root": "/store/software/docker",
    "storage-driver": "overlay2" # 这个是 Docker 是默认设置，这里也可以不用加
}
```
重启 Docker & 清理原安装(存储)目录
最后，重启 Docker 服务：

```shell script
sudo systemctl restart docker
# or
sudo service docker restart
```

通过上述任一方法完成迁移之后，在确认 Docker 能正常工作之后，删除备份的原目录数据：

```shell script
rm -rf /var/lib/docker.bak
```

### 迁移方案参考链接

https://ld246.com/article/1566017283738
