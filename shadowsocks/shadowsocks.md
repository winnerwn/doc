## 安装ssserver
```
pip install shadowsocks


添加配置文件

{
"server":"my_server_ip",
"server_port":8388,
"local_address": "127.0.0.1",
"local_port":1080,
"password":"mypassword",
"timeout":300,
"method":"aes-256-cfb",
"fast_open": false,
"workers": 1
}
```

## 安装sslocal
```
sslocal -s server_ip -p server_port  -l 1080 -k password -t 600 -m aes-256-cfb
```

## 启动本地流量转发

yum install privoxy -y

vim /etc/privoxy/config
```
# 转发地址
forward-socks5   /               127.0.0.1:1080 .
# 监听地址
listen-address  localhost:8118
# local network do not use proxy
forward         192.168.*.*/     .
forward            10.*.*.*/     .
forward           127.*.*.*/     .
```

systemctl start privoxy.service



