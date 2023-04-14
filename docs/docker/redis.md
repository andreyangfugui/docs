* docker 搭建redis

```shell script
docker run --restart=always -itd --name redis -p 6379:6379 -v <redis data path>:/data redis:6.2.5-alpine3.14 --requirepass xxxxx
    (
        xxxxx替换为随机生成强密码
        <redis data path>替换为本机指定存放redis的路径
    )
    
```

* 如何给已运行的无密码redis容器配置密码 <br /> 思路: <br />将容器内redis已有数据拷到本地指定数据目录下 -> 停容器 -> 重跑一个带密码且挂载数据目录的容器 -> 删除旧容器。
```shell script
mkdir -p <redis data path>
docker cp <container name>:/data/dump.rdb <redis data path>
docker stop <container name>
docker run --restart=always -itd --name <container name>2 -p 6379:6379 -v <redis data path>:/data redis:6.2.5-alpine3.14 --requirepass xxxxx
    (
        xxxxx替换为随机生成强密码
        <redis data path>替换为本机指定存放redis的路径
    )
```

确认新容器的密码认证和数据无误 <br /> 删除原redis容器(重要,如果旧容器配有自启重启后会发生冲突)
```shell script
docker rm <container name>
```