* 安装

```shell script
docker run \
 -itd  \
 --hostname 172.16.xxx.xxx \
 -p 443:443 \
 -p 80:80 \
 -p 9922:22 \
 -v /data/gitlab/etc:/etc/gitlab  \
 -v /data/gitlab/log:/var/log/gitlab \
 -v /data/gitlab/opt:/var/opt/gitlab \
 --restart always \
 --privileged=true \
 --name gitlab \
 gitlab/gitlab-ce
```
