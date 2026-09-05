# Git 使用笔记（Version 2.0）

> 一句话：Git 是分布式版本管理工具——用「工作区 → 暂存区 → 仓库」三层模型记录每一次改动，分支让你并行开发互不干扰，远程仓库让团队协作同步。

---

## 一、常用指令速查表（先记这张表）

### ① 本地基础

| 命令 | 作用 | 示例 |
|------|------|------|
| `git init` | 初始化仓库（只需一次） | `git init` |
| `git add 文件` / `git add .` | 工作区 → 暂存区 | `git add .` |
| `git commit -m "注释"` | 暂存区 → 仓库（生成一个版本） | `git commit -m "初始化项目"` |
| `git status` | 看当前改动（哪个文件改了、在哪个区） | `git status` |
| `git log` | 看提交历史（版本列表） | `git log --oneline --graph --all` |

### ② 分支

| 命令 | 作用 |
|------|------|
| `git branch` | 查看分支（当前带 `*`） |
| `git branch 名字` | 创建分支 |
| `git checkout 分支` | 切换分支 |
| `git checkout -b 名字` | 创建并切换过去（最常用） |
| `git merge 分支` | 把该分支合并到当前分支 |
| `git branch -d 分支` | 删除分支（有安全检查） |
| `git branch -D 分支` | 强制删除（不检查） |

### ③ 远程

| 命令 | 作用 |
|------|------|
| `git remote add origin <SSH地址>` | 绑定远程仓库 |
| `git remote -v` | 查看绑定（fetch 拉 / push 推两个方向） |
| `git push -u origin main` | 首次推送 + 建立追踪关系 |
| `git push -u origin 分支名` | 推分支，远程没有就自动建同名 |
| `git push` | 日常推送（已建追踪后直接敲） |
| `git pull` | 拉取并合并（工作前必敲） |
| `git fetch` | 只拉取、不合并 |
| `git clone <地址>` | 克隆整个仓库到本地 |

---

## 二、详细解释

### 1. 核心心智模型：三个区 ⭐（一切的基础）

Git 管文件，就是把文件在三个「区」之间搬来搬去：

```
工作区         暂存区          仓库(本地)
Working Dir → Staging  →   Repository
   │   git add     │   git commit   │
```

| 区 | 是什么 | 里面放什么 |
|----|--------|-----------|
| 工作区 | 你电脑上真实的文件 | 你正在改的代码 |
| 暂存区 | 一个"待提交"的暂存篮 | `git add` 进去的文件 |
| 仓库 | 本地版本库 | 一个个 commit（版本） |

**`git status` 输出就是这三区的实时快照**，读懂它 = 掌握 Git：

| status 里的话 | 含义 |
|--------------|------|
| `Changes not staged for commit` | 改了，但**没** `add`（还躺在工作区） |
| `Changes to be committed` | 已经 `add` 了，在暂存区等 `commit` |
| `Untracked files` | 新文件，从没被 `add` 追踪过 |

> 最常读反的地方：`not staged` 是「**还没**进暂存区」，不是"已经提交"。`to be committed` 才是「等着提交」。

### 2. 基础连招（单人日常）

```bash
git init                          # 1 初始化（只需一次）
git add .                         # 2 全部改动进暂存区
git commit -m "初始化项目"         # 3 提交成一个版本
git status                        # 随时查看当前状态
git log --oneline --graph --all   # 看历史（一行一条 + 图形分支）
```

> 注意：`git status`（看当前改动）和 `git log`（看历史版本）是两个命令，别混。

### 3. 分支协作 ⭐（真实团队场景）

**规范**：master（生产，只放能上线的）/ develop（开发，集成大家的功能）/ feature（某个具体功能，如 feature/login）。

**关键纪律**：不要在 develop 上直接写代码——每次新需求都从 develop 拉一条 feature 分支：

```bash
git checkout -b feature/login   # 从当前分支创建并切过去
# ... 写代码 ...
git checkout develop            # 切回 develop
git merge feature/login         # 把登录功能合并进来
git branch -d feature/login     # 合并完删掉临时分支
```

**冲突处理**（merge 时两个人改了同一处）：

```bash
<<<<<<< HEAD
你的代码
=======
别人的代码
>>>>>>> feature/login
```

