# 进程监控

## 统计进程打开句柄数

```shell script
lsof -n | awk '{print $2}' | sort | uniq -c | sort -nr | more 
```

## 统计进程对应线程数

```shell script
ps -eLf | grep $pid | grep -v grep | wc -l
$pid 替换为进程对应pid号或者进程名
```