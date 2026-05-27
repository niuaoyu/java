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
