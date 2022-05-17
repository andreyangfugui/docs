## `win10` 解除端口占用

* 找到端口占用PID
```shell script
Netstat -ano|findstr "8080"
```

* 找到对应pid占用程序
```shell script
tasklist | findstr "pid"
```

* 终结程序
```shell script
tskill 10200
```
