

开了2台，1台装上了Hindsight给Claude、Codex和Hermes加上记忆，另一台dd之后装了new-api做中转。我自己有一台LA的小鸡，都是用上面的Hermes通过密钥管理Bohrium这两台小鸡。

```
#!/bin/sh  
  
# ==========================================================  
# 用户配置区域 (请在此处修改为你的实际信息!!!)  
# ==========================================================  
  
# 你的公网 IP (在 NAT 面板查看)  
PUBLIC_IP="93.113.178.124"  
  
# 请填入你分配到的 4 个端口  
PORT_HY2=21630          # 节点1: Hysteria2 端口 (UDP)  
PORT_TUIC=21631         # 节点2: TUIC 端口 (UDP)  
PORT_REALITY_1=21632    # 节点3: Reality 微软伪装 (TCP)  
PORT_REALITY_2=21633    # 节点4: Reality 苹果伪装 (TCP)  
  
# ==========================================================  
# 下面无需修改  
# ==========================================================  
  
# 1. 基础环境检查与清理  
echo -e "\033[33m[1/6] 正在更新 Alpine 源并安装依赖...\033[0m"  
apk update  
apk add curl openssl ca-certificates  
  
# 清理可能存在的旧服务  
if rc-service sing-box status > /dev/null 2>&1; then  
    rc-service sing-box stop  
    rc-update del sing-box default  
fi  
rm -rf /usr/local/bin/sing-box /etc/sing-box  
  
# 2. 架构判断与下载 Sing-box  
echo -e "\033[33m[2/6] 下载并安装 Sing-box...\033[0m"  
ARCH=$(uname -m)  
case $ARCH in  
    x86_64) SB_ARCH="amd64" ;;  
    aarch64) SB_ARCH="arm64" ;;  
    *) echo "不支持的架构: $ARCH"; exit 1 ;;  
esac  
  
SB_VER="1.8.8"  
DOWNLOAD_URL="https://github.com/SagerNet/sing-box/releases/download/v${SB_VER}/sing-box-${SB_VER}-linux-${SB_ARCH}.tar.gz"  
  
curl -L -o sing-box.tar.gz "$DOWNLOAD_URL"  
tar -zxvf sing-box.tar.gz  
mv sing-box-${SB_VER}-linux-${SB_ARCH}/sing-box /usr/local/bin/  
chmod +x /usr/local/bin/sing-box  
rm -rf sing-box.tar.gz sing-box-${SB_VER}-linux-${SB_ARCH}  
  
# 3. 证书与密钥生成  
echo -e "\033[33m[3/6] 生成证书与密钥...\033[0m"  
mkdir -p /etc/sing-box  
  
# 自签名证书 (给 Hy2/TUIC 用)  
openssl req -x509 -newkey rsa:2048 -nodes -sha256 \  
    -keyout /etc/sing-box/self.key \  
    -out /etc/sing-box/self.crt \  
    -days 3650 \  
    -subj "/CN=bing.com"  
  
# 生成 Reality 必须的参数  
UUID=$(/usr/local/bin/sing-box generate uuid)  
# Key 1  
KEYS_1=$(/usr/local/bin/sing-box generate reality-keypair)  
PK_1=$(echo "$KEYS_1" | grep "PrivateKey" | cut -d: -f2 | tr -d ' "')  
PUB_1=$(echo "$KEYS_1" | grep "PublicKey" | cut -d: -f2 | tr -d ' "')  
SID_1=$(openssl rand -hex 8)  
# Key 2  
KEYS_2=$(/usr/local/bin/sing-box generate reality-keypair)  
PK_2=$(echo "$KEYS_2" | grep "PrivateKey" | cut -d: -f2 | tr -d ' "')  
PUB_2=$(echo "$KEYS_2" | grep "PublicKey" | cut -d: -f2 | tr -d ' "')  
SID_2=$(openssl rand -hex 8)  
  
# 4. 写入配置文件  
echo -e "\033[33m[4/6] 写入 Sing-box 配置文件...\033[0m"  
cat << EOF > /etc/sing-box/config.json  
{  
  "log": { "level": "info", "output": "/var/log/sing-box.log" },  
  "inbounds": [  
    {  
      "type": "hysteria2",  
      "tag": "in-hy2",  
      "listen": "::",  
      "listen_port": $PORT_HY2,  
      "users": [{ "password": "$UUID" }],  
      "tls": { "enabled": true, "certificate_path": "/etc/sing-box/self.crt", "key_path": "/etc/sing-box/self.key" }  
    },  
    {  
      "type": "tuic",  
      "tag": "in-tuic",  
      "listen": "::",  
      "listen_port": $PORT_TUIC,  
      "users": [{ "uuid": "$UUID", "password": "$UUID" }],  
      "congestion_control": "bbr",  
      "tls": { "enabled": true, "certificate_path": "/etc/sing-box/self.crt", "key_path": "/etc/sing-box/self.key" }  
    },  
    {  
      "type": "vless",  
      "tag": "in-reality-ms",  
      "listen": "::",  
      "listen_port": $PORT_REALITY_1,  
      "users": [{ "uuid": "$UUID", "flow": "xtls-rprx-vision" }],  
      "tls": {  
        "enabled": true, "server_name": "www.microsoft.com",  
        "reality": { "enabled": true, "handshake": { "server": "www.microsoft.com", "server_port": 443 }, "private_key": "$PK_1", "short_id": ["$SID_1"] }  
      }  
    },  
    {  
      "type": "vless",  
      "tag": "in-reality-apple",  
      "listen": "::",  
      "listen_port": $PORT_REALITY_2,  
      "users": [{ "uuid": "$UUID", "flow": "xtls-rprx-vision" }],  
      "tls": {  
        "enabled": true, "server_name": "gateway.icloud.com",  
        "reality": { "enabled": true, "handshake": { "server": "gateway.icloud.com", "server_port": 443 }, "private_key": "$PK_2", "short_id": ["$SID_2"] }  
      }  
    }  
  ],  
  "outbounds": [{ "type": "direct" }]  
}  
EOF  
  
# 5. 配置 OpenRC 服务  
echo -e "\033[33m[5/6] 配置 OpenRC 开机自启...\033[0m"  
cat << 'EOR' > /etc/init.d/sing-box  
#!/sbin/openrc-run  
name="sing-box"  
description="Sing-box Proxy Service"  
command="/usr/local/bin/sing-box"  
command_args="run -c /etc/sing-box/config.json"  
command_background=true  
pidfile="/run/sing-box.pid"  
depend() {  
    need net  
    after firewall  
}  
EOR  
chmod +x /etc/init.d/sing-box  
rc-update add sing-box default  
rc-service sing-box restart  
  
# 6. 输出结果  
echo ""  
echo "=========================================================="  
echo "          Alpine Linux 四合一节点安装完成"  
echo "=========================================================="  
echo "提示: NAT机器使用自签名证书，Hy2/Tuic 必须开启【跳过证书验证】"  
echo "=========================================================="  
echo ""  
echo ">>> 节点 1: Hysteria 2 (UDP暴力加速)"  
echo "hysteria2://$UUID@$PUBLIC_IP:$PORT_HY2/?insecure=1&sni=bing.com#Hy2-NAT"  
echo ""  
echo ">>> 节点 2: TUIC v5 (UDP低延迟)"  
echo "tuic://$UUID:$UUID@$PUBLIC_IP:$PORT_TUIC/?congestion_control=bbr&alpn=h3&sni=bing.com&allow_insecure=1#TUIC-NAT"  
echo ""  
echo ">>> 节点 3: VLESS Reality (TCP/Microsoft - 推荐主力)"  
echo "vless://$UUID@$PUBLIC_IP:$PORT_REALITY_1?encryption=none&flow=xtls-rprx-vision&security=reality&sni=www.microsoft.com&fp=chrome&pbk=$PUB_1&sid=$SID_1&type=tcp&headerType=none#Reality-MS"  
echo ""  
echo ">>> 节点 4: VLESS Reality (TCP/Apple - 备用防封)"  
echo "vless://$UUID@$PUBLIC_IP:$PORT_REALITY_2?encryption=none&flow=xtls-rprx-vision&security=reality&sni=gateway.icloud.com&fp=safari&pbk=$PUB_2&sid=$SID_2&type=tcp&headerType=none#Reality-Apple"  
echo ""  
echo "=========================================================="
```