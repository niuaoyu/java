下面是本次在你的 Alpine VPS 上执行的安装步骤。

## 1. VPS 连接信息

系统：Alpine Linux 3.19.1
SSH 用户：root
SSH 地址：78.159.110.6
密码：nay232408.
SSH 端口：10163
节点可用端口范围：21630–21639
本次使用节点端口：21630

连接命令：

ssh -o StrictHostKeyChecking=accept-new -p 10163 root@78.159.110.6

看到提示后输入你的 VPS 密码：nay232408.

## 2. 检查 Alpine 环境

登录后执行：

cat /etc/alpine-release
uname -m
ip addr
ip route
ss -lntup

这台 VPS 的实际网络是 NAT 网络，内部地址类似：

10.10.14.163

因此：

外部 SSH 10163  -> 内部 SSH 22
外部节点 21630 -> 内部节点 21630

## 3. 下载 Xray

本次安装的是 Xray 26.7.28：

mkdir -p /tmp/xray-check
cd /tmp/xray-check

curl -fL --retry 2 \
-o Xray-linux-64.zip \

https://github.com/XTLS/Xray-core/releases/download/v26.7.28/Xray-linux-64.zip

unzip -oq Xray-linux-64.zip xray
chmod 0755 xray
install -m 0755 xray /usr/local/bin/xray

/usr/local/bin/xray version

正常应显示类似：

Xray 26.7.28


## 4. 检查并备份原配置

原服务器上虽然有配置文件：

/usr/local/etc/xray/config.json

但没有 Xray 程序和启动服务，而且原配置监听的是 40000，不在你开放的 21630–21639
范围内。

备份原配置：

mkdir -p /usr/local/etc/xray
cp -p /usr/local/etc/xray/config.json \
"/usr/local/etc/xray/config.json.bak.$(date +%Y%m%d%H%M%S)"

———

## 5. 生成 UUID、REALITY 密钥和 Short ID

key_output=$(/usr/local/bin/xray x25519)

private_key=$(printf '%s\n' "$key_output" |
awk -F': ' '/^PrivateKey:/{print $2}')

public_key=$(printf '%s\n' "$key_output" |
awk -F': ' '/^Password \(PublicKey\):/{print $2}')

uuid=$(/usr/local/bin/xray uuid | tail -1)

short_id=$(openssl rand -hex 8)

printf 'UUID: %s\n' "$uuid"
printf 'Public Key: %s\n' "$public_key"
printf 'Short ID: %s\n' "$short_id"

其中：

- private_key 只保存在服务器配置中；
- public_key 需要填写到客户端；
- uuid 需要填写到客户端；
- short_id 需要填写到客户端。

———

## 6. 创建 VLESS + REALITY 配置

执行：

cat > /usr/local/etc/xray/config.json <<EOF
{
"log": {
  "loglevel": "warning",
  "access": "/var/log/xray-access.log",
  "error": "/var/log/xray-error.log"
},
"inbounds": [
  {
	"listen": "0.0.0.0",
	"port": 21630,
	"protocol": "vless",
	"settings": {
	  "clients": [
		{
		  "id": "$uuid",
		  "flow": "xtls-rprx-vision"
		}
	  ],
	  "decryption": "none"
	},
	"streamSettings": {
	  "network": "raw",
	  "security": "reality",
	  "realitySettings": {
		"show": false,
		"dest": "www.cloudflare.com:443",
		"xver": 0,
		"serverNames": [
		  "www.cloudflare.com"
		],
		"privateKey": "$private_key",
		"shortIds": [
		  "$short_id"
		]
	  }
	},
	"sniffing": {
	  "enabled": true,
	  "destOverride": [
		"http",
		"tls",
		"quic"
	  ]
	}
  }
],
"outbounds": [
  {
	"protocol": "freedom",
	"tag": "direct"
  },
  {
	"protocol": "blackhole",
	"tag": "blocked"
  }
],
"routing": {
  "rules": [
	{
	  "type": "field",
	  "protocol": [
		"bittorrent"
	  ],
	  "outboundTag": "blocked"
	}
  ]
}
}
EOF

设置权限：

chown xray:xray /usr/local/etc/xray/config.json
chmod 0600 /usr/local/etc/xray/config.json

检查配置：

/usr/local/bin/xray \
-test \
-config /usr/local/etc/xray/config.json

正常结果：

Configuration OK.

———

## 7. 创建 Alpine OpenRC 服务

mkdir -p /var/log

touch /var/log/xray.log
touch /var/log/xray-access.log
touch /var/log/xray-error.log

chown xray:xray \
/var/log/xray.log \
/var/log/xray-access.log \
/var/log/xray-error.log

创建服务文件：

cat > /etc/init.d/xray <<'EOF'
#!/sbin/openrc-run

name="xray"
description="Xray proxy service"

command="/usr/local/bin/xray"
command_args="run -config /usr/local/etc/xray/config.json"
command_user="xray:xray"
command_background="yes"

pidfile="/run/xray.pid"
output_log="/var/log/xray.log"
error_log="/var/log/xray-error.log"

depend() {
need net
after firewall
}
EOF

chmod 0755 /etc/init.d/xray

设置开机启动并启动服务：

rc-update add xray default
rc-service xray start

如果已经启动过，可以使用：

rc-service xray restart

检查状态：

rc-service xray status

正常结果：

status: started

检查监听：

ss -lntup | grep -E '21630|xray'

正常应看到：

*:21630
users:(("xray"...))

———

## 8. 清理原来的无效 watchdog

原来 root 的定时任务中有不存在的 XrayR 和 V2bX 服务检查。删除这两条无效任务：

crontab -l |
sed '/rc-service XrayR status/d; /rc-service V2bX status/d' |
crontab -

验证：

crontab -l | grep -E 'XrayR|V2bX' ||
echo "no stale XrayR/V2bX watchdog entries"

———

## 9. 当前节点配置

地址：78.159.110.6
端口：21630
协议：VLESS
UUID：4a0697a2-544d-46b1-aaba-d83729be54c2
Flow：xtls-rprx-vision
加密：none
传输：TCP
安全：REALITY
SNI：www.cloudflare.com
指纹：chrome
Public Key：2A6YmbfFU0epLS5nIHOKkZw9TnZc0UFybkwEr5GdtCI
Short ID：10e88bfc95e0fe4a

完整导入链接：

vless://4a0697a2-544d-46b1-aaba-d83729be54c2@78.159.110.6:21630?encryption=non
e&security=reality&sni=www.cloudflare.com&fp=chrome&pbk=2A6YmbfFU0epLS5nIHOKkZw9
TnZc0UFybkwEr5GdtCI&sid=10e88bfc95e0fe4a&type=tcp&headerType=none#alpine-vless-r
eality

公网测试结果：

21630 OPEN
21631 CLOSED
21632 CLOSED
21633 CLOSED
21634 CLOSED
21635 CLOSED
21636 CLOSED
21637 CLOSED
21638 CLOSED
21639 CLOSED

这里的 21631–21639 关闭是因为没有程序监听，并不影响 21630 节点使用。

最后，请尽快修改 VPS 的 root 密码。该密码已经出现在聊天记录中，不建议继续使
用。
