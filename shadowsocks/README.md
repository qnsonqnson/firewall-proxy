# Build Environment
Debian 9 && Ubuntu 16~18
# Config
- update and install Docker
```
apt-get update && apt-get install -y wget vim
wget -qO- get.docker.com | bash
```
- install Shadowsocks
```
docker pull teddysun/shadowsocks-libev
mkdir -p /etc/shadowsocks-libev
```
- Paste config files
```
cat > /etc/shadowsocks-libev/config.json <<EOF
{
    "server":"0.0.0.0",
    "server_port":9000,     # modify port
    "password":"password0", # modify password
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
- Start Service （Please modify "9000" to your set port before）
```
docker run -d -p 9000:9000 -p 9000:9000/udp --name ss-libev --restart=always -v /etc/shadowsocks-libev:/etc/shadowsocks-libev teddysun/shadowsocks-libev
```
- Start BBR Accelerate (A solotion to decrease network delay from Google) ：
```
bash -c 'echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf'
bash -c 'echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf'
sysctl -p
```
# Client
- Android 6.0+： [Shadowsocks Download](https://github.com/shadowsocks/shadowsocks-android/releases) | [Plugin Download](https://github.com/shadowsocks/simple-obfs-android/releases)    
- Windows 7+: [Shadowsocks Download](https://github.com/shadowsocks/shadowsocks-windows/releases)      
[Plugin Download](https://github.com/shadowsocks/simple-obfs/releases)    
The config on Windows:

![2.jpg](https://github.com/charlieethan/firewall-proxy/blob/master/photos/2.jpg)
```
obfs-local
obfs=tls;www.github.com
```
