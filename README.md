<div align="center">

# 📐 QZeroCodeStyle

### AI 辅助编码时的规范约束 Skill

[![License](https://img.shields.io/badge/license-MIT-yellow.svg)](./LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-orange.svg)](./SKILL.md)
[![Codex](https://img.shields.io/badge/Codex-Skill-blue.svg)](./SKILL.md)

`代码风格规范` · `Git 提交规范` · `注释规范` · `接口设计规范` · `Kotlin/Java 规范`

[规则来源](#规则来源) · [安装](#安装) · [规范概览](#规范概览) · [目录结构](#目录结构)

</div>

---

## 规则来源

本 Skill 中的编码规则来自两个渠道。第一条渠道是 [LeonardNJU/code-humanizer](https://github.com/LeonardNJU/code-humanizer)，我蒸馏了该仓库中的 AI 代码改写规则——这些规则原本用于检测和修正 AI 生成代码中的"过度工程化""无意义包装"等问题，我将其中适合作为编码期约束的部分抽取出来，转化为指导 AI 写代码时的前置规则。通过这种方式，与其让 AI 生成一套充满过度抽象、无意义分层、魔术字符串的代码后再去改写，不如从一开始就用正确的风格约束来引导生成过程。

第二条渠道则来自我个人在使用 AI 辅助编码时的实践经验。在与 Claude Code、Codex 等工具长期协作的过程中，我逐渐积累了一些发现：AI 在注释编写、接口设计、状态表示等方面存在反复出现的低效模式，这些问题不完全是正确性问题，更多的是代码的"可维护性味道"。我将这些经验总结为具体的、可执行的规则条目，纳入本 Skill 统一管理。

需要说明的是，本 Skill 虽然蒸馏了 code-humanizer 的部分理念，但定位与 code-humanizer 并不相同。code-humanizer 是一个后处理工具，对已生成的代码进行改写；而本 Skill 是一个前置约束，在 AI 生码之前就将规范注入上下文。两者各有所长，可以互补使用。

---

## 安装

### 方法一：Git 克隆 + 软链接（推荐）

```bash
# 1. 克隆本仓库
git clone git@github.com:QZero233/MyCodeStyle.git

# 2. 为 Claude Code 创建软链接（将 <repo-path> 替换为克隆后的实际目录路径）
ln -s <repo-path> ~/.claude/skills/qzero-code-style

# 3. 为 Codex 创建软链接
ln -s <repo-path> ~/.codex/skills/qzero-code-style
```

### 方法二：通过 npx 一键安装

```bash
npx skills add https://github.com/QZero233/MyCodeStyle.git
```

### 方法三：直接克隆到 skills 目录

```bash
# Claude Code
git clone git@github.com:QZero233/MyCodeStyle.git ~/.claude/skills/qzero-code-style

# Codex
git clone git@github.com:QZero233/MyCodeStyle.git ~/.codex/skills/qzero-code-style
```

### 方法四：手动安装

1. 下载或克隆本仓库到本地任意路径
2. 将仓库目录通过软链接或直接复制放置到对应工具的 skills 目录：
   - **Claude Code**: `~/.claude/skills/`
   - **Codex**: `~/.codex/skills/`
3. 确保目录结构如下：

```text
qzero-code-style/
├── SKILL.md        # Skill 定义文件
├── CLAUDE.md       # 项目级指令
├── CODE_STYLE.md   # 代码风格规范
├── GIT_STYLE.md    # Git 提交规范
├── README.md       # 本说明文档
└── LICENSE         # MIT 许可证
```

### 验证安装

重启 Claude Code 或 Codex 后，在对话中输入：

```
/qzero-code-style
```

如果安装成功，该 Skill 将被激活，并在后续写代码和 Git 提交时自动引用对应的规范文档。

---

## 规范概览

### 代码规范 (CODE_STYLE.md)

| 编号 | 规则 | 核心要点 |
|------|------|----------|
| #1 | 只解决当前问题 | 不为未来需求设计接口，不为一个实现建抽象层 |
| #2 | 少写 Wrapper | 只有在隐藏复杂逻辑、隐藏第三方接口、保证统一行为时才允许 |
| #3 | 少写 Helper | 禁止 helper / utils / misc / common，函数须归属明确领域 |
| #4 | 一个函数只做一件事 | 输入、输出、副作用明确，不混入日志/metric/参数检查 |
| #5 | 不要为了 DRY 而 DRY | 允许少量重复，如果抽象理解成本更高就保持重复 |
| #6 | 优先 Early Return | 避免深层 if 嵌套，用 early return 展平控制流 |
| #7 | 不要写显而易见的注释 | 注释解释"为什么"而非"是什么代码" |
| #8 | 变量必须体现业务 | 避免 data/result/temp/item/obj 等无意义命名 |
| #9 | 控制抽象层数 | Controller → Service → Repository，不超过五六层 |
| #10 | 日志只记录有价值的信息 | 记录错误、性能、状态变化、关键参数，不记录 start/end |
| #11 | 不要写防御性模板代码 | 只在存在真实风险时判空/try-catch/参数检查 |
| #12 | 一个 Context 优于十几个参数 | 参数超 5 个时优先用 RequestContext |
| #13 | 保持代码有"人味" | 允许少量重复、长短不一致的函数、不完全对称的实现 |
| #14 | 每增加一层都问自己一句 | "这一层隐藏了什么复杂性？" 答不上来就删除 |
| #15 | 优先可读性 | 性能一致时永远选最容易读懂的那一种 |
| #16 | 注释规范 | 函数前加注释（作用/入参/返回值/副作用），分支路径前标注触发条件 |
| #17 ⚠️ | 更新注释规范 | 修改代码后必须同步检查注释，硬性要求，提交前必执行 |
| #18 | 用接口替代函数传递 | 不传裸函数/回调，定义职责清晰、注释完备的接口 |
| #19 | 接口命名以 I 开头 | Kotlin/Java 接口必须以大写 I 开头（如 ISkillMetaInfoProvider） |
| #20 | 禁止魔法字符串/数字 | 可枚举量必须用 enum，传输边界用常量+注释做转换 |

### Git 提交规范 (GIT_STYLE.md)

- **Commit Message 格式**：`<type>(<scope>): <description>`，description 用中文
- **Commit Body 要求**：
  1. 说明本次修改的主要内容及要解决的问题
  2. 逐文件说明每个文件的修改内容与原因

---

## 目录结构

```text
qzero-code-style/
├── SKILL.md        # Skill 定义文件（引用各规范文档）
├── CLAUDE.md       # 项目级指令
├── CODE_STYLE.md   # 代码风格规范（20 条规则）
├── GIT_STYLE.md    # Git 提交规范
├── README.md       # 本说明文档
└── LICENSE         # MIT 许可证
```

---

## 许可

MIT License
