>建议配置: Ubuntu18 4C8G40G
```shell script
sudo su

echo "\
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

# deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

"> /etc/apt/sources.list

apt update
apt install -y sqlite3 sqlite gcc pkg-config libssl-dev libevent-dev
wget https://github.com/coturn/coturn/archive/refs/tags/docker/4.5.2-r11.tar.gz
tar zxvf 4.5.2-r11.tar.gz
cd coturn-docker-4.5.2-r11
mkdir /opt/coturn
./configure --prefix=/opt/coturn
make 
make install

ln -sf /opt/coturn/bin/turnserver  /usr/local/bin/turnserver
ln -sf /opt/coturn/bin/turnadmin  /usr/local/bin/turnadmin

echo "\
relay-device=ens3                 # ens3替换为虚拟机对应网卡
listening-ip=$ip                  # $ip替换为虚拟机本机ip
listening-port=3478
relay-ip=$ip                      # $ip替换为虚拟机本机ip
pidfile="/var/run/turnserver.pid"
min-port=49152
max-port=65535
user=admin:admin
cli-password=qwerty 
" > /opt/coturn/etc/turnserver.conf

turnadmin -k -u admin -p admin -r admin
turnserver -o -a -f -user=admin:admin -r admin
```

访问 https://webrtc.github.io/samples/src/content/peerconnection/trickle-ice/

在ICE servers一栏下输入:
```shell script
STUN or TURN URI: stun:$ip:3478 # $ip替换为虚拟机本机ip

TURN username: admin

TURN password: admin
```
点击Add server,再次输入:
```shell script
STUN or TURN URI: turn:$ip:3478 # $ip替换为虚拟机本机ip

TURN username: admin

TURN password: admin
```
点击Add server

拉到最下面 点击Gather candidates，address输出三栏即成功

服务开机自启:
```shell script
echo "\
[Install]
WantedBy=multi-user.target
" >> /lib/systemd/system/rc-local.service

systemctl enable rc-local
touch /etc/rc.local
chmod +x /etc/rc.local
echo "\
#!/bin/bash
turnadmin -k -u admin -p admin -r admin
turnserver -o -a -f -user=admin:admin -r admin
" > /etc/rc.local
```