# Superpowers 完整开发循环

## 概述

Superpowers 的完整开发循环由 14 个核心技能组成，按阶段串联，确保 AI 代理遵循成熟的软件工程实践。

---

## 循环流程图

```
用户请求
   │
   ▼
[1] brainstorming          ← 精炼设计，苏格拉底式提问
   │ 设计获批
   ▼
[2] using-git-worktrees    ← 创建隔离分支工作区
   │ 工作区就绪
   ▼
[3] writing-plans          ← 分解任务（每个 2-5 分钟）
   │ 计划确认
   ▼
[4] subagent-driven-development / executing-plans   ← 执行计划
   │
   ├─── 每个任务内部循环:
   │       │
   │       ▼
   │    [4a] test-driven-development
   │         RED   → 写失败测试
   │         GREEN → 最小代码通过
   │         REFACTOR → 重构
   │         Commit 每个阶段
   │       │
   │       ▼
   │    [4b] requesting-code-review   ← 派遣 reviewer 子代理
   │       │
   │       ▼
   │    [4c] receiving-code-review    ← 系统化响应反馈
   │
   ├─── 遇到 Bug 时:
   │       │
   │       ▼
   │    [5] systematic-debugging      ← 4 阶段根因分析
   │
   ▼
[6] verification-before-completion    ← 声明完成前的强制验证
   │ 验证通过
   ▼
[7] finishing-a-development-branch    ← 选择: merge / PR / 保留 / 丢弃
   │
   ▼
完成
```

---

## 各阶段详解

### 阶段 1 — 设计精炼

**技能:** `superpowers:brainstorming`

**触发时机:** 任何创建功能、构建组件、修改行为之前（强制）

**目的:** 在写代码前，通过提问厘清需求、探索设计方案。

**过程:**
1. 探索用户意图与真实需求
2. 提出替代方案并对比权衡
3. 以易消化的方式呈现设计
4. 保存设计文档供后续参考

---

### 阶段 2 — 工作区隔离

**技能:** `superpowers:using-git-worktrees`

**触发时机:** 开始任何需要与当前工作区隔离的功能开发前

**目的:** 创建独立的 git worktree，保证并行开发不产生分支冲突。

**过程:**
1. 创建 worktree（智能选择目录）
2. 运行项目初始化
3. 验证测试基线（确保初始状态干净）

---

### 阶段 3 — 制定计划

**技能:** `superpowers:writing-plans`

**触发时机:** 设计获批、实现前

**目的:** 将设计拆解为 2-5 分钟的小任务，每个任务含：
- 确切的文件路径
- 完整的代码片段
- 验证步骤

**原则:** 假设执行者技能熟练但对上下文一无所知。

---

### 阶段 4 — 执行计划

**技能:** `superpowers:subagent-driven-development` 或 `superpowers:executing-plans`

**触发时机:** 有书面计划、开始实现时

**区别:**
- `subagent-driven-development`：有子代理支持时使用，每任务后自动 review
- `executing-plans`：无子代理时使用，顺序执行并手动验证

#### 每个任务内的子循环

##### 4a. 测试驱动开发

**技能:** `superpowers:test-driven-development`

**铁律:** 没有失败的测试，绝不写实现代码。

| 阶段 | 行动 | Commit |
|------|------|--------|
| RED | 写失败测试，确认失败原因正确 | `test: add failing test for X` |
| GREEN | 写最小代码使测试通过 | `feat: implement X` |
| REFACTOR | 重构，保持测试绿色 | `refactor: clean up X` |

##### 4b. 请求代码审查

**技能:** `superpowers:requesting-code-review`

**触发时机:** 每个任务完成后、合并前

**过程:**
1. 获取当前 git SHA
2. 派遣 code-reviewer 子代理
3. 按严重级别处理反馈：
   - **Critical** — 立即修复
   - **Important** — 继续前修复
   - **Minor** — 记录备用

##### 4c. 响应代码审查

**技能:** `superpowers:receiving-code-review`

**触发时机:** 收到审查反馈后

**响应步骤:**
1. **READ** — 完整读取反馈，不急于反应
2. **UNDERSTAND** — 用自己的话复述问题
3. **VERIFY** — 在代码库中核实反馈是否准确
4. **EVALUATE** — 评估对本代码库是否技术合理
5. **RESPOND** — 确认或有据可依地反驳
6. **IMPLEMENT** — 逐条实现，每条单独测试

---

### 阶段 5 — 系统化调试（按需）

**技能:** `superpowers:systematic-debugging`

**触发时机:** 遇到任何 bug、测试失败、意外行为

**铁律:** 在理解根因之前，永不修改代码。

**4 个阶段:**
1. 根因调查
2. 假设形成
3. 验证假设
4. 实施修复

**包含技术:** 根因追踪、纵深防御、条件等待

---

### 阶段 6 — 完成前验证

**技能:** `superpowers:verification-before-completion`

**触发时机:** 即将声明工作"完成"、"已修复"、"可合并"之前

**铁律:** 合理化不是验证。

**验证清单:**
- 运行完整测试套件
- 检查覆盖关键路径
- 识别并消除自我欺骗的红旗
- 确认输出与预期一致

**红旗信号（出现则停止）:**
- "测试应该通过" 而非实际运行
- "这部分应该没问题" 而非验证过
- 只运行了部分测试

---

### 阶段 7 — 分支收尾

**技能:** `superpowers:finishing-a-development-branch`

**触发时机:** 实现完成、所有测试通过、准备整合

**核心原则:** 测试通过 → 呈现选项 → 用户选择 → 清理

**选项:**
| 选项 | 适用场景 |
|------|---------|
| 本地合并 | 快速整合，无需审查 |
| 创建 PR | 需要团队审查或 CI |
| 保留分支 | 工作暂停，稍后继续 |
| 丢弃分支 | 方案作废，从头再来 |

---

## 并行执行（可选优化）

**技能:** `superpowers:dispatching-parallel-agents`

**触发时机:** 存在 2 个及以上相互独立的任务

**过程:**
1. 识别独立的工作域
2. 创建聚焦的代理任务
3. 并行派遣，互不干扰
4. 审查并整合结果

---

## 关键原则汇总

| 原则 | 说明 |
|------|------|
| **先设计，再编码** | brainstorming 是强制前置步骤 |
| **测试先行** | RED-GREEN-REFACTOR，无例外 |
| **根因优先** | 不明白原因就不修改代码 |
| **证据优于声明** | 运行验证，不凭感觉说"完成了" |
| **每任务审查** | 不等到最后才 review |
| **简单第一** | 减少复杂度是首要目标 |

---

## 技能速查表

| 技能 | 触发时机 |
|------|---------|
| `brainstorming` | 任何创意工作前（强制） |
| `using-git-worktrees` | 功能开发前，需要隔离 |
| `writing-plans` | 设计批准后，实现前 |
| `test-driven-development` | 实现任何功能或修复前 |
| `executing-plans` | 有计划、无子代理时 |
| `subagent-driven-development` | 有计划、有子代理时 |
| `systematic-debugging` | 遇到 bug 或失败时 |
| `requesting-code-review` | 任务完成后、合并前 |
| `receiving-code-review` | 收到审查反馈后 |
| `verification-before-completion` | 声明完成前 |
| `finishing-a-development-branch` | 实现完成、测试通过后 |
| `dispatching-parallel-agents` | 存在 2+ 独立任务时 |
