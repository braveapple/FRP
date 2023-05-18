# FRP
内网穿透工具 FRP

## 服务端配置 frps.ini

```
[common]
# frp 监听的端口，默认是7000，可以改成其他的
bind_port = 7000
# 授权码，请改成更复杂的
token = 12345678

# frp 管理后台端口，请按自己需求更改
dashboard_port = 7500
# frp 管理后台用户名和密码，请改成自己的
dashboard_user = admin
dashboard_pwd = admin
enable_prometheus = true

# frp 日志配置
log_file = /var/log/frps.log
log_level = info
log_max_days = 3
```

## 客户端配置 frpc.ini

```
[common]
server_addr = xx.xx.xx.xx # 服务端公网 IP 地址
server_port = 7000
token = 12345678

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000

[web]
type = tcp
local_ip = 127.0.0.1
local_port = 6666
remote_port = 8081
```
