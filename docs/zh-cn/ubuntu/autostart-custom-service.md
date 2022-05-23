## 自定义服务并自启

```shell script
cat >/etc/systemd/system/define.service <<EOF
[Unit]
Description=define
Documentation=https://define/
After=network.target
[Service]
Type=simple
User=root
ExecStart=define
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
systemctl restart define.service
systemctl enable define.service

systemctl start define.service
```

#### define需要替换为特定的命令，介绍，网址，服务名等。