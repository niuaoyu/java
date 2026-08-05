

开了2台，1台装上了Hindsight给Claude、Codex和Hermes加上记忆，另一台dd之后装了new-api做中转。我自己有一台LA的小鸡，都是用上面的Hermes通过密钥管理Bohrium这两台小鸡。


# 教程

[自建代理节点教程——零基础从头搭建你的专属 VLESS+Reality 节点 - 开发调优 / 开发调优, Lv1 - LINUX DO](https://linux.do/t/topic/1750919)




下载x-ui

在国内访问不了github，尽管开梯子了，但是在terminal看不了，还是要在当前会话加上这两句胡
export http_proxy="http://127.0.0.1:7897"
export https_proxy="http://127.0.0.1:7897"

curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh | sudo -E bash
