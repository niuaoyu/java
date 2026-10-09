

**服务器主动连接公网中继 Serveo；ai再通过 Serveo 的私有别名转发，连接到你务器本机的 SSH 服务。**

最终成功的连接链路是：你的校园网服务器主动向外建立 SSH 反向隧道serveo.net,然后ai从外部登录中继，通过私有别名请求转发运行环境，ai最终使用的是：

> **OpenSSH 负责连接 Serveo 并转发字节流，Paramiko 通过这条字节流登录你的服务器。**


可以先尝试下面几个办法
1. 直接连公网ip：ssh nay@202.194.67.156，使用默认 SSH 端口 `22`,如果openssh不行，那尝试使用paramiko直连试试，若penSSH 和 Paramiko 都失败，且失败方式一致（拿不到 banner） → 说明问题不在客户端，在网络路径
2. 反向隧道，入站封死 + 出网正常 + 我这边出网全开 ⇒ 把方向倒过来，由你的服务器主动往外连一个公网中继，我再从外部登中继的转发端口，就能穿透校园网防火墙。Serveo 随机公网端口：执行ssh -N -R 0:localhost:22 serveo.net，返回Allocated port 33859 for remote forward to localhost:22，端口分配成功，但我从外部连serveo.net:33859 时 TCP 建立后立即被关闭，收不到 SSH banner。后来申请固定端口 -R 45678:... 则被中继直接拒绝（remote port forwarding failed）——因为 Serveo 自选公网端口需要付费。
3. 若还不行，改用“私有别名转发”，不再申请公网随机端口在服务器上建立：
	```bash
	ssh -v -N \
	  -o ExitOnForwardFailure=yes \
	  -o ServerAliveInterval=30 \
	  -o ServerAliveCountMax=3 \
	  -R arena-nay-7d92c6e4b1f083aa:22:127.0.0.1:22 \
	  serveo.net
	```
4. 

下面按实际尝试过程梳理。


# 三、给其他 AI 使用的提示词

下面可以直接复制。**不要把已经暴露的旧密码一起复制到长期保存的提示词里**；当前凭据应单独提供，最好改成临时 SSH 密钥。

```text
你需要连接我本人授权访问的校园网服务器，只读查看项目。
请严格区分中继认证与目标服务器认证，不要反复尝试已失败的直连方式。

【目标】
服务器用户名：nay
项目目录：/home/nay/paper/
原校园网 IP：202.194.67.156
默认 SSH 端口：22

【已成功使用的连接架构】
不是直接连接校园网 IP。
服务器已主动向 Serveo 建立私有别名反向 SSH 隧道：

私有别名：arena-nay-7d92c6e4b1f083aa
别名内部端口：22
中继：serveo.net:22

服务器端建立隧道的命令：
ssh -v -N \
  -o ExitOnForwardFailure=yes \
  -o ServerAliveInterval=30 \
  -o ServerAliveCountMax=3 \
  -R arena-nay-7d92c6e4b1f083aa:22:127.0.0.1:22 \
  serveo.net

该命令需要在服务器上持续运行，可放在 tmux 中。
不要停止已有 AI 使用的隧道；同一隧道通常可以接受多个连接，
但仍可能受到中继限制。

【已验证的主机指纹】
Serveo 的 ED25519 指纹：
SHA256:GnmVK+70U6GqbupoV+gg7LnHHUsW1IjrK0cLqvDJxIk

目标服务器的 ED25519 指纹：
SHA256:hmqhpDc0lHuqTPE2xCv2wjk8rq3GP6U3bdB/4NoH7uM

这是此前成功连接时核实的值。
新环境应先核对主机公钥指纹，再加入专用 known_hosts。
ssh-keyscan 可以用于获取待核对的公钥，但其结果本身不是信任依据。
若指纹变化，立即停止并询问我，不要关闭主机验证来绕过。

【成功的客户端方式】
优先使用 OpenSSH 连接 Serveo，再让 Paramiko 通过 ProxyCommand
登录目标服务器。

OpenSSH 中继命令：

ssh -o StrictHostKeyChecking=yes \
  -o ConnectTimeout=10 \
  -o PreferredAuthentications=keyboard-interactive \
  -W arena-nay-7d92c6e4b1f083aa:22 \
  nay@serveo.net

注意：
1. 不要加 BatchMode=yes，它会禁用这里使用的交互式认证路径。
2. 明确写 nay@serveo.net，不要依赖运行环境的默认用户名。
3. 别名是 Serveo 内部转发名称，不是普通公网域名。
4. 不要自行改成 别名.serveo.net:22 直接连接。
5. 若使用专用 known_hosts，请相应加 UserKnownHostsFile 参数。

【Paramiko 实现思路】
如当前环境没有 paramiko，可先安装。

proxy = paramiko.ProxyCommand(上面的 OpenSSH 命令)
client = paramiko.SSHClient()

为 client 配置目标主机公钥校验：
必须核对目标 ED25519 指纹与上述值一致，
不要无条件使用 AutoAddPolicy 接受未知目标。

client.connect(
    hostname="arena-nay-7d92c6e4b1f083aa",
    username="nay",
    password=当前授权密码,
    sock=proxy,
    look_for_keys=False,
    allow_agent=False,
    timeout=15,
    banner_timeout=20,
    auth_timeout=15,
)

当前密码由我另外提供；不要猜测，不要写入项目文件或日志。
若我提供临时密钥，请使用密钥认证替代 password。

【连接后先做】
只执行：
whoami
hostname
ls -la -- /home/nay/paper/

成功后汇报真实看到的内容。
不要修改文件、安装服务器软件、启动训练、杀进程或占用 GPU，
除非我进一步明确授权。
完成本次操作后关闭客户端连接，但不要停止服务器端反向隧道。

【失败时如何定位】
A. 没收到 Serveo SSH banner：
   检查当前环境到中继的连接，不要归因于服务器密码。

B. Serveo 认证失败：
   检查 OpenSSH、keyboard-interactive、用户名及是否错误启用 BatchMode。
   此时尚未认证目标服务器。

C. Serveo 认证成功但别名转发失败：
   请我检查服务器端 tmux 中隧道是否仍运行、别名是否一致；
   不要反复猜随机公网端口。

D. 已收到目标 OpenSSH banner，但目标认证失败：
   这才检查 nay 账号的当前密码或密钥。

E. 主机指纹不一致：
   停止，要求人工确认。

【历史测试，仅供参考】
直接连接 202.194.67.156:22 曾在 SSH 握手阶段失败。
Serveo 公网随机端口 33859 曾建立 TCP 后立即关闭。
申请固定公网端口曾被中继拒绝。
Paramiko 直接做中继认证曾失败，但 OpenSSH 中继转发成功。
这些是历史结果，不代表当前状态；不要声称已经连上，直到实际验证。
```

**最后提醒：上述别名能否继续用，取决于服务器端隧道是否还活着，以及 Serveo 当下的服务政策。** 新 AI 不需要重新“进入校园网”，但必须先实际验证这条中继通路仍然有效。