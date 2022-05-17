## logstash 安装 oss(腾讯云 阿里云)

```
/usr/share/logstash/bin/logstash-plugin install logstash-output-oss
```

```
  s3{
     access_key_id => "crazy_key"             (optional)
     secret_access_key => "monkey_access_key" (optional)
     region => "eu-west-1"                    (optional, default = "us-east-1")
     bucket => "your_bucket"                  (required)
     size_file => 2048                        (optional) - Bytes
     time_file => 5                           (optional) - Minutes
     codec => "plain"                         (optional)
     canned_acl => "private"                  (optional. Options are "private", "public-read", "public-read-write", "authenticated-read", "aws-exec-read", "bucket-owner-read", "bucket-owner-full-control", "log-delivery-write". Defaults to "private" )
   }
```

## s3

```
  s3 {
        endpoint => "https://xx-1251612638.cos.ap-nanjing.mxxcloud.com"
        access_key_id => "access_key_id"
        secret_access_key => "access_key_secret" 
        region => "ap-nanjing"
        bucket => "xx-1251612638"
        time_file => 1
        codec => "json"
        canned_acl => "public-read"
   }
```

## oss

```
  oss {
        endpoint => "https://xx-1251612638.cos.ap-nanjing.mxxcloud.com"
        bucket => "xx-1251612638"
        access_key_id => "access_key_id"
        access_key_secret => "access_key_secret"
        region => "ap-nanjing"
        prefix => "oss/database"                         
        recover => true                                      
        rotation_strategy => "size_and_time"                  
        time_rotate => 1                                     
        size_rotate => 1000
        temporary_directory => "/home/ubuntu"                            
        encoding => "gzip"                                 
        additional_oss_settings => {
            max_connections_to_oss => 1024                      
            secure_connection_enabled => true                  
        }
        codec => json {
            charset => "UTF-8"
        }
   }
```

## kafka

```
    kafka {
         bootstrap_servers => "localhost:9092" #kafka服务器地址
         topic_id => "log_topic"
         batch_size => 5
         codec => "json" #写入的时候使用json编码，因为logstash收集后会转换成json格式
    }
```

## 完整配置

```
input {
    syslog {
        port => 514
    }
}

filter {
    prune {
        whitelist_names => [ "message" ]
   }
}
output {
   stdout { }

   s3 {
        endpoint => "https://cos.ap-nanjing.mxxcloud.com"
        access_key_id => "access_key_id"
        secret_access_key => "access_key_secret"
        region => "ap-nanjing"
        bucket => "xx-1251612638"
        time_file => 1
        codec => "json"
        canned_acl => "public-read"
   }
}
```

## kafka and s3(tencent oss)

```
input {
    syslog {
        port => 5514
    }
}

filter {
    prune {
        whitelist_names => [ "message", "logsource", "program", "@timestamp" ]
   }
      if ([program] != "log" and [program] != "user-manager") {
       drop {}
   }

}

output {
    if [message] =~ /^log_on_line.*/ {
        kafka {
            bootstrap_servers => "localhost:9092" #kafka服务器地址
            topic_id => "log_topic"
            batch_size => 5
            codec => "json" # 写入的时候使用json编码，因为logstash收集后会转换成json格式
       }
    }
    if [message] =~ /^log_off_line.*/ {
        s3 {
            endpoint => "https://cos.ap-nanjing.mxxcloud.com"
            access_key_id => "access_key_id"
            secret_access_key => "access_key_secret"
            region => "ap-nanjing"
            bucket => "xx-1251612638"
            time_file => 1
            codec => "json"
            canned_acl => "public-read"
        }
    }
}
```

## rabbmitmq and s3(tencent oss)

```
input {
    syslog {
        port => 5514
    }
}

filter {
    prune {
        whitelist_names => [ "message", "logsource", "program", "@timestamp" ]
    }

    if []
}

output {
    if [message] =~ /^log_on_line.*/ {
        rabbitmq {
            exchange => "xx-exchange"
            host => "localhost"
            exchange_type => "direct"
            key => "logstash"
            user => "admin"
            password => "admin"
            heartbeat => 0
        }
    }
    if [message] =~ /^log_off_line.*/ {
        s3 {
            endpoint => "https://cos.ap-nanjing.mxxcloud.com"
            access_key_id => "access_key_id"
            secret_access_key => "access_key_secret"
            region => "ap-nanjing"
            bucket => "xx-1251612638"
            time_file => 1
            codec => "json_lines"
            canned_acl => "public-read"
        }
    }
}
```

