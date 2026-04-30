# 赏析：using-git-worktrees（Git 工作树技能）

## 技能定位与核心目标

`using-git-worktrees` 是 Superpowers 体系中的**工作空间隔离基础设施技能**，专门解决多任务并行开发时的分支切换问题。核心目标是：通过 Git worktree 机制创建共享同一仓库的隔离工作空间，让 AI 能在不污染当前工作区的情况下安全地开展新功能开发。

这个技能的定位是**被调用者**——它被 `brainstorming`、`subagent-driven-development`、`executing-plans` 等上层技能强制依赖，是整个开发流程的隔离地基。

---

## 核心设计哲学

### 1. 系统性目录选择而非随意创建（Systematic Directory Selection）

技能的核心原则是"系统性目录选择 + 安全验证 = 可靠隔离"。目录选择遵循严格的优先级顺序：

1. **检查已有目录**（`.worktrees/` > `worktrees/`）
2. **检查 CLAUDE.md 配置**（项目级偏好）
3. **询问用户**（最后手段）

这个优先级设计的动机是：尊重项目已有的约定，避免在同一项目中出现多种 worktree 目录风格。如果项目已经有 `.worktrees/`，就不应该再创建 `worktrees/`，否则会造成混乱。

### 2. 安全验证优先于创建（Safety Before Creation）

对于项目本地目录（`.worktrees/` 或 `worktrees/`），技能要求在创建 worktree 之前**必须**验证该目录已被 `.gitignore` 忽略：

```bash
git check-ignore -q .worktrees 2>/dev/null
```

如果未被忽略，技能不会跳过或警告，而是立即修复：添加到 `.gitignore` 并提交。这遵循了"立即修复损坏的东西"（Fix broken things immediately）原则——发现问题就解决，不留技术债。

全局目录（`~/.config/superpowers/worktrees/`）则不需要此验证，因为它完全在项目之外。

### 3. 自动检测而非硬编码（Auto-Detection over Hardcoding）

项目依赖安装步骤通过检测项目文件来自动选择命令：
- `package.json` → `npm install`
- `Cargo.toml` → `cargo build`
- `requirements.txt` → `pip install -r requirements.txt`
- `pyproject.toml` → `poetry install`
- `go.mod` → `go mod download`

这种设计避免了"假设项目使用特定工具"的错误，使技能能跨语言、跨框架通用。

### 4. 基线验证作为前提（Baseline Verification）

创建 worktree 并安装依赖后，技能要求运行测试验证基线是否干净。如果测试失败，不是静默继续，而是**报告失败并询问是否继续**。

这个设计的动机是：如果不验证基线，后续开发中出现的测试失败就无法区分是"新引入的问题"还是"原本就存在的问题"，调试成本会大幅上升。

### 5. 强制声明开始（Announce at Start）

技能要求 AI 在开始时声明："I'm using the using-git-worktrees skill to set up an isolated workspace."

这是一种**透明度设计**——让用户知道 AI 正在做什么，而不是静默地执行一系列 git 命令。这也为用户提供了干预的机会（如果他们不想使用 worktree）。

---

## 关键机制细节

### 目录优先级的边界情况处理

技能明确处理了"两个目录都存在"的情况：`.worktrees/` 优先于 `worktrees/`。这个细节看似微小，但避免了 AI 在面对歧义时随机选择或报错。

### `git check-ignore` 的选择

技能使用 `git check-ignore -q` 而非直接读取 `.gitignore` 文件。这个选择很重要：`git check-ignore` 会考虑本地 gitignore、全局 gitignore 和系统 gitignore，而直接读取 `.gitignore` 只能检查项目级配置。这确保了验证的完整性。

### 项目名称检测

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

使用 `git rev-parse --show-toplevel` 获取仓库根目录，再用 `basename` 提取项目名。这比直接用当前目录名更可靠——即使 AI 当前在子目录中工作，也能正确获取项目名。

### 路径构建的两种模式

技能明确区分了两种路径构建模式：
- **项目本地**：`$LOCATION/$BRANCH_NAME`（相对路径）
- **全局**：`~/.config/superpowers/worktrees/$project/$BRANCH_NAME`（包含项目名，避免不同项目的 worktree 混在一起）

### 与其他技能的集成关系

技能明确声明了它的调用者：
- `brainstorming`（Phase 4，设计批准后）—— **必须调用**
- `subagent-driven-development` —— **必须调用**
- `executing-plans` —— **必须调用**

以及它的配对技能：
- `finishing-a-development-branch` —— 工作完成后的清理

这种显式的集成关系声明，让整个技能体系的依赖图变得清晰可见。

### 四类常见错误的预防

技能专门列出了四类常见错误及其修复方式：

| 错误 | 根本原因 | 修复 |
|------|----------|------|
| 跳过 ignore 验证 | 急于创建 | 始终使用 `git check-ignore` |
| 假设目录位置 | 不遵循优先级 | 严格按优先级：已有 > CLAUDE.md > 询问 |
| 带失败测试继续 | 忽视基线 | 报告失败，获得明确许可 |
| 硬编码安装命令 | 假设技术栈 | 从项目文件自动检测 |

---

## 亮点总结

| 亮点 | 说明 |
|------|------|
| 三级目录优先级 | 已有目录 > CLAUDE.md 配置 > 询问用户，尊重项目约定 |
| `git check-ignore` 验证 | 覆盖本地/全局/系统 gitignore，比直接读文件更可靠 |
| 立即修复原则 | 发现未忽略就立即添加 .gitignore 并提交，不留技术债 |
| 自动检测技术栈 | 支持 Node/Rust/Python/Go，跨语言通用 |
| 基线测试验证 | 创建后立即验证测试通过，确保能区分新旧问题 |
| 强制声明开始 | 透明度设计，让用户知道 AI 正在做什么 |
| 全局目录包含项目名 | 避免不同项目的 worktree 混在一起 |
| 显式集成关系声明 | 明确列出调用者和配对技能，依赖图清晰 |
| 失败时询问而非继续 | 基线测试失败时报告并请求许可，不静默跳过 |

---

## 总结性评价

`using-git-worktrees` 是 Superpowers 体系中**基础设施层**的代表性技能。它解决的问题看似简单（创建一个 git worktree），但技能的价值在于将这个操作中所有可能出错的地方都系统性地处理掉。

这个技能最值得关注的设计是**安全验证优先于创建**的原则。在大多数工具中，"创建 worktree"是一个单步操作；而在这个技能中，创建之前必须经过目录选择、ignore 验证、必要时修复 .gitignore 并提交，之后还要安装依赖、验证基线。这种"把简单操作做复杂"的设计，实际上是在把所有隐性的前提条件显式化。

`git check-ignore` 的选择体现了技能设计者对 Git 机制的深入理解——直接读 `.gitignore` 是初学者的做法，而 `git check-ignore` 才是正确的验证方式。这种细节上的精确性，是这个技能可信赖的基础。

作为被多个上层技能强制依赖的基础设施，这个技能的可靠性直接影响整个开发流程的稳定性。它通过系统性的优先级规则和强制验证步骤，确保了无论在什么项目、什么技术栈下，worktree 的创建都是安全、一致、可预期的。
