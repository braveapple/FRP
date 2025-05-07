# FRP
内网穿透工具 FRP

假设我们有 A、B 和 C 三台机器，其中 A 和 C 机器在不同的局域网，B 机器具有公网 IP 地址。
由于 A 机器不能直接访问 C 机器，因此采用 A 和 C 机器都能访问的 B 机器作为代理服务端。
这样 A 机器可以通过 B 机器来访问 C 机器。

安装 FRP  
```
cd /opt
git clone https://github.com/braveapple/FRP.git
```

## 1. 服务端配置 -> B 机器

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
log_max_days = 30
```

在 `/etc/systemd/system` 目录下创建服务文件 frps.service
```
sudo vim /etc/systemd/system/frps.service
```

在 frps.service 文件中添加如下内容：
```
[Unit]
Description=frps
After=network.target
Wants=network.target

[Service]
# 执行失败后重启
Restart=on-failure
# 重启时间间隔
RestartSec=5
# frpc 启动命令，可以修改为自己的启动命令
ExecStart=/opt/frp/frps -c /opt/frp/frps.ini

[Install]
WantedBy=multi-user.target
```

设置开机自启并启动服务
```
# 刷新服务列表
systemctl daemon-reload
# 设置开机自启
systemctl enable frps
# 启动服务
systemctl start frps
```
由于服务可能会在开机时启动失败，因此在设置开机自启命令时，最好在 [Service] 中定义 Restart 和 RestartSec

## 2. 客户端配置 -> C 机器

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

## 3. 代理服务访问 -> A 机器

A 机器 ssh 访问 C 机器
```
# 6000 是 [ssh] 中的远程端口 remote_port
# test 是客户端的用户
# xx.xx.xx.xx 是服务端的公网 IP 地址
ssh -p 6000 test@xx.xx.xx.xx
```

A 机器访问 C 机器的 6666 端口的网页
```
# xx.xx.xx.xx 是服务端的公网 IP 地址
# 8081 是 [web] 中的远程端口 remote_port
http://xx.xx.xx.xx:8081
```

## 4. 常用的 systemctl 命令
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
