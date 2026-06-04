## 1. 新环境下用 SSH 连接 GitHub

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

---

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

## 2.1 当前目录/1.python/basic有这个文件夹，现在删除了，但是有一部分已经推到remote，如何rm -r --cache？也可能没推到，如何看远端cache？
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

---

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

把 `git ls-files | grep -i python` 的输出发给我，我帮你确认。




# 3.忽略任意层级下的名叫modules文件或目录
1. 仓库根目录的.gitignore 添加 `**/modules/`，这是匹配所有的叫models的目录和文件
2. 如果想要删的是目录不是文件，可以写成`**/modules/`
3. 如果只想删除某个路径下的所有目录，写成，`src/**/modules`
4. 如果想要删除除了某个src目录下的，其他所有的modules，可以写`**/modules`+`!/src/**/modules`
5. 如果当前的modules已经被git追踪了（git add 或者git commit），仅仅加.gitignore是不够的，需要先从git索引移除（移除索引不会影响本地文件）`git rm -r --cached **/modules`，做完后执行 `git add .gitignore`
