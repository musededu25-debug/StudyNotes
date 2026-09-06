# tldr 使用笔记（Version 2.0）

> 本机版本：tlrc v1.13.1（实现 tldr client spec v2.3，Apple Silicon）
> 一句话：tldr 是 `man` 的「速查版」——每条命令只挑最常用的几个例子，一眼扫过就能想起来怎么用，不用翻几千行的完整手册。

---

## 一、常用指令速查表（先记这张表）

| 命令 | 作用 | 示例 |
|------|------|------|
| `tldr 命令` | 看命令的常用用法（默认英文） | `tldr tar` |
| `tldr 命令 子命令` | 看子命令页面 | `tldr git commit` |
| `tldr -u` | 更新本地缓存 | `tldr --update` |
| `tldr -L zh 命令` | 临时用中文看某条命令 | `tldr -L zh tar` |
| `tldr -L en 命令` | 临时用英文看某条命令 | `tldr -L en tar` |
| `tldr -s 关键词` | 搜含关键词的命令页面 | `tldr -s archive` |
| `tldr -l` | 列出当前平台收录的命令 | `tldr -l` |
| `tldr --list-languages` | 看已装的语言包 | `tldr --list-languages` |

---

## 二、详细解释

### 1. tldr 是什么

- **全称**：Too Long; Didn't Read（太长不看）
- **定位**：`man` 的补充。`man` 是几千行的完整参考手册；tldr 每条命令只挑**最常用的 5~10 个实例**，每个实例配一句人话说明，直接复制改占位符就能跑。
- **本体**：本机装的是 `tlrc`（Rust 写的官方 tldr 客户端），命令名是 `tldr`。页面数据来自社区维护的 [tldr-pages](https://github.com/tldr-pages/tldr) 项目。

一句话：忘了某条命令怎么用 → `tldr 命令`，扫一眼就知道。

### 2. 命令结构

```
tldr   [选项]   命令名
```

```bash
tldr tar              # 看 tar 的常用用法
tldr git commit       # 看子命令页面（git 那一大堆命令都有独立页）
tldr -s archive       # 不知道命令名，搜「archive」相关的页面
```

### 3. 英文 / 中文切换 ⭐（重点）

本机**默认是英文**（配置文件里 `languages = ["en"]`）。想临时看中文，加 `-L zh`：

```bash
tldr tar          # 英文（默认）
tldr -L zh tar    # 临时中文
tldr -L en tar    # 临时英文（默认英文时其实不用加）
```

- `-L` 只对**这一次**生效，不会改默认配置，用完即走。
- 本机 `en`、`zh` 两个语言包都已装好（`tldr --list-languages` 可见），所以 `-L zh` 直接用，不用先下载。
- 某条命令的中文页缺失时，tldr 会自动回退英文，不用管。

### 4. 和 man 的区别 ⭐

| 场景 | man | tldr |
|------|-----|------|
| 查冷门参数 / 完整细节 | ✅ 全都有 | ❌ 刻意不列全 |
| 忘了常用用法，想快速回忆 | ❌ 要翻几千行 | ✅ 一眼扫完 |
| 给可复制的示例 | ❌ 得自己拼命令 | ✅ 复制改占位符就跑 |
| 中文 | ❌ 基本英文 | ✅ 可临时切中文 |

**结论**：tldr 不是 man 的替代，是「速查」。速查用 tldr，深挖回 `man` 或 `命令 --help`。

### 5. 常用选项分组

#### 查页面
- `tldr 命令`：看常用用法
- `tldr 命令 子命令`：看子命令（如 `tldr git commit`）
- `tldr -s 关键词`：搜含关键词的页面
- `tldr -l`：列出当前平台收录的命令；`tldr -a` 列出全部

#### 语言 / 平台
- `-L zh` / `-L en`：临时切换语言
- `-p linux` / `-p osx`：临时指定平台（默认按本机 macOS）

#### 缓存
- `tldr -u`（`--update`）：更新本地缓存，下载最新页面（平时会自动更新）
- `tldr -o`：离线模式，不联网更新
- `tldr -i`：看缓存信息（路径、已装语言、页数）

#### 显示样式
- `--long-options`：长选项优先（显示 `--all` 这种）
- `--short-options`：短选项优先（显示 `-a` 这种）
- `tldr -R 命令`：输出原始 markdown（想复制原文用）

---

## 三、进阶（按需了解）

### 配置文件与缓存位置

- 配置：`~/Library/Application Support/tlrc/config.toml`
- 缓存页面：`~/Library/Caches/tlrc/`（断网也能用，靠它离线）

```bash
tldr --config-path     # 打印配置路径（目录不存在会自动建）
tldr --gen-config      # 打印默认配置内容
```

### 改默认语言（一般不推荐）

如果想让 tldr **默认**显示中文，编辑 config.toml，把：

```toml
languages = ["en"]
```

改成：

```toml
languages = ["zh"]
```

然后 `tldr -u` 下载中文页面。之后默认中文，想临时看英文就用 `tldr -L en 命令`。

> 本机之前试过默认中文，但 `zh` 语言包下载不稳定，所以又切回了英文（`languages = ["en"]`）。**建议保持「默认英文 + 临时 `-L zh`」**，最省心。

### 其他小技巧

- `tldr --edit 命令`：在浏览器打开该页面源码，方便给社区提交修改。
- `tldr -c 命令`：紧凑输出，去掉空行。

---

## 总结：要不要用

- **必用**：作为 `man` 的日常速查，忘了命令怎么用先 `tldr 命令`，比翻 man 快太多。
- **中文**：默认英文够用；偶尔想看中文就 `tldr -L zh 命令`，零成本。
- **边界**：查冷门参数、完整细节时还是要回 `man` / `--help`，tldr 只覆盖常用场景。
