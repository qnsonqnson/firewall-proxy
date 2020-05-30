# 準備工作
- 你需要有一個自己的功能變數名稱，會正確的設置 `DNS解析` ，如果不會請自行 GOOGLE
- **請注意配置中後面的備註部分，按要求修改**
# 配置環境
純淨的 Debian 9 系統
# 配置內容
- 安裝基礎工具  
```bash
apt-get update && apt-get -y install socat wget screen
```
- 安裝證書生成腳本  
```bash
wget -qO- get.acme.sh | bash 
source ~/.bashrc
```
- 安裝證書  (**your_domain.com** 改為你的功能變數名稱）
```bash
acme.sh --issue --standalone -d your_domain.com -k ec-256
mkdir /etc/v2ray
acme.sh --installcert -d your_domain.com --fullchain-file /etc/v2ray/v2ray.crt --key-file /etc/v2ray/v2ray.key --ecc
```
- 安裝V2ray 
```bash 
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
bash <(curl -L -s https://install.direct/go.sh)
```
- 安裝密碼套件  （如果中途失去連接可用 **screen -R openssl** 恢復當前窗口，腳本中的選項 **全部填 n**）
```bash
screen -S openssl        
cd /tmp && wget --no-check-certificate https://raw.githubusercontent.com/stylersnico/nginx-openssl-chacha/master/build.sh && sh build.sh
```
- 編輯 v2ray 配置 
```bash
vim /etc/v2ray/config.json
```
- 複製配置  
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
        "path": "/your_path"   #更改路徑
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
- 複製配置  
```bash
server {
    listen 443 ssl http2;                                                       
    ssl_certificate       /etc/v2ray/v2ray.crt;  
    ssl_certificate_key   /etc/v2ray/v2ray.key;
    ssl_protocols         TLSv1.3;                    
    ssl_ciphers           ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4:!DH:!DHE;
    ssl_prefer_server_ciphers on;

    listen 80;
    server_name  your_domain.com;    #改為你的功能變數名稱
    location / {
        proxy_pass https://proxy.com;     #改為你想偽裝的網址
        proxy_redirect     off;
        proxy_connect_timeout      75; 
        proxy_send_timeout         90; 
        proxy_read_timeout         90; 
        proxy_buffer_size          4k; 
        proxy_buffers              4 32k; 
        proxy_busy_buffers_size    64k; 
        proxy_temp_file_write_size 64k; 
     }

    location /your_path {       ##改為你在上面修改的路徑
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
    server_name ip.ip.ip.ip;    #改為你伺服器的 IP 地址
    return 301 https://your_domain.com$request_uri;    #改為你的功能變數名稱
}

server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
  }
```
- 啟動服務  
```bash 
nginx -s reload
service v2ray restart
```
- 開啟 BBR 加速 
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# 客戶端配置

![2](https://github.com/charlieethan/firewall-proxy/blob/master/photos/1.jpg)

**yourdomain**填你的功能變數名稱 ，**id**和**alterId**填你上面設置的  
**Path**填上面設置的路徑 ，其餘部分照抄即可
# 客戶端
Windows系統: [點擊下載](https://github.com/2dust/v2rayN/releases)

Android系統: [點擊下載](https://github.com/2dust/v2rayNG/releases) 
