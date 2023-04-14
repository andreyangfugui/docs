>* ES 默认没有安全策略，数据容易出现被人随意访问，修改甚至删除的情况，因此我们需要引入安全策略以确保数据安全，以下为初期搜集到的安全策略配置方案:
>    * 关闭外网，采用Nginx配置转发;
>    * X-pack;
>    * shield权限管理(收费);
>    * SearchGuard，search-guard是shield的替代品，提供加密，身份验证和授权，基于search guard SSL;
>    * Nginx转发需要额外引入nginx，相当于在ES应用上层进行处理; shield收费, SearchGuard需要额外引入组件,安全需求级别不是特别高就不考虑;
>    * 所以我们选择了X-pack认证方式，使用ES自带的工具组件来完成对Elasticsearch集群的安全访问


这里以一个正在运行的es集群为例，说明如何引入TLS和角色管理 \
首先是ip：
```shell script
192.168.0.2 es01
192.168.0.3 es02
192.168.0.4 es03
```
我们到es01这台机器上生成TLS所需的证书，大致流程如下: \
编辑一个yml文件，指定各个角色\
(比如需要es内部通信节点,kibana监控所在节点，以及一个ca证书用于程序访问es) \
再让es根据yml文件生成对应的证书
```shell script
cd <es path> 
    (<es path>替换为es安装目录所在路径)
vim instance.yml
    (编辑好后保存退出)
./bin/elasticsearch-certutil cert ca --pem --in instances.yml --out /root/certs.zip
```
###### 其中,instances.yml内容示例如下:
```shell script
instances:
      - name: "es"
        dns: ['es01', 'es02', 'es03']
      - name: "kibana"
        dns: ['es01']
```
将生成的证书压缩包解压缩到特定目录,更改文件权限
```shell script
unzip -d /root/certs.zip <es path>/config/certs
chown -R <es user> <es path>/config/certs
cp -R <es path>/cofig/certs <kibana path>/config/
chown -R <kibana user> <kibana path>/config/certs
    (
        <es user>替换为es服务进程对应的用户名
        <kibana user>替换为kibana服务进程对应的用户名
        <kibana path>替换为kibana安装目录所在路径
    )
```
修改es和kibana配置，这里仅将变动的配置列出以供参考:

###### elasticsearch.yml
```shell script
xpack.security.enabled: true
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.key: <es path>/config/certs/es/es.key
xpack.security.http.ssl.certificate: <es path>/config/certs/es/es.crt
xpack.security.http.ssl.certificate_authorities: <es path>/config/certs/ca/ca.crt
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.key: <es path>/config/certs/es/es.key
xpack.security.transport.ssl.certificate: <es path>/config/certs/es/es.crt
xpack.security.transport.ssl.certificate_authorities: ["<es path>/config/certs/ca/ca.crt"]
```
###### kibana.yml
```shell script
elasticsearch.username: elastic
elasticsearch.password: <es pass>
    (将<es pass>替换为前面设置的elastic用户密码)
xpack.security.enabled: True
server.ssl.enabled: True
server.ssl.certificate: /opt/kibana/config/certs/kibana/kibana.crt
server.ssl.key: /opt/kibana/config/certs/kibana/kibana.key
elasticsearch.ssl.verificationMode: certificate
elasticsearch.ssl.certificateAuthorities: ["/opt/kibana/config/certs/ca/ca.crt"]
```
证书和配置分发及赋权:
```shell script
scp -r <es cert directory path> root@<es host>:<es path>/config
scp -r <elasticsearch.yml path> root@<es host>:<es path>/config
ssh <es host>
chown -R <es user> <es cert directory path>
chown <es user> <elasticsearch.yml path>
```

增加es的管理员用户并设置密码:
```shell script
./bin/elasticsearch-setup-passwords interactive (手动设置各个用户密码)
./bin/elasticsearch-setup-passwords auto (生成随机密码，会将生成的密码打印到屏幕上，记得保存)
```
依次重启es集群和kibana服务以使配置项生效

>* 验证手段:
>  * curl http://es01:9200 (此条应返回empty reply from sever)
>  * curl -l -k -uelastic@\<es pass\> https://es01:9200
>  * 以elastic账号登录kibana操作es集群

