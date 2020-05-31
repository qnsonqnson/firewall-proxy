# 准备工作
1.你需要拥有一个自己的**域名**，并会正确设置**DNS解析**，如果你不知道以上两个名词的含义，请自行Google学习     
2.搭建环境 ：纯净的 Debian 9 && Ubuntu 16~18 系统

# 开始部署
- 下载证书申请脚本
```bash
apt-get update && apt-get -y install socat         
wget -qO- get.acme.sh | bash       
source ~/.bashrc
```
- 安装脚本 （注意：**yourdomain.com**请替换为你自己的域名）
```bash
acme.sh --issue --standalone -d yourdomain.com -k ec-256
mkdir /etc/trojan
acme.sh --installcert -d yourdomain.com --fullchain-file /etc/trojan/trojan.crt --key-file /etc/trojan/trojan.key --ecc
```
- 安装Nginx
```bash
apt update
apt install nginx
```
- 移除默认代理组 （注意：**yourdomain.com**请替换为你自己的域名,执行后按**Ctrl+X**退出）
```bash
rm /etc/nginx/sites-enabled/default
nano /etc/nginx/sites-available/yourdomain.com
```
- 添加新的代理组并编辑配置文件 （注意：**yourdomain.com**请替换为你自己的域名）
```bash
ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
vim /etc/nginx/conf.d/about.conf
```
- 将以下内容粘贴 
```bash
server {
    listen 127.0.0.1:80 default_server;
    server_name yourdomain.com;  #改为你的域名
    location / {
        proxy_pass proxy.com;   #改为你想伪装的网站
        proxy_redirect     off;
        proxy_connect_timeout      75; 
        proxy_send_timeout         90; 
        proxy_read_timeout         90; 
        proxy_buffer_size          4k; 
        proxy_buffers              4 32k; 
        proxy_busy_buffers_size    64k; 
        proxy_temp_file_write_size 64k; 
    }

}

server {
    listen 127.0.0.1:80;
    server_name ip.ip.ip.ip;   #改为你服务器的IP
    return 301 https://yourdomain.com$request_uri;   #改为你的域名
}

server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
```
- 安装Trojan并编辑配置文件
```bash
wget -qO- get.docker.com | bash
docker pull teddysun/trojan
cd /etc/trojan && vim config.json
```
- 将以下内容粘贴
```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"   #改为你的密码
    ],
    "log_level": 1,
    "ssl": {
        "cert": "/etc/trojan/trojan.crt",
        "key": "/etc/trojan/trojan.key",
        "key_password": "",
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384",
        "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
        "prefer_server_cipher": true,
        "alpn": [
            "http/1.1"
        ],
        "reuse_session": true,
        "session_ticket": false,
        "session_timeout": 600,
        "plain_http_response": "",
        "curves": "",
        "dhparam": ""
    },
    "tcp": {
        "prefer_ipv4": false,
        "no_delay": true,
        "keep_alive": true,
        "reuse_port": false,
        "fast_open": false,
        "fast_open_qlen": 20
    },
    "mysql": {
        "enabled": false,
        "server_addr": "127.0.0.1",
        "server_port": 3306,
        "database": "trojan",
        "username": "trojan",
        "password": ""
    }
}
```
- 启动BBR加速
```bash
sudo bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
sudo bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sudo sysctl -p
```
- 启动服务
```bash
nginx -s reload
docker run -d --name trojan --restart always --net host -v /etc/trojan:/etc/trojan teddysun/trojan
```

# 客户端
安卓系统 ：[点击下载](https://github.com/trojan-gfw/igniter/releases)          
> 配置如下： **地址**填你的域名，**端口**填 443 ，**密码**填你刚才设置的密码，其他选项无需更改        

Windows系统 ：[点击下载](https://github.com/Trojan-Qt5/Trojan-Qt5/releases)   
> 项目地址 & 使用说明 ：https://github.com/TheWanderingCoel/Trojan-Qt5
