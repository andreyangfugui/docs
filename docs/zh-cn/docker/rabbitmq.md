# rabbitmq以及tsl 集锦

## docker部署

```
docker run --restart=always -itd --name rabbitmq-prod -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=qMfyBOgmDPkX2d2bw7QkmF6Gxfs0CUnX -p 15672:15672 -p 5672:5672 rabbitmq:management
```

## 权限控制

```
rabbitmqctl [--node <node>] [--longnames] [--quiet] set_permissions [--vhost <vhost>] <username> <conf> <write> <read>

conf 主要匹配exchange  write（exchange queue） read（exchange queue）

rabbitmqctl set_permissions -p /test zs '^echo$' '^(echo|ttt)$' '^(echo|ttt)$'
```

```shell script
docker run -dit --name rabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=9U8k8L10OiaNOEX6yaMSUjwfvzVXNhPm -p 15672:15672 -p 5672:5672 rabbitmq:management

docker run -dit --name rabbitmq-tsl -p 15672:15672 -p 5672:5672 rabbitmq:management

docker run --restart=always -itd  --name rabbit-tsl -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=123456 -v /home/ubuntu/rabbitmq/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf -v /home/ubuntu/rabbitmq/keys:/etc/rabbitmq/keys -p 18672:15672 -p 8672:5672 rabbitmq:management


docker run --restart=unless-stopped -d  \
        -p 8672:5672 -p 18672:15672 -p 5671:5671  \
        -v  /home/ubuntu/rabbitmq-prod/rabbitmq/:/etc/rabbitmq/   \
        --name rabbitmq \
        rabbitmq:management
```


## 注意 

当tsl配置密码的时候 rabbitmq中也需要配置密码

```shell script
cp /root/tls-gen/basic/result/ca_certificate.pem .
cp /root/tls-gen/basic/result/server_certificate.pem .
cp /root/tls-gen/basic/result/server_key.pem .


cp /root/tls-gen/basic/result/ca_certificate.pem .
cp /root/tls-gen/basic/result/client_certificate.pem .
cp /root/tls-gen/basic/result/client_key.pem .
```

> [参考连接 Github](https://github.com/Nepitwin/RabbitSSL)


## 配置文件 `rabbitmq.conf`

```
default_user  = admin
default_pass  = admin
#default_vhost = /

listeners.tcp = none
listeners.ssl.default = 5671

ssl_options.cacertfile           = /etc/ssl/rabbit/ca_certificate.pem
ssl_options.certfile             = /etc/ssl/rabbit/server_certificate.pem
ssl_options.keyfile              = /etc/ssl/rabbit/server_key.pem
ssl_options.verify               = verify_peer
ssl_options.password             = 123456
ssl_options.fail_if_no_peer_cert = true
```
