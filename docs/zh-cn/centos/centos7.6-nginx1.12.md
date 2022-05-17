## 问题描述

> nginx反向代理502-Bad Gateway问题解决

在centos7.6 nginx 1.12下面不能使用proxy_pass转发请求


## 解决办法

```shell
getsebool httpd_can_network_connect
```

结果

`httpd_can_network_connect --> off`


```
setsebool -P httpd_can_network_connect 1
```

## 思路

可以看出是`selinux`的问题，可以通过禁用`selinux`或者上面的解法解决问题