```
input {
    syslog {
        port => 5514
    }
}

filter {
    prune {
        whitelist_names => [ "message", "logsource", "program", "@timestamp" ]
    }
    mutate {  
        gsub => [ "message", "\n", "" ]  
    }  
}

output {
    stdout {}

        rabbitmq {
            exchange => ""
            host => "localhost"
            exchange_type => "direct"
            key => "python-test"
            user => "admin"
            password => "admin"
        }
}
```

## rsyslog

> ubuntu下面记得再50-default.conf的字符集前面 如 20-docker.conf

```
template(name="DynaFile" type="string" string="/var/log/syslogs/%programname%.log")
  
if $programname contains "user" then {
    action(type="omfile" dynaFile="DynaFile")
    stop
}
```

```
filter {
    prune {
        whitelist_names => [ "message", "logsource", "program", "@timestamp" ]
   }
}
```

```
docker run --name mongo-test --privileged=true -v /home/CveManager/docker-cve/mongo.bk/mongo-entrypoint/:/docker-entrypoint-initdb.d/ -p 8080:27017 --restart=always -d mongo  --auth
```

## Config Syslog

```
input {
    tcp {
        port => 5514
        type => syslog
    }
}

filter {
    if [type] == "syslog" {
        grok {
            match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:[%{POSINT:syslog_pid}])?: %{GREEDYDATA:syslog_message}" }
            add_field => [ "received_at", "%{@timestamp}" ]
            add_field => [ "received_from", "%{host}" ]
        }
        date {
            match => [ "syslog_timestamp", "MMM d HH:mm:ss.SSS", "MMM dd HH:mm:ss.SSS" ]
        }
    }
}
output {
    stdout {}
}
```


```
Jun 21 10:38:30 log log {"a1": 9992}

%{MONTH:month} %{MONTHDAY:day} %{TIME:time} %{HOSTNAME:hostname} %{HOSTNAME:program} %{DATA:data}
```


```
input {
    syslog {
        port => 5514
    }
}

filter {
    json {
        source => "message"
    }
    prune {
        whitelist_names => [ "msg", "logsource", "program", "time", "level", "type" ]
    }
    mutate {
        remove_field => ["@timestamp", "timestamp"]
    }
}

output {
    if [type] == "real-time" {
        rabbitmq {
            exchange => "xx-exchange"
            host => "localhost"
            exchange_type => "direct"
            key => "logstash"
            user => "admin"
            password => "admin"
        }
    }

    if [type] == "off-line" {
        elasticsearch {
            hosts => ["172.16.102.26"]
            index => "xx-log"
        }
    }
    stdout {}
}
```



```
input {
    kafka {
        bootstrap_servers => ["localhost:9200"]
        client_id => "test"
        group_id => "test"
        auto_offset_reset => "earliest" //从最新的偏移量开始消费
        consumer_threads => 5
        topics => ["echo-test"] //数组类型，可配置多个topic
    }
}

output {
    s3 {
        endpoint => "https://cos.ap-nanjing.mxxcloud.com"
        access_key_id => "access_key_id"
        secret_access_key => "access_key_secret"
        region => "ap-nanjing"
        bucket => "xx-1251612638"
        time_file => 1
        codec => "json_lines"
        canned_acl => "public-read"
    }
}


input {
    kafka {
        bootstrap_servers => ["localhost:9092"]
        client_id => "hk"
        group_id => "hk"
        auto_offset_reset => "earliest"
        consumer_threads => 20
        topics => ["echo"]
    }
}

output {
    stdout {}
    s3 {
        endpoint => "https://cos.ap-nanjing.mxxcloud.com"
        access_key_id => "access_key_id"
        secret_access_key => "access_key_secret"
        region => "ap-nanjing"
        bucket => "xx-1251612638"
        time_file => 1
        rotation_strategy => "time"
        codec => "json_lines"
        encoding => "gzip"
        canned_acl => "public-read"
    }
}
```

