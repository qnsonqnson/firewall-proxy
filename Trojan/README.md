# Prepare
- You need correctly appoint your Domain to your Server IP, and DO NOT open CDN service at first        
- **Please pay attention to the marks on each line of the config files, and modify them as requested**      
# Build Environment 
Debian 9 && Ubuntu 16~18        
# Content
- download script
```bash
apt-get update && apt-get -y install socat         
wget -qO- get.acme.sh | bash       
source ~/.bashrc
```
- install script （please modify **yourdomain.com** to your domain）
```bash
acme.sh --issue --standalone -d yourdomain.com -k ec-256
mkdir /etc/trojan
acme.sh --installcert -d yourdomain.com --fullchain-file /etc/trojan/trojan.crt --key-file /etc/trojan/trojan.key --ecc
```
- install Nginx
```bash
apt update
apt install nginx
```
- modify config files（please modify **yourdomain.com** to your domain）
```bash
rm /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
vim /etc/nginx/conf.d/about.conf
```
- paste config below       
```bash
server {
    listen 127.0.0.1:80 default_server;
    server_name yourdomain.com;    #modify "your_domain.com" to your domain
    location / {
        proxy_pass proxy.com;         #modify to any website URL you want to disguise
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
    server_name ip.ip.ip.ip;      #modify to your server IP address
    return 301 https://yourdomain.com$request_uri;   #modify "your_domain.com" to your domain
}

server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
```
- install Trojan service
```bash
wget -qO- get.docker.com | bash
docker pull teddysun/trojan
cd /etc/trojan && vim config.json
```
- paste config below        
```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"   #modify to your password
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
- Start BBR Accelerate (A solotion to decrease network delay from Google) ：
```bash
sudo bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
sudo bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sudo sysctl -p
```
- Start Service     
```bash
nginx -s reload
docker run -d --name trojan --restart always --net host -v /etc/trojan:/etc/trojan teddysun/trojan
```

# Client
Android 6.0+ ：[Download](https://github.com/trojan-gfw/igniter/releases)                  

Windows 7.0+ ：[Download](https://github.com/Trojan-Qt5/Trojan-Qt5/releases)   
