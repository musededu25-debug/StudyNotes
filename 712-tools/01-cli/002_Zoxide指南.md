# ZoXide 使用笔记（Version 2.0）

> 本机版本：zoxide 0.10.0 / fzf 0.74.3 / yazi 26.8.15（Apple Silicon）
> 一句话：zoxide 是「智能 cd」，fzf 是「通用模糊搜索器」，yazi 是「终端文件管理器」。三者共享一个目录记忆库，配合起来能大幅减少切换目录、找文件时的打字量。

---

## 一、常用指令速查表（先记这张表）

### ① zoxide（终端里的 `z`）

| 命令 | 作用 | 示例 |
|------|------|------|
| `z 关键词` | 跳转到历史目录 | `z cake` |
| `z 词1 词2` | 多关键词精确匹配 | `z vue pro` |
| `z -` | 回到上一个目录 | `z -` |
| `z ..` | 上一级 | `z ..` |
| `zi` | 交互式选目录（用 fzf） | `zi` |

### ② fzf（终端里，已配 fd 扫描）

| 按键 | 作用 |
|------|------|
| `Ctrl-R` | 搜历史命令 |
| `Ctrl-T` | 搜文件、把路径贴到命令行 |
| `Alt-C` | 搜目录并 cd 过去 |

### ③ yazi（文件管理器里）

| 按键 | 作用 |
|------|------|
| `Z`（大写） | zoxide 历史目录跳转 |
| `z`（小写） | fzf 搜当前目录树 |

---

## 二、详细解释

### 1. 三个工具各自是什么

| 工具 | 定位 | 解决什么问题 |
|------|------|-------------|
| **zoxide** | 智能 cd | 目录切换：`z 关键词` 跳到你常去的地方，不用打全路径 |
| **fzf** | 通用模糊搜索器 | 从一堆东西里快速挑出目标：历史命令、文件、目录、git 分支…… |
| **yazi** | 终端文件管理器 | 在终端里浏览/管理文件，像图形界面的访达，但全程键盘 |

**三者关系**：zoxide 和 yazi 都要用 fzf 来做「交互式选择」的前端；而 zoxide 和 yazi **共用同一份目录记忆库**（zoxide 的数据库）。

### 2. zoxide —— 智能 cd

**核心原理**：它悄悄记录你访问过的每个目录，按「频率 + 新鲜度」（frecency）打分。你敲 `z 关键词`，它在数据库里模糊匹配，跳去得分最高的那个。

```bash
z cake      # 跳到含 "cake" 的历史目录（如 .../cake-shop）
z mypro     # 跳到 .../myProject
z vue pro   # 同时含 "vue" 和 "pro" 的目录，多关键词缩小范围
z -         # 回上一目录（相当于 cd -）
z ..        # 上一级
```

**交互式兜底**：记不清目录名时用 `zi`，弹出 fzf 列表上下选，回车跳转。

**本机状态**：已装已激活（`~/.zshrc` 里有 `eval "$(zoxide init zsh)"`），数据库 40 条记录，位置 `~/.local/share/zoxide/db.zo`，由 zoxide 自动维护，**不要手动改**。

### 3. fzf —— 通用模糊搜索器

**核心原理**：你给它一份列表（文件、目录、命令、commit……），它弹一个交互框，输几个字母实时过滤，回车输出选中项。

三大全局快捷键：

- `Ctrl-R`：搜历史命令（替代一次次按上箭头）
- `Ctrl-T`：搜当前目录文件，把路径贴到命令行
- `Alt-C`：搜目录并 cd 过去

**本机状态**：fzf 已装（0.74.3），shell 集成已配好（`Ctrl-R / Ctrl-T / Alt-C` 全部可用），底层用 fd 扫描——搜隐藏文件、排除 `.git` / `node_modules`。具体配置见「三、进阶」。

### 4. yazi —— 终端文件管理器

**核心用法**：终端敲 `yazi` 进入，方向键（或 `j`/`k`）移动，回车进入目录/打开文件，`q` 退出，`~` 回主目录。

**内置 zoxide 和 fzf**：新版 Yazi（26.x）已经内置了这两个工具，**零配置**：

- 按 `Z`（大写）= zoxide 跳转：弹出 fzf，浏览 zoxide 历史目录，选一个跳过去
- 按 `z`（小写）= fzf 搜索：搜**当前目录树**里的文件/目录

> 注意大小写：`Z` 是历史目录（zoxide），`z` 是当前目录树（fzf）。

**本机状态**：已装（26.8.15），`~/.config/yazi/` 还是空的（没做过任何配置），直接开箱即用。

### 5. 三者配合的案例 ⭐（重点）

#### 案例一：z 为主，zi 兜底

记得目录名 → `z cake` 直接跳；记不清 → `zi` 弹出列表挑。

#### 案例二：z 和 yazi 共享一份记忆库

这是最值钱的一点——终端里的 `z` 和 yazi 里的 `Z`，用的是**同一份 zoxide 数据库**：

- 你在终端 `z mypro` 常去 Vue 项目 → 在 yazi 按 `Z` 输 `mypro` 也能跳到
- 一处积累，两处受益，目录记得越用越准

#### 案例三：yazi 里按小写 `z` 搜文件

在 yazi 里 `z`（小写）不是跳历史目录，而是 fzf 搜当前目录树，快速定位一个文件/子目录。

#### 案例四：完整工作流（进阶版见「三」）

```
终端: z project        # 先跳到项目目录
      yazi             # 打开文件管理器浏览/找文件
      [在 yazi 里] Z   # 想换目录就用 zoxide 跳
      [在 yazi 里] z   # 想找文件就用 fzf 搜
      q                # 退出回到终端
```

