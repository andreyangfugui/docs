### **CMAK(Cluster Manager for Apache Kafka)**

是由 Yahoo 开源的 Kafka 集群管理平台。我们可能听到更多的是 kafka-manager。

>因为误用了 Apache 的商标，所以才从 kafka-manager 改名为 CMAK。

* Cmak 前置要求:
    * kafka
    * JDK 11+
    * ZooKeeper

这里以搭建cmak 3.0.0.5为例:
```shell scrpit
wget https://github.com/yahoo/CMAK/releases/download/3.0.0.5/cmak-3.0.0.5.zip
unzip cmak-3.0.0.5.zip
mv cmak-3.0.0.5 cmak
```

修改cmak 配置文件application.conf(conf/application.conf)
```shell script
kafka-manager.zkhosts="172.16.xx.xx:2181,172.16.xx.xx:2181,172.16.xx.xx:2181"
cmak.zkhosts="172.16.xx.xx:2181,172.16.xx.xx:2181,172.16.xx.xx:2181"
```

后台启动cmak
```shell script
nohup ./bin/cmak -Dconfig.file=/opt/cmak-3.0.0.5/conf/application.conf -Dhttp.port=8080 -java-home /usr/lib/jvm/java-11-openjdk-amd64 &
```
* 参数解释：
    * -Dconfig.file：指明 CMAK 配置文件路径
    * -Dhttp.port：Web监听端口，默认9000端口
    * -java-home：指定 JDK 路径，不指定会使用默认java路径。这里由于需要用 JDK11，而服务器上默认使用JDK8，所以需要指定 JDK11 的路径。