- 冲突现场**不会**被记录成版本——它只是工作区里的临时状态，Git 历史里全是干净的版本
- 处理：手动删掉 `<<<<<<<` `=======` `>>>>>>>` 标记，留下想要的那份代码，然后：

```bash
git add .
git commit -m "解决合并冲突"
```

### 4. 时光机（reset / reflog）

```bash
git log                       # 看历史，找目标 commitID
git reset --hard <commitID>   # 回退到那个版本
git reflog                    # 连"已经删掉/回退掉"的记录都能找回
```

- `reset --hard` 是**移动当前分支的指针**，不是"切换分支"
- `git reflog` 是后悔药：`log` 里看不到的 commitID，它都留着，复制回来再 `reset --hard` 就回得去

### 5. .gitignore

| 写法 | 作用 |
|------|------|
| `secret.txt` | 忽略单个文件 |
| `*.log` | 忽略一类文件（通配符） |
| `node_modules/` | 忽略整个目录（结尾斜杠） |
| `!error.log` | 例外放行（先忽略 `*.log`，再放行这个） |

### 6. 远程协作

**首次打通（SSH 公钥）**：

```bash
ssh-keygen -t rsa            # 生成公钥（一路回车）
cat ~/.ssh/id_rsa.pub        # 复制公钥，贴到 Gitee/GitHub
ssh -T git@gitee.com         # 验证是否打通
```

**绑定 + 首次推送**：

```bash
git remote add origin <SSH地址>   # 绑定远程，起名 origin
git remote -v                     # 查看（fetch 拉 / push 推两个方向）
git push -u origin main           # 首次推送 + 建立追踪（-u = --set-upstream）
```

**日常协作铁律**：

```bash
git pull            # 开工前先拉（pull = fetch 拉取 + merge 合并）
# ... 写代码、add、commit ...
git push            # 建好追踪后，直接 push 就行
```

- `git push -u origin dev`：本地 dev 推到远程，远程没有就**自动创建同名分支**并绑定
- `git push -f`：强行覆盖远程，危险，除非确认要覆盖别人的提交

---

## 三、进阶 + 避坑（真实事故高发区）

- **`reset --hard` 前的自保**：`--hard` 会连**暂存区**一起重置，所以 `git add` 救不了你未提交的改动。回退前先 `git status` 确认干净；有改动就先 `git commit`（或 `git stash`）再回退。
- **半成品后悔药 `git reset HEAD`**：已经 `git add` 但还没 commit，想退回工作区（保留改动）：`git reset HEAD 文件名`。它只动指针和暂存区，**不动工作区**，和 `--hard` 是两个东西。（新版也可用 `git restore --staged 文件名`，效果一样。）
- **占位符别照抄**：笔记里的 `b1`、`名字`、`feature/login` 都是代号，真机里敲真实分支名。
- **`.gitignore` 终极坑**：忽略规则**管不了已经追踪过的文件**。如果 `password.txt` 之前已经 `git add` 过，写进 `.gitignore` 也照样被追踪推送。要先踢出追踪名单：

```bash
git rm --cached password.txt   # 从追踪名单移除（保留本地文件）
git commit -m "停止追踪 password.txt"
```

- **`|` 不是分隔符**：shell 里 `|` 是管道（把左边输出喂给右边）。要"接着执行"用换行或 `&&`：

```bash
git add . && git commit -m "x"
```

---

## 四、进真实团队前还要补的（待磨练清单）

> 前面的二、三已经覆盖了「个人用 Git」和四个核心坑。这一节是**从"自己会用"到"能进团队协作"之间**还差的技能，按出现频率分三档，方便逐个补。

### ① 立刻要会（进团队第一天就要用）

