# HomeBrew 使用笔记（Version 2.0）

> 本机版本：Homebrew 6.0.21（Apple Silicon，根目录 /opt/homebrew）
> Homebrew 是什么：macOS（和 Linux）的包管理器，用一条命令安装 / 卸载 / 升级软件。

---

## 一、常用指令速查表（先记这张表）

| 命令 | 含义 | 示例 |
|------|------|------|
| `brew search 关键词` | 搜索软件 | `brew search qq` |
| `brew install 软件` | 安装命令行工具 | `brew install node` |
| `brew install --cask 软件` | 安装图形界面软件 | `brew install --cask wechat` |
| `brew uninstall 软件` | 卸载（GUI 软件加 `--cask`） | `brew uninstall --cask wechat` |
| `brew reinstall 软件` | 重装修复（替代先卸载再安装） | `brew reinstall node` |
| `brew update` | 更新 Homebrew 自身 + 软件索引 | `brew update` |
| `brew upgrade` | 升级已装软件（不写名字 = 全部） | `brew upgrade` |
| `brew upgrade 软件` | 只升级某一个 | `brew upgrade --cask wechat` |
| `brew list` | 列出已装软件 | `brew list --formula` |
| `brew leaves` | 只看「手动装的顶层软件」 | `brew leaves` |
| `brew info 软件` | 看软件详情（版本 / 依赖 / 注意事项） | `brew info node` |
| `brew outdated` | 看有哪些可以升级 | `brew outdated` |
| `brew cleanup` | 清理旧版本和下载缓存 | `brew cleanup` |
| `brew doctor` | 体检，排查环境问题 | `brew doctor` |
| `brew services 动作 服务` | 管理后台服务（启动 / 停止 / 自启） | `brew services start mysql` |
| `brew pin / unpin 软件` | 锁定 / 解锁版本（防止误升级） | `brew pin node` |
| `brew bundle dump / install` | 导出 / 还原整个环境 | `brew bundle dump` |

---

## 二、详细解释

### 1. 命令结构

```
brew   指令   对象
```

- **brew**：告诉终端要用 Homebrew 这款软件
- **指令**：要做什么（install / search / upgrade …）
- **对象**：对哪个软件操作

### 2. 两个最核心、最容易混的概念

#### （1）update vs upgrade ⭐

| 命令 | 作用 | 记忆 |
|------|------|------|
| `brew update` | 更新 Homebrew 自身 + 拉取最新软件索引 | 刷「价目表」 |
| `brew upgrade [软件]` | 把已安装的软件升级到新版本 | 换「新货」 |

`update` 不升级已装的软件，`upgrade` 不更新索引，两者是两回事。
新版 Homebrew 在敲 `install` / `upgrade` 时会自动先 `update`，所以多数时候直接 `brew upgrade` 即可。

#### （2）formula vs cask ⭐

| 类别 | 是什么 | 例子 | 安装写法 |
|------|--------|------|----------|
| Formula（公式） | 命令行工具 / 库 | git、node、python、wget | `brew install node` |
| Cask（木桶） | 图形界面 GUI 应用 | 微信、Chrome、VS Code | `brew install --cask wechat` |

装 GUI 软件务必带 `--cask`（旧版 `brew cask install` 已废弃）。

### 3. 房子结构：几个词搞懂东西装到哪了

- **prefix（根目录）**：Homebrew 的大本营，本机是 `/opt/homebrew`（Intel 机器是 `/usr/local`）。用 `brew --prefix` 查看。
- **Cellar（酒窖）**：每个 formula 的存放地，路径 `/opt/homebrew/Cellar/软件名/版本号/`。
- **Caskroom**：每个 cask 的存放地，路径 `/opt/homebrew/Caskroom/软件名/版本号/`。
- **link（软链接）**：把软件的可执行文件链到 `/opt/homebrew/bin`，终端才能直接敲命令。
- **Tap（仓库）**：软件索引从哪来。默认官方 `homebrew/core` 和 `homebrew/cask`；第三方用 `brew tap 用户名/仓库` 添加。
- **Bottle（瓶子）**：预编译好的二进制包，装它不用从源码编译，快。

### 4. 每个命令的详细说明

#### install / uninstall / reinstall
- `brew install git wget node` 一次装多个
- 卸载 GUI 软件要带 `--cask`：`brew uninstall --cask wechat`
- `brew reinstall 软件` 重装修复：软件坏了/配置乱了，比「先 uninstall 再 install」一步到位

#### search
- `brew search qq`（formula 和 cask 都搜）
- `brew search --cask qq`（只在 GUI 软件里搜）

#### info（装之前 / 装之后都先看它）
- `brew info node`：版本、是否已装、依赖、**caveats（注意事项）**
- 装完任何软件后，看 info 最后的 caveats，往往就是「下一步该做什么」

#### upgrade / outdated
- `brew outdated` 先看有哪些能升级
- `brew upgrade` 全部升；`brew upgrade --cask wechat` 升单个
- `brew upgrade -n` 只预览会升什么、不真升（安全，建议先试这个）
- 想升「自更新」软件（Chrome、微信这类），用 `brew upgrade --greedy`

#### list / leaves
- `brew list` 列出所有（含依赖，很乱）
- `brew leaves` 只看自己手动装的顶层软件（干净）
- 分类查看：`brew list --formula` / `brew list --cask`

#### cleanup / autoremove
- `brew cleanup` 清旧版本和缓存（新版 upgrade 后会自动跑，一般不用手动）
- `brew autoremove` 清掉不再被依赖的孤儿包

#### services（后台服务）
- `brew services start mysql`：立即启动 **并** 设为开机自启
- `brew services run mysql`：立即启动，**不** 设自启
- `brew services stop mysql`：停止并取消自启
- `brew services list`：看所有服务状态

#### pin / unpin
- `brew pin node` 锁定版本，`brew upgrade` 会跳过它；`brew unpin node` 解锁

#### doctor
- 环境体检，出问题时第一件事跑它

### 5. 换机还原（Brewfile）

```bash
# 旧机：导出清单（Brewfile 生成在当前目录）
cd ~/Desktop && brew bundle dump

# 新机：按清单还原（在 Brewfile 所在目录执行）
cd ~/Desktop && brew bundle install
```

- `dump` 会连同 tap（第三方仓库）一起记录，第三方源也能还原。
- 指定文件名：`brew bundle dump --file=/路径/文件名`

### 6. 提速技巧

- 跳过本次自动更新：`HOMEBREW_NO_AUTO_UPDATE=1 brew install node`
- 国内可换清华 / 中科大镜像源（替换 HOME 源和 bottles 源）

---

## 三、进阶命令（按需了解）

### config —— 一次性看全环境
`brew config` 输出 Homebrew 全部配置：版本、安装路径、CPU、系统版本、编译器、各类路径。出问题排查时比 `brew --version` 有用得多。

### deps / uses —— 查依赖关系（简单常用）
- `brew deps node` / `brew deps --tree node`：看 node 依赖了谁（`--tree` 树状更直观）
- `brew uses 软件`：反过来看「谁依赖它」，删包前先确认没有别的软件需要它
- 注意：这是查依赖关系，跟「看打包脚本」不是一回事，很简单。

### cat —— 看软件的打包脚本（进阶，收藏）
`brew cat node` 打印 node 的 formula 源码，用来学别人怎么打包。现在还没到这一步，先知道有这个命令即可。

### edit —— 编辑打包脚本（进阶，暂不需要）
`brew edit node` 在编辑器里打开 formula 修改。只有自己写/改公式时才用，现在不用学。
