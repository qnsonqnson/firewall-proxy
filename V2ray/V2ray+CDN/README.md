# 准备工作
- 你需要有一个自己的域名，会正确的设置 `DNS解析` ，如果不会请自行 GOOGLE
- 本配置 **只支持** 最高安全级别的 **TLS1.3** ，为你的数据安全保驾 
- **请注意配置中后面的备注部分，按要求修改**
# 配置环境
纯净的 Debian 9 系统
# 配置内容
- 安装基础工具  
```bash
apt-get update && apt-get -y install socat wget screen
```
- 安装证书生成脚本  
```bash
wget -qO- get.acme.sh | bash 
source ~/.bashrc
```
- 安装证书  (**your_domain.com** 改为你的域名）
```bash
acme.sh --issue --standalone -d your_domain.com -k ec-256
mkdir /etc/v2ray
acme.sh --installcert -d your_domain.com --fullchain-file /etc/v2ray/v2ray.crt --key-file /etc/v2ray/v2ray.key --ecc
```
- 安装V2ray 
```bash 
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
bash <(curl -L -s https://install.direct/go.sh)
```
- 安装密码套件  （如果中途失去连接可用 **screen -R openssl** 恢复当前窗口，脚本中的选项 **全部填 n**）
```bash
screen -S openssl
cd /tmp && wget --no-check-certificate https://raw.githubusercontent.com/stylersnico/nginx-openssl-chacha/master/build.sh && sh build.sh
```
- 编辑 v2ray 配置 
```bash
vim /etc/v2ray/config.json
```
- 复制配置  
```bash
{
  "log": {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "port": 10000,
      "listen":"127.0.0.1",
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811",    #更改id
            "alterId": 60     #更改alterID
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
        "path": "/your_path"   #更改路径
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```
- 修改 Nginx 配置 
```bash
mkdir /etc/nginx/conf.d
vim /etc/nginx/conf.d/about.conf
```
- 复制配置  
```bash
server {
    listen 443 ssl http2;                                                       
    ssl_certificate       /etc/v2ray/v2ray.crt;  
    ssl_certificate_key   /etc/v2ray/v2ray.key;
    ssl_protocols         TLSv1.3;                    
    ssl_ciphers           ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4:!DH:!DHE;
    ssl_prefer_server_ciphers on;

    listen 80;
    server_name  your_domain.com;    #改为你的域名
    location / {
        proxy_pass https://proxy.com;     #改为你想伪装的网址
        proxy_redirect     off;
        proxy_connect_timeout      75; 
        proxy_send_timeout         90; 
        proxy_read_timeout         90; 
        proxy_buffer_size          4k; 
        proxy_buffers              4 32k; 
        proxy_busy_buffers_size    64k; 
        proxy_temp_file_write_size 64k; 
     }

    location /your_path {       ##改为你在上面修改的路径
        proxy_redirect off;
        proxy_pass http://127.0.0.1:10000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_read_timeout 300s;
    }
}
server {
    listen 127.0.0.1:80;
    server_name ip.ip.ip.ip;    #改为你服务器的 IP 地址
    return 301 https://your_domain.com$request_uri;    #改为你的域名
}

server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
  }
```
- 启动服务  
```bash 
nginx -s reload
service v2ray restart
```
- 开启 BBR 加速 
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# 客户端配置

![2](https://github.com/charlieethan/firewall-proxy/blob/master/photos/1.jpg)

**yourdomain**填你的域名 ，**id**和**alterId**填你上面设置的  
**Path**填上面设置的路径 ，其余部分照抄即可
# 客户端
Windows系统: [点击下载](https://github.com/2dust/v2rayN/releases)

Android系统: [点击下载](https://github.com/2dust/v2rayNG/releases) 

