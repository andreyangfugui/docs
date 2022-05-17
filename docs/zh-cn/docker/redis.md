## docker 搭建redis

```shell script
docker run --restart=always -itd --name redis -p 6379:6379 redis:6.2.5-alpine3.14
```
