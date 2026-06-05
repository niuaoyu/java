## 1. 新环境下用 SSH 连接 GitHub

没开启ssh 要先开 sudo apt install openssh-server
检查是否开启 sudo systemctl status ssh

```bash
# ① 生成 SSH key（已有则跳过）
ssh-keygen -t ed25519 -C "your_email@example.com"

# ② 查看公钥并复制
cat ~/.ssh/id_ed25519.pub

# ③ 粘贴到 GitHub：Settings → SSH and GPG keys → New SSH key

# ④ 改 remote 为 SSH 地址
git remote set-url origin git@github.com:用户名/仓库名.git

# ⑤ 验证
ssh -T git@github.com
# 显示 "Hi 用户名! You've successfully authenticated" 即为成功
```

### GitHub 完整使用教程（SSH + Clash代理）

---

#### 一、为什么用 SSH 而不是 HTTPS？

|对比|HTTPS|SSH|
|---|---|---|
|认证方式|每次输用户名+Token|一次配置，永久免密|
|推送|需要 PAT Token|直接推|
|稳定性|受网络影响大|更稳定|
|推荐度|临时用|长期用 ✅|

---

#### 二、生成 SSH 密钥

**Windows（PowerShell）/ Ubuntu（Terminal）命令一样：**

```bash
# 生成密钥，邮箱换成你的github注册邮箱
ssh-keygen -t ed25519 -C "your_email@example.com"

# 一路回车（不设密码更方便）
# 密钥保存在：
# Windows: C:\Users\你的用户名\.ssh\id_ed25519.pub
# Ubuntu:  ~/.ssh/id_ed25519.pub
```

**查看公钥内容：**

```bash
# Ubuntu
cat ~/.ssh/id_ed25519.pub

# Windows PowerShell
cat $env:USERPROFILE\.ssh\id_ed25519.pub
```

**复制输出内容，粘贴到 GitHub：**

> GitHub → Settings → SSH and GPG keys → New SSH key → 粘贴 → 保存

---

#### 三、配置 SSH config（核心）

这一步决定你用不用代理走 SSH。

**Ubuntu：**

```bash
nano ~/.ssh/config
```

**Windows：**

```
记事本打开 C:\Users\你的用户名\.ssh\config（没有就新建）
```

---

##### 情况A：有 Clash 代理（推荐，国内必备）

Clash 默认本地代理端口是 `7890`。

**Ubuntu 写法：**

```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    ProxyCommand nc -X 5 -x 127.0.0.1:7890 %h %p
```

**Windows 写法：**

```
Host github.com
    HostName github.com
    User git
    IdentityFile C:/Users/你的用户名/.ssh/id_ed25519
    ProxyCommand connect -S 127.0.0.1:7890 %h %p
```

> Windows 需要安装 connect.exe，方法见下方附录。

---

##### 情况B：无代理（直连）

```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
```

---

### 情况C：两种都想要（动态切换）

**Ubuntu：**

```
# 有代理时用这个 Host
Host github-proxy
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    ProxyCommand nc -X 5 -x 127.0.0.1:7890 %h %p

# 无代理时用这个 Host
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
```

有代理时 clone：

```bash
git clone git@github-proxy:用户名/仓库名.git
```

无代理时 clone：

```bash
git clone git@github.com:用户名/仓库名.git
```

---

#### 四、验证 SSH 是否通

```bash
ssh -T git@github.com
# 成功显示：Hi 用户名! You've successfully authenticated...
```

如果失败加 `-v` 看详情：

```bash
ssh -Tv git@github.com
```

---

#### 五、Clone 仓库

```bash
# 任意目录下执行，仓库会克隆到当前文件夹下
git clone git@github.com:你的用户名/仓库名.git

# 进入仓库
cd 仓库名
```

---

#### 六、已有仓库改为 SSH（从 HTTPS 切换）

```bash
# 查看当前 remote
git remote -v

# 如果是 https 开头，改成 ssh
git remote set-url origin git@github.com:你的用户名/仓库名.git

# 确认
git remote -v
```

---

#### 七、日常推送拉取

```bash
# 拉取远端最新
git pull

# 提交推送
git add .
git commit -m "描述"
git push

# 第一次推送新分支
git push -u origin main
```

---

#### 八、附录：Windows 安装 connect.exe

Windows SSH 不支持 `nc` 命令，需要 `connect`：

**方法1：装 Git for Windows（推荐）** 装完后 connect.exe 已内置，config 里改为：

```
ProxyCommand "C:/Program Files/Git/mingw64/bin/connect.exe" -S 127.0.0.1:7890 %h %p
```

**方法2：用 scoop 安装**

```powershell
scoop install connect
```

---

#### 九、常见问题

**Q：push 时提示 Permission denied (publickey)**

```bash
# 检查 ssh-agent 是否加载了密钥
ssh-add ~/.ssh/id_ed25519
ssh -T git@github.com
```

**Q：代理端口不是 7890 怎么办**

打开 Clash 面板查看实际端口，把配置里的 `7890` 替换掉。

**Q：Ubuntu 没有 nc 命令**

```bash
sudo apt install netcat-openbsd
```

**Q：想确认走没走代理**

```bash
ssh -Tv git@github.com 2>&1 | grep "proxy\|connect"
```



## 2. 删除远端文件，保留本地

```bash
# ① 从 Git 追踪移除（--cached 保留本地文件）
git rm -r --cached 文件或文件夹路径

# ② 在 .gitignore 中加一条，防止再次误追
echo "要忽略的路径/" >> .gitignore

# ③ 提交并推送
git add .gitignore
git commit -m "remove xxx from remote tracking"
git push
```

**核心点：** `--cached` 是只删远端不删本地的关键，不加 `--cached` 会把本地文件也删掉。

### 2.1 当前目录/1.python/basic有这个文件夹，现在删除了，但是有一部分已经推到remote，如何rm -r --cache？也可能没推到，如何看远端cache？
先确认远端实际有什么：

```bash
# 查看远端已有的文件列表
git ls-remote origin
# 或者看本地追踪的远端状态
git ls-tree -r origin/main --name-only | grep -i python
```
---
**然后找到正确路径：**
```bash
# 查看git追踪了哪些文件（本地缓存）
git ls-files | grep -i python
```
这个命令会显示 git 实际追踪的路径，用这个路径来执行 rm --cached。

**根据输出结果执行删除：**
```bash
# 把上面查到的实际路径填进去，例如：
git rm -r --cached "1.python/basic"

# 然后提交
git add .
git commit -m "remove 1.python/basic from tracking"
git push
```
---
**如果 git ls-files 没有输出**，说明这个文件夹从未被追踪到远端，不需要做任何操作，直接忽略即可。


# 3.忽略任意层级下的名叫modules文件或目录
1. 仓库根目录的.gitignore 添加 `**/modules/`，这是匹配所有的叫models的目录和文件
2. 如果想要删的是目录不是文件，可以写成`**/modules/`
3. 如果只想删除某个路径下的所有目录，写成，`src/**/modules`
4. 如果想要删除除了某个src目录下的，其他所有的modules，可以写`**/modules`+`!/src/**/modules`
5. 如果当前的modules已经被git追踪了（git add 或者git commit），仅仅加.gitignore是不够的，需要先从git索引移除（移除索引不会影响本地文件）`git rm -r --cached **/modules`，做完后执行 `git add .gitignore`
