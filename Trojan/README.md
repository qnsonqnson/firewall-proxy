# 準備工作
1.你需要擁有一個自己的**功能變數名稱**，並會正確設置**DNS解析**，如果你不知道以上兩個名詞的含義，請自行Google學習     
2.搭建環境 ：純淨的 Debian 9 && Ubuntu 16~18 系統

# 開始部署
- 下載證書申請腳本
```bash
apt-get update && apt-get -y install socat         
wget -qO- get.acme.sh | bash       
source ~/.bashrc
```
- 安裝腳本 （注意：**yourdomain.com**請替換為你自己的功能變數名稱）
```bash
acme.sh --issue --standalone -d yourdomain.com -k ec-256
mkdir /etc/trojan
acme.sh --installcert -d yourdomain.com --fullchain-file /etc/trojan/trojan.crt --key-file /etc/trojan/trojan.key --ecc
```
- 安裝Nginx
```bash
apt update
apt install nginx
```
- 移除默認代理組 （注意：**yourdomain.com**請替換為你自己的功能變數名稱,執行後按**Ctrl+X**退出）
```bash
rm /etc/nginx/sites-enabled/default
nano /etc/nginx/sites-available/yourdomain.com
```
- 添加新的代理組並編輯配置檔 （注意：**yourdomain.com**請替換為你自己的功能變數名稱）
```bash
ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
vim /etc/nginx/conf.d/about.conf
```
- 將以下內容粘貼   
```bash
server {
    listen 127.0.0.1:80 default_server;
    server_name yourdomain.com;   #改爲你的功能變數名稱
    location / {
        proxy_pass proxy.com;   #改爲你想僞裝的網站
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
    server_name ip.ip.ip.ip;    #改爲你伺服器的IP
    return 301 https://yourdomain.com$request_uri;   #改爲你的功能變數名稱
}

server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
```
- 安裝Trojan並編輯配置檔
```bash
wget -qO- get.docker.com | bash
docker pull teddysun/trojan
cd /etc/trojan && vim config.json
```
- 將以下內容粘貼 
```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"   #改爲你的密碼
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
- 啟動BBR加速
```bash
sudo bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
sudo bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sudo sysctl -p
```
- 啟動服務
```bash
nginx -s reload
docker run -d --name trojan --restart always --net host -v /etc/trojan:/etc/trojan teddysun/trojan
```

# 客戶端
安卓系統 ：[點擊下載](https://github.com/trojan-gfw/igniter/releases)          
> 配置如下： **地址**填你的功能變數名稱，**端口**填 443 ，**密碼**填你剛才設置的密碼，其他選項無需更改        

Windows系統 ：[點擊下載](https://github.com/Trojan-Qt5/Trojan-Qt5/releases)   
> 專案地址 & 使用說明 ：https://github.com/TheWanderingCoel/Trojan-Qt5