---

## 三、进阶（按需了解）

### fzf 的 shell 集成（Ctrl-R / Ctrl-T / Alt-C）

本机已配好，`.zshrc` 里加了两样东西：

**① 启用 shell 集成**（激活那三个快捷键）：

```bash
eval "$(fzf --zsh)"
```

**② 让 fzf 用 fd 扫描**（更快、搜隐藏文件、排除垃圾目录）：

```bash
export FZF_DEFAULT_COMMAND='fd --type f --strip-cwd-prefix --hidden --exclude .git --exclude node_modules'
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"
export FZF_ALT_C_COMMAND='fd --type d --strip-cwd-prefix --hidden --exclude .git --exclude node_modules'
```

- `FZF_DEFAULT_COMMAND`：直接跑 `fzf` 或 `**` 补全时用的列表
- `FZF_CTRL_T_COMMAND`：`Ctrl-T` 找文件
- `FZF_ALT_C_COMMAND`：`Alt-C` 找目录
- fd 参数：`--type f` / `--type d`（找文件/目录）、`--hidden`（含隐藏文件）、`--exclude`（排除 `.git` / `node_modules`）、`--strip-cwd-prefix`（去掉 `./` 前缀）

改完 `source ~/.zshrc` 或重开终端生效。`Ctrl-R` 搜历史命令特别香。

### yazi 退出自动 cd（让 yazi 和终端无缝衔接）

默认退出 yazi 后，终端还停在原目录。配一个 `y` 函数，退出 yazi 时自动 cd 到 yazi 里最后所在目录：

```bash
# 加到 ~/.zshrc
function y() {
  local tmp="$(mktemp -t "yazi-cwd.XXXXXX")"
  yazi "$@" --cwd-file="$tmp"
  if cwd="$(cat -- "$tmp")" && [ -n "$cwd" ] && [ "$cwd" != "$PWD" ]; then
    cd -- "$cwd"
  fi
  rm -f -- "$tmp"
}
```

之后用 `y` 代替 `yazi`，退出时自动「落」在你最后浏览的目录，和 `z` 配合极其顺滑。

### 让 yazi 也「喂」数据库（可选，默认不需要）

默认 yazi 只读 zoxide 数据库，不往里写。想让 yazi 里 cd 逛过的目录也进数据库，创建 `~/.config/yazi/init.lua`：

```lua
require("zoxide"):setup {
    update_db = true,
}
```

### 让 cd 也变成 zoxide（熟练后可选）

```bash
eval "$(zoxide init zsh --cmd cd)"
```

这样 `cd`、`cdi` 直接就是 zoxide 了。建议先用 `z` 养成习惯再切换。

---

## 总结：要不要用

- **zoxide**：必用。切换目录少打 80% 的路径，几乎零学习成本。
- **fzf**：和 zoxide 搭配用（`zi`）就用上了；再配 shell 集成还能白捡 `Ctrl-R` 历史命令搜索。
- **yazi**：想要「终端里的访达」就用，内置 zoxide/fzf，和终端无缝互补。

---

## 📌 附：2026.9.3 今日改动 + 踩坑记录（临时，下次深入学习后合并进正文）

> 这块是今天现场操作的记录，还没整理成系统笔记。下次深入学习后，把要点吸收进正文再删掉这里。

### 今天在机器上真正改的两处（都已生效）

**① `.zshrc` 加了 `y` 函数**（退出自动 cd）——对应正文「三、进阶 · yazi 退出自动 cd」，现已真正配好：

```bash
#2026.9.3 yazi 配置：退出时自动 cd
function y() {
  local tmp="$(mktemp -t "yazi-cwd.XXXXXX")"
  yazi "$@" --cwd-file="$tmp"
  if cwd="$(cat -- "$tmp")" && [ -n "$cwd" ] && [ "$cwd" != "$PWD" ]; then
    cd -- "$cwd"
  fi
  rm -f -- "$tmp"
}
```

**② 新建 `~/.config/yazi/init.lua`**（让 yazi 也喂数据库）——对应正文「三、进阶 · 让 yazi 也喂数据库」，也已建好：

```lua
-- 2026.9.3 让 yazi 也「喂」zoxide 数据库
require("zoxide"):setup {
    update_db = true,
}
```

> 关键：这两样**不在一个地方**。`y` 函数是 shell 配置 → 放 `.zshrc`；`update_db` 是 yazi 自己的配置 → 放 `~/.config/yazi/init.lua`（Lua 语法，zsh 不会去执行它）。

### 今天踩的坑（务必记住）

**「在 yazi 里跳目录，按 q 却回到原点」→ 因为用了 `yazi`，而不是 `y`。**

- 魔法全在 `y` 这个包装函数里：它给 yazi 塞了 `--cwd-file`，退出后读那个临时文件、再 `cd` 过去。
- 直接敲 `yazi` 没有 `--cwd-file`，退出时什么都没写，终端自然停在原点。
- `source ~/.zshrc` 只是「加载」`y` 函数，**不会拦截 `yazi`**。
- 自测：`type y` 应打印出函数体；确认后再用**小写 `y`** 打开才有效。

### 下次深入学习的方向（TODO）

- [ ] `--cwd-file` 完整原理：yazi 退出时把最后所在目录写进临时文件，`y` 读出来 `cd`。
- [ ] `y` 函数逐行拆解：`mktemp -t`（建临时文件）、`cat -- "$tmp"`、`[ -n "$cwd" ] && [ "$cwd" != "$PWD" ]` 这个防呆判断。
- [ ] `update_db = true` 用几天后的体验：`z` 跳转是更准了还是变杂了，决定保留还是改回 `false`。
