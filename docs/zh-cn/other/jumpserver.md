## 获取软件包

https://www.jumpserver.org/

## 安装

* You can use the following command to start, and then visit
cd /opt/jumpserver-installer-v2.14.2
./jmsctl.sh start

*  Other management commands
./jmsctl.sh stop
./jmsctl.sh restart
./jmsctl.sh backup
./jmsctl.sh upgrade
For more commands, you can enter ./jmsctl.sh --help to understand

* Web access
http://172.16.xxx.xxx:80
Default username: admin  Default password: admin

* SSH/SFTP access
ssh -p2222 admin@172.16.xxx.xxx
sftp -P2222 admin@172.16.xxx.xxx

## 官方文档

* More information
Official Website: https://www.jumpserver.org/
Documentation: https://docs.jumpserver.org/