**1. 全局身份配置 `git config`**

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
git config --global --list        # 查看当前配置
```

不配或配错，commit 的署名就是乱的，团队没法定位"这段是谁写的"。

**2. commit 规范（Conventional Commits）**

团队评审的第一道门槛——commit 信息要一眼看懂：

```
feat: 新增登录功能
fix: 修复登录后跳转错误
docs: 更新 README
refactor: 重构用户模块
test: 补充单元测试
chore: 更新依赖
```

规则：`类型: 用英文动词开头说清做了什么`，一个 commit 只做一件事。

**3. `git diff` 看具体改动**

提交前自查、code review 都靠它：

```bash
git diff              # 工作区 vs 暂存区（还没 add 的）
git diff --staged     # 暂存区 vs 仓库（已经 add 的）
git diff 分支1 分支2  # 两个分支整体差异
```

**4. `git stash` 手头改一半要切分支**

```bash
git stash          # 把未提交改动"藏"起来，工作区变干净
git checkout 别的分支
git stash pop      # 回来再取回
git stash list     # 看藏了几个
```

（这就是前面 `reset --hard` 自保里说的 `git stash` 本体。）

**5. 撤销工作区改动 `git restore`**

```bash
git restore 文件名           # 丢弃某文件改动，回到上次 add/commit 状态
git restore --staged 文件名  # 把 add 的退回工作区（= 前面 reset HEAD）
git restore .                # 丢弃所有工作区改动（危险，慎用）
```

**6. Pull Request / Merge Request + Code Review（团队协作的真正形态）**

个人练 Git 和团队用 Git 最大的区别在这：不是本地 merge 完直接推，而是**推上去走评审**：

1. 本地 push 你的 feature 分支到远程
2. 网页上发起 PR，请求合并进 develop/main
3. 同事 review 代码、提意见 → 你改完再 push
4. 评审通过，由维护者点合并

### ② 很快会碰到（一两周内）

**7. `git rebase` 让历史一条直线**

你开发 feature 期间，develop 被别人推进了。同步有两种：

```bash
git merge develop     # 产生一条合并记录，历史分叉
git rebase develop    # 把你的 commit 挪到最新点之后，历史一条直线
```

团队若要求"历史线性"用 rebase。⚠️ rebase 会**改写 commit**，别对已推送共享的分支用。

**8. `git revert` 安全反做（比 reset --hard 温柔）**

```bash
git revert <commitID>   # 生成一个新 commit，内容 = 撤销那个 commit 的改动
```

和 `reset --hard` 的区别：revert 不改写历史、能安全推到远程，是团队协作里回退的首选。

**9. `git tag` 发版打标签**

```bash
git tag v1.0.0           # 给当前 commit 打标签
git tag                  # 列出所有标签
git push origin v1.0.0   # 推标签到远程
```

发版时靠 tag 定位"这个版本对应哪个 commit"。

**10. `git blame` 查某行代码谁写的**

```bash
git blame 文件名            # 每行显示作者 + commit
git blame -L 10,20 文件名   # 只看第 10~20 行
```

出 bug 定位"这段是谁、为什么这么写"，找人问。

### ③ 进阶了解（遇到再查）

**11. `git cherry-pick` 挑单个 commit**

```bash
git cherry-pick <commitID>
```

feature 里某次修复也适用于 develop，单独挑过去，不用整体 merge。

**12. 全局 .gitignore（机器级，对应 fd 的 ~/.fdignore 思路）**

```bash
# ~/.gitignore_global 写机器相关文件，如 .DS_Store、*.swp
git config --global core.excludesfile ~/.gitignore_global
```

`.DS_Store`（Mac 桌面垃圾）这类"只有你这台机器有"的，放全局合适。

**13. 推送被拒的完整处理**

```bash
git pull --rebase   # 拉取 + 把你的 commit 挪到最新之后
git push            # 再推
```

**14. `git bisect` 二分定位 bug**（高阶）

"某功能以前好好的，现在坏了，但不知道哪次提交坏的"时，用二分法自动定位引入 bug 的那个 commit。遇到再学。

### 自查清单（复习时对着打勾）

- [ ] 能不看笔记说出「三个区」和 status 三句标签的含义
- [ ] 知道 feature 分支工作流：checkout -b → 开发 → merge → 删分支
- [ ] 冲突能手动清理并正确 add + commit
- [ ] 知道 `reset --hard` / `reset HEAD` / `revert` 三者区别
- [ ] 知道 `.gitignore` 管不了已追踪文件，会用 `git rm --cached`
- [ ] 会用 `git stash` 切分支、`git diff` 看改动、`git restore` 撤销
- [ ] commit 信息会按 `类型: 说明` 写
- [ ] 知道 PR/Code Review 的基本流程
- [ ] 能独立完成一次：pull → 开发 → add/commit → push → 发起 PR
