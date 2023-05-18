# FRP
内网穿透工具 FRP

## 服务端配置

配置 frps.ini 
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

## 客户端配置

配置 frpc.ini
```
[common]
# 服务端公网 IP 地址
server_addr = xx.xx.xx.xx
server_port = 7000
token = 12345678

# ssh 代理
[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000

# web 代理
[web]
type = tcp
local_ip = 127.0.0.1
local_port = 6666
remote_port = 8081
```

在 `/etc/systemd/system` 目录下创建服务文件 frpc.service
```
sudo vim /etc/systemd/system/frpc.service
```

在 frpc.service 文件中添加如下内容：
```
[Unit]
Description=frpc
After=network.target
Wants=network.target

[Service]
# 执行失败后重启
Restart=on-failure
# 重启时间间隔
RestartSec=5
# frpc 启动命令，可以修改为自己的启动命令
ExecStart=/opt/frp/frpc -c /opt/frp/frpc.ini

[Install]
WantedBy=multi-user.target
```

设置开机自启并启动服务
```
# 刷新服务列表
systemctl daemon-reload
# 设置开机自启
systemctl enable frpc
# 启动服务
systemctl start frpc
```
由于服务可能会在开机时启动失败，因此在设置开机自启命令时，最好在 [Service] 中定义 Restart 和 RestartSec

## 常用的 systemctl 命令
```
# 关闭开机自启
systemctl disable frpc
# 停止服务
systemctl stop frpc
# 重启服务
systemctl restart frpc
# 查看状态
systemctl status frpc
# 查看是否设置开机自启
systemctl is-enabled frpc
```
