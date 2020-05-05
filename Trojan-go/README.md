## Trojan-Go 与原版 Trojan 的区别
- Socks5/HTTP 代理自动适配 
- 多路复用，显著提升并发性能 （例如打开 [Pixiv](https://www.pixiv.net) 等图片站时，**显著提升加载速度**）  
- 自动化HTTPS证书申请，使用ACME协议从Let's Encrypt自动申请和更新HTTPS证书   
- **Trojan-Go 项目地址** ：https://github.com/p4gefau1t/trojan-go
## 前期准备 
- 一台可用的 VPS   
- 一个 **没有被 DNS 污染的域名**    
- **确保使用的域名已经成功解析到你的 VPS服务器，并且 未开启 CDN选项**   
- 纯净的 **Debian 9** 系统 
## 配置内容 
- 1. 升级并安装必要软件   
```bash
sudo -i
apt-get upgrade
apt install -y unzip wget
```
- 2. 拉取安装包   
```bash
wget https://github.com/charlieethan/firewall-proxy/releases/download/V0.4.4/trojan-go.zip
chmod +x trojan-go.zip
unzip trojan-go.zip
```
- 3. 自动申请证书 （**按提示输入你的 域名 和 邮箱**）  
```bash
rm -rf trojan-go.zip
sudo ./trojan-go -autocert request
```
- 4. 写入 Trojan-GO 配置文件     
```bash
vim server.json
```
> 配置a ：不需要使用 **CDN** 进行流量中转 （你的服务器 IP地址 未被墙）  
```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #修改为你设定的密码
    ],
    "ssl": {
        "cert": "/root/server.crt",
        "key": "/root/server.key"
    }
}
```
>> 配置b ：需要使用**CDN** 进行流量中转 （你的服务器 IP地址 已经被墙）  
```bash
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password0"  #修改为你设定的密码
    ],
    "ssl": {
        "cert": "/root/server.crt",
        "key": "/root/server.key"
    },
      "websocket": {
        "enabled": true,
        "path": "/your_path",    #修改为你设定的路径
        "hostname": "your_domain.com",   #修改为你的域名
        "obfuscation_password": "password1",   #修改为另一个密码，切勿与上面的密码相同
        "double_tls": true
    }
}
```
**`:wq!`保存并退出** 

- 5. 安装 Nginx  
```bash
apt update
apt install nginx
```
- 6. 移除默认安全组 （ **your_domain.com 改为你的域名 ，Ctrl+X 保存并退出**）
```bash
rm /etc/nginx/sites-enabled/default
nano /etc/nginx/sites-available/your_domain.com
```
- 7. 配置 Nginx （**your_domain.com 改为你的域名**）   
```bash
ln -s /etc/nginx/sites-available/your_domain.com /etc/nginx/sites-enabled/
vim /etc/nginx/conf.d/about.conf
```
**复制下列配置**  
```bash
server {
    listen 127.0.0.1:80 default_server;
    server_name your_domain.com;   #修改为你的域名
    location / {
        proxy_pass https://your_proxy.com;   #修改为你想伪装的网站域名，例如 https://www.pexels.com  
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
    server_name ip.ip.ip.ip;  #修改为你服务器的 IP地址
    return 301 https://your_domain.com$request_uri;   #修改为你的域名
}

server {
    listen 0.0.0.0:80;
    listen [::]:80;
    server_name _;
    return 301 https://$host$request_uri;
}
```
- 8. 启动服务  
```bash
nohup ./trojan-go -config server.json >trojan-go.log 2<&1 &
nginx -s reload
```
- 9. 开启 BBR 加速 
```bash
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
## 客户端的使用 
PC平台 ：https://github.com/Trojan-Qt5/Trojan-Qt5/releases   
安卓平台 ：https://github.com/charlieethan/firewall-proxy/releases/download/V0.4.4/igniter-go-v0.4.0.apk   
**如果未开启 Websocket ，请关闭相关开关**
