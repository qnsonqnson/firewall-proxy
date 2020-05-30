## Trojan-Go 與原版 Trojan 的區別
- 極低的記憶體佔用 （相比原版降低多達 30% ）
- 多路複用，顯著提升併發性能 （例如打開 [Pixiv](https://www.pixiv.net) 等圖片站時，**顯著提升加載速度**）  
- 自動化HTTPS證書申請，使用ACME協議從Let's Encrypt自動申請和更新HTTPS證書   
- **Trojan-Go 專案地址** ：https://github.com/p4gefau1t/trojan-go
## 前期準備 
- 一臺可用的 VPS   
- 一個 **沒有被 DNS 污染的功能變數名稱**    
- **確保使用的功能變數名稱已經成功解析到你的 VPS伺服器，並且 未開啟 CDN選項**   
- 純淨的 **Debian 9** 系統 
## 配置內容 
- 1. 升級並安裝必要軟體   
```bash
sudo -i
apt-get upgrade
apt install -y unzip wget
```
- 2. 拉取安裝包   
```bash
wget https://github.com/charlieethan/firewall-proxy/releases/download/V0.5.1/trojan-go.zip
chmod +x trojan-go.zip
unzip trojan-go.zip
```
- 3. 自動申請證書 （**按提示輸入你的 功能變數名稱 和 郵箱**）  
```bash
rm -rf trojan-go.zip
sudo ./trojan-go -autocert request
```
- 4. 寫入 Trojan-GO 配置檔     
```bash
vim server.json
```
> 配置a ：不需要使用 **CDN** 進行流量中轉 （你的伺服器 IP地址 未被牆）  
```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #修改為你設定的密碼
    ],
    "ssl": {
        "cert": "/root/server.crt",
        "key": "/root/server.key",
	"sni": "your_domain.com",    #修改為你的功能變數名稱
        "fallback_port": 3000 
    }
}
```
>> 配置b ：需要使用**CDN** 進行流量中轉 （你的伺服器 IP地址 已經被牆）  
```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #修改為你設定的密碼
    ],
    "ssl": {
        "cert": "/root/server.crt",
        "key": "/root/server.key",
	"sni": "your_domain.com",    #修改為你的功能變數名稱
        "fallback_port": 3000 
    },
      "websocket": {
        "enabled": true,
        "path": "/your_path",    #修改為你設定的路徑
        "hostname": "your_domain.com",   #修改為你的功能變數名稱
        "obfuscation_password": "password1",   #修改為另一個密碼，切勿與上面的密碼相同
        "double_tls": true
    }
}
```
**`:wq!`保存並退出** 

- 5. 安裝 Nginx  
```bash
apt update
apt install nginx
```
- 6. 移除默認安全組 （ **your_domain.com 改為你的功能變數名稱 ，Ctrl+X 保存並退出**）
```bash
rm /etc/nginx/sites-enabled/default
nano /etc/nginx/sites-available/your_domain.com
```
- 7. 配置 Nginx （**your_domain.com 改為你的功能變數名稱**）   
```bash
ln -s /etc/nginx/sites-available/your_domain.com /etc/nginx/sites-enabled/
vim /etc/nginx/conf.d/about.conf
```
**複製下列配置**  
```bash
server {
    listen 127.0.0.1:80 default_server;
    server_name your_domain.com;   #修改為你的功能變數名稱
    location / {
        proxy_pass https://your_proxy.com;   #修改為你想偽裝的網站功能變數名稱，例如 https://unsplash.com/  
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
    server_name ip.ip.ip.ip;  #修改為你伺服器的 IP地址
    return 301 https://your_domain.com$request_uri;   #修改為你的功能變數名稱
}

server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
server {
	listen 127.0.0.1:3000;
	server_name _;
	return 400;
}
```
- 8. 啟動服務  
```bash
nohup ./trojan-go -config server.json >trojan-go.log 2<&1 &
nginx -s reload
```
- 9. 開啟 BBR 加速 
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
## 客戶端的使用 
PC平臺 ：https://github.com/Trojan-Qt5/Trojan-Qt5/releases   
安卓平臺 ：[點擊下載](https://github.com/charlieethan/firewall-proxy/releases/download/V0.5.1m/Igniter-Go-v0.5.1.apk)			

**移動版推薦配置如下 ：**		
```bash
{
    "run_type": "client",
    "local_addr": "127.0.0.1",
    "local_port": 1080,
    "remote_addr": "your_domain",
    "remote_port": 443,
    "password": [
        "your_password"
    ],
    "ssl": {
        "verify": true,
        "sni": "your_domain",
        "session_ticket": true,
        "reuse_session": true,
        "fingerprint": "auto"
    },
    "mux": {
        "enabled": true,
        "concurrency": 8,
        "idle_timeout": 60
    },
    "websocket": {
        "enabled": false,
        "path": "\/path",
        "double_tls": false,
        "obfuscation_password": ""
    }
}
```		
**注：如果開啟 websocket ，請自行按照伺服器端對本地配置進行修改**
