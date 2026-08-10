

开了2台，1台装上了Hindsight给Claude、Codex和Hermes加上记忆，另一台dd之后装了new-api做中转。我自己有一台LA的小鸡，都是用上面的Hermes通过密钥管理Bohrium这两台小鸡。



# 视频教程

油管上也有不良林的，vless+reality 感觉很稳


# 前置检查


## ip 检测

[IP地址信息和纯净度检测 | IPPure](https://ippure.com/)
[IP Geolocation API | 20B+ Requests Served - ipdata](https://ipdata.co/)
https://ping0.ipyard.com/






### 普通 VPS 的 IP 可能被 GFW 针对 TCP 流量

ping 通了不代表 TCP 没被封。
GFW 可以选择性地对某个 IP 的 TCP 流量进行封锁，而 ICMP（ping）放行。

**验证方法**：在服务器上用 `tcpdump` 抓包，同时在客户端发起连接，看包有没有到达：
在服务器上执行 ：tcpdump -i eth0 port <机场开放的端口> -c 20`

如果客户端发起连接但服务器一个包都没收到，说明 TCP 流量在中国侧就被丢弃了，需要换 IP，而不是折腾配置



# 教程

[自建代理节点教程——零基础从头搭建你的专属 VLESS+Reality 节点 - 开发调优 / 开发调优, Lv1 - LINUX DO](https://linux.do/t/topic/1750919)




下载x-ui

在国内访问不了github，尽管开梯子了，但是在terminal看不了，还是要在当前会话加上这两句胡
export http_proxy="http://127.0.0.1:7897"
export https_proxy="http://127.0.0.1:7897"

curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh | sudo -E bash




# 部署安全

1. 开启白名单，防止被扫到：https://linux.do/t/topic/2575515
2. 分成多ip：3Xui - 路由分流  [用自己的VPS自建的机场，如何实现多IP的分布。 - 开发调优 - LINUX DO](https://linux.do/t/topic/2611701)
3. 

