
[(4 条消息) 一个有趣的 Git 练习网站 - 知乎](https://zhuanlan.zhihu.com/p/383960650)
https://liaoxuefeng.com/books/git/

## 1. 新环境下用 SSH 连接 GitHub

没开启ssh 要先开 sudo apt install openssh-server
检查是否开启 sudo systemctl status ssh,可以设置开机自启动 sudo syystemctl enable ssh

```bash
# ① 生成 SSH key（已有则跳过）
ssh-keygen -t ed25519 -C "littlespark@gmail.com"

# ② 查看公钥并复制
cat ~/.ssh/id_ed25519.pub

# ③ 粘贴到 GitHub：Settings → SSH and GPG keys → New SSH key

# ④ 改 remote 为 SSH 地址
git remote set-url origin git@github.com:用户名/仓库名.git

# ⑤ 验证
ssh -T git@github.com
# 显示 "Hi 用户名! You've successfully authenticated" 即为成功

# 克隆代码到本地后
# 查看当前 remote
git remote -v

# 如果是 https 开头，改成 ssh
git remote set-url origin git@github.com:你的用户名/仓库名.git

# 拉取远端最新
git pull

# 提交推送
git add .
git commit -m "描述"
git push

# 第一次推送新分支
git push -u origin main
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

# 3.忽略任意层级下的名叫modules文件或目录
1. 仓库根目录的.gitignore 添加 `**/modules/`，这是匹配所有的叫models的目录和文件
2. 如果想要删的是目录不是文件，可以写成`**/modules/`
3. 如果只想删除某个路径下的所有目录，写成，`src/**/modules`
4. 如果想要删除除了某个src目录下的，其他所有的modules，可以写`**/modules`+`!/src/**/modules`
5. 如果当前的modules已经被git追踪了（git add 或者git commit），仅仅加.gitignore是不够的，需要先从git索引移除（移除索引不会影响本地文件）`git rm -r --cached **/modules`，做完后执行 `git add .gitignore`


理解 `main`、`develop`、`feature/*` 分支的区别；理解 **PR/MR 流程**。