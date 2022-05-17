
## 简介

> 在docker下面，进入docker里面 切换到  /opt/kafka/bin 目录

```
# 生产消息
./kafka-console-producer.sh --broker-list localhost:9092 --topic log_topic

# 消费消息
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic log_topic --from-beginning
```
