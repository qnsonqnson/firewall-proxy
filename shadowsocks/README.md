 # 配置環境
純淨的Debian 9 && 10 系統
# 配置內容
- 升級系統並安裝 Docker
```
apt-get update && apt-get install -y wget vim
wget -qO- get.docker.com | bash
```
- 安裝shadowsocks
```
docker pull teddysun/shadowsocks-libev
mkdir -p /etc/shadowsocks-libev
```
- 寫入配置
```
cat > /etc/shadowsocks-libev/config.json <<EOF
{
    "server":"0.0.0.0",
    "server_port":9000,     #修改端口
    "password":"password0", #修改密碼
    "timeout":300,
    "method":"aes-256-gcm",
    "fast_open":false,
    "nameserver":"8.8.8.8",
    "mode":"tcp_and_udp",
    "plugin":"obfs-server",
    "plugin_opts":"obfs=tls"
}
EOF
```
- 啟動服務 （請將`9000`改為你自己設置的端口）
```
docker run -d -p 9000:9000 -p 9000:9000/udp --name ss-libev --restart=always -v /etc/shadowsocks-libev:/etc/shadowsocks-libev teddysun/shadowsocks-libev
```
- 開啟 BBR 加速：
```
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# 客戶端
- 安卓系統 ： [Shadowsocks下載](https://github.com/shadowsocks/shadowsocks-android/releases) | [obfs插件下載](https://github.com/shadowsocks/simple-obfs-android/releases)    
- Windows系統 ：[點擊下載](https://github.com/shadowsocks/shadowsocks-windows/releases) | [obfs插件下載](https://github.com/shadowsocks/simple-obfs/releases)    
Windows系統 配置如下：  

![2.jpg](https://github.com/charlieethan/firewall-proxy/blob/master/photos/2.jpg)
```
obfs-local
obfs=tls;www.github.com
```
- Windows 用戶可以使用我總結的規則來加速 **Onedrive雲盤** 在本地的下載速度   
規則如下 ：https://github.com/charlieethan/firewall-proxy/releases/download/V4.1.10.0/user-rule.txt  
**下載後請直接複製到 Shadowsocks 所在的檔夾並選擇覆蓋，並重啟 Shadowsocks 服務**
