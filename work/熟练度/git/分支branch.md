
简单的操作：在main分支创建dev分支，然后“**当前分支**” * 指向dev分支，然后在dev分支做任何操作之后，觉得不错后，可以合并到主分支，这时候操作是，先git checkout main分支 然后再git  merge dev 
```bash
git checkout -b dev / git switch -c dev    # 当前HEAD * 转到dev分支
touch dev/dev.md                           # 创建文件，修改

git checkout main  / git switch main       # 切回main分支
git merge dev                              # main 和 dev 分支合并 
git branch -d dev                          # 可以心安理得的删除dev分支了

```
之前说过 git checkout -- files 容易起冲突，所以checkout 换成switch更好，就两点不同 当切到一个新的分支是 `git switch -c dev`，切到是一个已经存在的分支的时候直接`git switch main`

### --no-ff

**Fast-forward（快进合并）** = 当合并分支时，如果目标分支（main）**没有新提交**，Git 直接把 main 的指针移动到 dev 的最新commit，**不创建新的合并commit**。

有个问题main分支合并到dev分支的时候，这时候就看不到那些commit是dev分支做的信息了
````plain
git merge --no-ff -m "merge with no-ff" dev
````
**`--no-ff` 让Git保留分支的"完整性"，虽然多了一个合并commit，但换来了清晰的开发历史**

```
场景：dev 分支领先于 main

Fast-forward 合并前：
main:  A → B
dev:   A → B → C → D

Fast-forward 合并后（默认）：
main:  A → B → C → D  （指针直接移动）
dev:   A → B → C → D

结果：main 和 dev 指向同一个commit，没有合并commit记录！


# 1. 查看当前状态
git log --oneline --graph --all

# * def456g (dev) D commit
# * abc123f C commit
# * ghi789h (main) B commit
# * jkl012i A commit

# 2. 默认合并（Fast-forward）
git checkout main
git merge dev

# 输出： Updating ghi789h..def456g
#       Fast-forward

# 3. 查看历史
git log --oneline --graph

# * def456g (HEAD -> main, dev) D commit
# * abc123f C commit
# * ghi789h B commit
# * jkl012i A commit
# 注意：没有新的合并commit！main 和 dev 指向同一个commit def456g
```

**`--no-ff`= 即使可以快进， 也强制创建一个新的合并commit**，记录合并操作。

```
# 1. 使用 --no-ff 合并
git checkout main
git merge --no-ff dev -m "merge with no-ff"

# 2. 查看历史
git log --oneline --graph
# *   abc123f (HEAD -> main) merge with no-ff
# |\
# | * def456g (dev) D commit
# | * ghi789h (dev) C commit
# |/
# * jkl012i B commit
# * mno345p A commit

# 注意：多了一个合并commit（abc123f），dev 的历史保留在侧支
```
因为也是创建一个commit所以也需要 -m "写上commit的信息"，同时可以看到graph上可以看到dev做的事情！回头回溯的时候也好找得到！
没有--no-ff，无法知道 C 和 D 是在 dev 分支开发的，历史变成了一条直线，分支信息永久丢失







dev分支，修改readme.md一部分，切回main分支的时候main分支也修改readme.md文档，这时候`git merge dev` 就出现合并冲突，这时候重新打开readme.md文档，就会出现有冲突的地方，类似
```md
Git tracks changes of files. 
<<<<<<< main分支修改的内容 
======= 
>>>>>>> dev分支修改的内容
```
直接删除不想要的就行了，比如
```md
Git tracks changes of files. 
dev分支修改的内容
```
重新add commit即可