
[(4 条消息) 一个有趣的 Git 练习网站 - 知乎](https://zhuanlan.zhihu.com/p/383960650)
https://liaoxuefeng.com/books/git/


# 最常用的 Git 指令

1. 查看状态 / 历史
	1. 查看当前状态：`git status`
	2. 查看提交历史（简洁）：`git log --oneline`
	3. 查看分支图：`git log --oneline --graph --all`
	4. 查看某文件历史：`git log --oneline -- 文件路径`

2. 分支操作
	1. 查看所有分支：`git branch -a`
	2. 创建并切换分支：`git switch -c 分支名`
	3. 切换分支：`git switch 分支名`
	4. 删除本地分支：`git branch -d 分支名`
	5. 删除远程分支：`git push origin --delete 分支名`
	6. 重命名分支：`git branch -m 旧名 新名`

3. 提交（工作区 → 暂存区 → 仓库）
	1. 添加所有修改：`git add .`
	2. 添加指定文件：`git add 文件路径`
	3. 提交：`git commit -m "说明"`
	4. 跳过暂存直接提交已追踪文件：`git commit -am "说明"`
	5. 修改最近一次提交：`git commit --amend --no-edit`

4. 撤销 / 回退
	1. 撤销 add（保留工作区）：`git restore --staged .`
	2. 丢弃工作区修改（⚠️ 会丢失）：`git restore .`
	3. 回退到某 commit（保留修改）：`git reset --soft 哈希`
	4. 回退到某 commit（清暂存区）：`git reset --mixed 哈希`
	5. 回退到某 commit（⚠️ 全清空）：`git reset --hard 哈希`
	6. 撤销某次提交（安全，生成反向 commit）：`git revert 哈希`

5. 远程操作
	1. 查看远程地址：`git remote -v`
	2. 修改远程地址：`git remote set-url origin 新地址`
	3. 拉取远程最新（仅下载）：`git fetch origin`
	4. 拉取并合并：`git pull origin main`
	5. 拉取并变基（历史线性）：`git pull --rebase origin main`
	6. 推送到远程：`git push origin main`
	7. 首次推送并绑定分支：`git push -u origin main`
	8. 强制推送（⚠️ 覆盖远程）：`git push --force-with-lease origin main`

6. 合并 / 变基
	1. 合并分支到当前分支：`git merge 分支名`
	2. 合并并保留分支历史：`git merge --no-ff 分支名 -m "说明"`
	3. 变基到 main：`git rebase origin/main`
	4. 解决冲突后继续：`git add . && git rebase --continue`
	5. 放弃合并：`git merge --abort`

7. Tag（版本标记）
	1. 查看所有 tag：`git tag -l`
	2. 打轻量 tag：`git tag v1.0.0`
	3. 打带注释 tag：`git tag -a v1.0.0 -m "说明"`
	4. 推送单个 tag：`git push origin v1.0.0`
	5. 推送所有 tag：`git push origin --tags`
	6. 删除本地 tag：`git tag -d v1.0.0`
	7. 删除远程 tag：`git push origin --delete v1.0.0`

8. 查看差异
	1. 查看工作区改动：`git diff`
	2. 查看暂存区改动：`git diff --staged`
	3. 对比两个 commit：`git diff 哈希1 哈希2`
	4. 只看改动的文件列表：`git diff --name-only 哈希1 哈希2`
	5. 看统计信息：`git diff --stat 哈希1 哈希2`

9. 忽略文件（.gitignore）
	1. 任意路径下的文件：`文件名`
	2. 任意路径下的文件夹：`文件夹名/`
	3. 只忽略根目录：`/文件名`
	4. 例外（不忽略）：`!文件名`
	5. 检查某文件是否被忽略：`git check-ignore -v 文件路径`
	6. 查看所有被忽略文件：`git status --ignored`
	7. 停止追踪已提交文件：`git rm --cached 文件路径`

10. 查看追踪情况
	1. 查看所有被追踪文件：`git ls-files`
	2. 查看某目录被追踪文件：`git ls-files 目录/`
	3. 查看某 commit 下内容：`git ls-tree -r HEAD 目录/`

11. 克隆 / 初始化
	1. 克隆仓库：`git clone git@github.com:用户名/仓库.git`
	2. 克隆到当前空目录：`git clone git@github.com:用户名/仓库.git .`
	3. 初始化本地仓库：`git init`

12. 配置
	1. 查看当前邮箱：`git config user.email`
	2. 设置全局用户名：`git config --global user.name "名字"`
	3. 设置全局邮箱：`git config --global user.email "邮箱"`
	4. 显示中文路径：`git config --global core.quotepath false`
	5. 换行符处理（Windows）：`git config --global core.autocrlf true`





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