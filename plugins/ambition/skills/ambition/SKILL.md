---
name: ambition
description: 大规模软件开发全生命周期管理器，覆盖 DISCOVER（需求采集）→ DEFINE（PRD/用户故事）→ PROTOTYPE（原型）→ BREAKDOWN（需求拆分）→ TRACK（状态管理/多 agent 调度）→ REPORT（进度汇报）六阶段。用于项目规划、PRD、原型、任务拆分、sprint、执行跟踪、多 agents 并行实现、轮询收敛结果、进度汇报等多阶段开发工作。
---

# Dev Workflow — 大规模开发全生命周期管理

把一个想法从模糊描述带到可执行交付。状态持久化在项目根目录 `.dev-workflow/`，任何新对话读取状态文件即可无缝续接。

## 六阶段总览

| 阶段 | 名称 | 产出 | 核心工件 |
|------|------|------|---------|
| 1 | **DISCOVER** 需求采集 | 弄清楚要做什么、为谁做 | `discovery.md` + assumption log |
| 2 | **DEFINE** 需求定义 | 可评审的需求规格 | `prd.md`（user stories + MoSCoW + AC） |
| 3 | **PROTOTYPE** 原型创建 | 编码前验证设计假设 | 线框图 / React artifact / API contract / ERD / 时序图 |
| 4 | **BREAKDOWN** 需求拆分 | 可并行执行的任务树 | `backlog.md`（Epic→Story→Task→Subtask） |
| 5 | **TRACK** 状态管理 | 持续推进、暴露阻塞 | 任务状态机 + sprint burndown + agent runbook |
| 6 | **REPORT** 进度汇报 | 对内对外同步 | standup / sprint review / RAG status |

阶段不是严格瀑布：TRACK 和 REPORT 贯穿开发期；发现需求变化可回到 DEFINE 修订 PRD（记录版本变更）。

---

## 两种入口模式

### 模式 A：自动规划（默认）
用户给出简单描述 + 项目深度，自动规划全流程、补足中间步骤：

> "我要做一个在线教育平台，标准深度"
> "/ambition 电商后台，快速验证就行"

**深度等级**（决定每个阶段做多深，不改变六阶段框架）：

| 深度 | 适用 | DISCOVER | DEFINE | PROTOTYPE | BREAKDOWN |
|------|------|----------|--------|-----------|-----------|
| `quick` 快速验证 | Demo/黑客松 | 5–8 个关键问题 | 精简 PRD（4 节） | text wireframe | Story→Task 两层 |
| `standard` 标准 | MVP/小团队 | 完整问题清单 | 标准 PRD（8 节） | wireframe + API contract | Epic→Story→Task |
| `full` 完整产品 | 正式产品 | 全维度 + NFR | 完整 12 节 PRD | React artifact + ERD | 全四层 + 依赖图 |
| `enterprise` 企业级 | 多团队/合规 | 全维度 + 干系人访谈 | 12 节 + 评审记录 | 全五种原型 | 全四层 + 排期 |

深度可从描述推断（"Hackathon"→quick，"企业系统"→enterprise）；推断不了就问一句。

### 模式 B：自定义
用户列出自己的阶段/流程，AI 为每个阶段补充：具体任务（3–7 条）、准入标准、完成标准（DoD）、交付物。补充后展示给用户确认再开始。

> "/ambition --custom"
> "我的流程是调研→方案→开发→上线，帮我补充"

用户自定义阶段可映射到六阶段工件（如"调研"→DISCOVER 的问题清单），缺失的关键环节（如完全没有需求定义）要提醒，但尊重用户最终决定。

---

## 阶段 1：DISCOVER 需求采集

读 `references/discovery-questions.md` 获取分类问题库（问题域/用户/功能/NFR/技术/风险六类）。

执行方式：
1. 先从用户描述里**提取已知信息**，绝不重复问已回答的
2. 按深度选问题数量，**一次最多问 5 个**，分批进行，保持对话感
3. 无法立即确认的判断写入 **assumption log**：

```markdown
## Assumption Log
| # | 假设 | 影响 | 状态 |
|---|------|------|------|
| A1 | 用户主要用移动端访问 | 决定响应式优先级 | ⚠️ 未确认 |
| A2 | 日活预计 < 1万 | 单体架构足够 | ✅ 已确认 2026-06-09 |
```

产出 `discovery.md`：已知信息汇总 + 问答记录 + assumption log + 开放问题。所有 ⚠️ 假设在进入 DEFINE 前逐条向用户确认或显式标记"带假设推进"。

## 阶段 2：DEFINE 需求定义

读 `references/prd-template.md` 获取完整 12 节 PRD 模板（含按深度裁剪指引）。

核心要求：
- **User stories**：`作为 [角色]，我希望 [行为]，以便 [价值]`，每条有唯一 ID（US-001）
- **MoSCoW 优先级**：Must / Should / Could / Won't(this time)——Won't 显式写出来，是范围控制的红线
- **验收标准用 Given/When/Then**：

```gherkin
US-003 用户登录
Given 已注册用户在登录页
When 输入正确的邮箱和密码并提交
Then 跳转到主页，导航栏显示用户名
Given 连续输错密码 5 次
When 再次尝试登录
Then 账户锁定 15 分钟并提示
```

PRD 完成后向用户做一次**评审走查**：逐节确认 Must 列表和 Won't 列表，签字（用户说"确认"）后才进 PROTOTYPE。

## 阶段 3：PROTOTYPE 原型创建

五种原型类型，按需选择（一个项目通常用 2–3 种）：

| 类型 | 验证什么 | 适用 |
|------|---------|------|
| **Text wireframe** | 页面布局和信息架构 | 所有深度，ASCII 框图 + 交互说明 |
| **React artifact** | 真实交互体验 | 有 UI 的项目，用真实示例数据，不用 Lorem ipsum |
| **API contract** | 前后端接口约定 | 有后端的项目，OpenAPI 风格端点定义 + 请求/响应示例 |
| **ERD** | 数据模型完整性 | 有持久化的项目，mermaid `erDiagram` 或表格 |
| **Sequence diagram** | 多组件协作流程 | 复杂流程（支付、鉴权、异步任务），mermaid `sequenceDiagram` |

每个原型做完都对照 PRD 的 Must 故事走查一遍：每个 Must 故事在原型里能走通吗？走不通的要么改原型，要么回 DEFINE 改需求。原型文件存 `.dev-workflow/prototypes/`。

## 阶段 4：BREAKDOWN 需求拆分

读 `references/estimation-guide.md` 获取故事点校准例子和容量规划规则。

四层结构：

```
Epic（一个完整业务能力，1–4 周）
└── Story（一个用户可感知的功能，对应 PRD 的 US-xxx）
    └── Task（一个工程任务，0.5–2 天，可独立验证）
        └── Subtask（可选，< 4 小时的步骤）
```

拆分规则：
- 每个 Story 必须能追溯到 PRD 故事 ID；找不到对应的，说明 PRD 漏了，回去补
- Task 用 **T恤尺码估点**：XS(1) / S(2) / M(3) / L(5) / XL(8)——XL 必须继续拆
- 标注依赖（`依赖: T003`），画出关键路径
- 按 MoSCoW 排序装入 sprint，Must 全部进前几个 sprint

产出 `backlog.md`，格式：

```markdown
## EPIC-1 用户系统 [M, 13pt]
### US-001 用户注册 [Must, 5pt]
| ID | 任务 | 点数 | 依赖 | 状态 |
|----|------|------|------|------|
| T001 | 设计用户表结构 | 1 | — | BACKLOG |
| T002 | 注册 API + 校验 | 3 | T001 | BACKLOG |
```

## 阶段 5：TRACK 状态管理

**任务状态机**（唯一合法状态集，状态写在 backlog.md 的状态列）：

```
BACKLOG → TODO → IN_PROGRESS → IN_REVIEW → DONE
                      ↓ ↑
                   BLOCKED（任何活跃状态都可进入/退出）
```

- `BACKLOG`：已拆分未排期 → `TODO`：进入当前 sprint → `IN_PROGRESS`：正在做（**同时 IN_PROGRESS 的任务 ≤ 2**，强制聚焦）→ `IN_REVIEW`：待验证（对照 AC 检查）→ `DONE`：AC 全过
- `BLOCKED` 必须带原因和记录时间；阻塞 > 1 天 → 绕开做别的并升级到 REPORT 层面

**Sprint burndown**：每个 sprint 在 `project.md` 维护剩余点数记录：

```markdown
### Sprint 2 燃尽（总 21pt，6/15–6/26）
| 日期 | 剩余点数 | 备注 |
|------|---------|------|
| 6/15 | 21 | sprint 开始 |
| 6/17 | 16 | T012 完成 |
| 6/19 | 16 | T015 BLOCKED：等第三方 API key |
```

剩余点数连续 3 天不降 = 红色信号，主动向用户提出：减范围 / 解阻塞 / 延期，三选一。

**进度命令**：用户说"当前进度"→ 读状态文件汇报；"完成了 T012"→ 更新状态机 + 燃尽表；"T015 被卡住了"→ 标 BLOCKED 记原因；"下个 sprint"→ 结算本 sprint、生成 review、装载下个 sprint。

### 多 agent 调度与轮询收敛

当用户要求"执行 backlog"、"多 agents 跑起来"、"并行推进"、"轮询结果"或当前 sprint 有 2 个以上互不依赖任务时，进入 agentic execution。只有在运行环境提供 subagent/thread/task 工具时才调度；没有则按同一 runbook 顺序执行并在 `agents.md` 标记 `mode: sequential-fallback`。

**准入检查**：
1. 只调度状态为 `TODO`、依赖全 `DONE`、验收标准清楚、点数 ≤ M(3) 的任务；L/XL 先拆。
2. 同一批次不得让两个 agents 修改同一文件、同一接口契约或同一数据迁移；有共享面就串行。
3. 每个任务必须有可运行验证命令或可观察交付物；没有验证就先补任务说明。
4. 保持主控 agent 负责调度、轮询、合并、最终验证；worker agents 不直接改全局计划。

**调度批次**：
- 从当前 sprint 选最多 `min(可用独立任务数, 4)` 个任务并行；默认 2-3 个，除非用户明确要求更大并发。
- 给每个 worker 一个自包含 prompt：任务 ID、关联 US、验收标准、允许修改范围、禁止范围、验证命令、期望返回格式。
- 将被派发任务从 `TODO` 改为 `IN_PROGRESS`，并在 `.dev-workflow/agents.md` 追加 run 记录。

Worker 返回格式固定为：

```markdown
STATUS: DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED
TASK: T012
CHANGED: files or artifacts
VERIFY: command + result summary
NOTES: risks, assumptions, follow-ups
```

**轮询规则**：
- 主控 agent 每轮检查所有活跃 worker 的状态；工具支持异步时按工具结果轮询，手动报告时每次用户问进度都先读 `agents.md`。
- 每轮把 worker 状态写入 `agents.md`，并同步 `backlog.md`：`DONE` 先进入 `IN_REVIEW`，`BLOCKED` 标原因，`NEEDS_CONTEXT` 补上下文后最多重派 2 次。
- 任何 worker 超过约定 SLA（默认 30 分钟或本环境可用的最长合理等待）无结果，在 `agents.md` 标 `STALE`，不要无限等；`backlog.md` 任务退回 `TODO` 或按原因标 `BLOCKED`，然后重派或拆小。
- 连续两轮无新增 DONE 且仍有活跃阻塞，生成 REPORT 风险提示。

**收敛与验收**：
1. 对每个 `DONE/DONE_WITH_CONCERNS` 结果先做 spec review：对照 PRD 的 user story 和 Given/When/Then。
2. 再做 integration review：检查文件冲突、接口契约、迁移顺序、测试覆盖和副作用。
3. 运行批次级验证命令；通过后把任务改为 `DONE`，更新 burndown；失败则生成修复任务或重派原 worker。
4. 批次完成后写一段 convergence summary：完成任务、未完成任务、阻塞、验证证据、下一批候选。

`agents.md` 结构：

```markdown
# Agent Runs

## RUN-20260619-001 Sprint 2 Batch 1
**mode**: parallel | **status**: POLLING | **started**: 2026-06-19 14:00
**batch goal**: 完成登录 Must path

| Agent | Task | 状态 | Lease/SLA | 修改范围 | 验证 | 备注 |
|-------|------|------|-----------|----------|------|------|
| A1 | T012 | IN_PROGRESS | 30m | auth api | npm test auth | - |
| A2 | T013 | BLOCKED | 30m | login ui | npm test ui | 缺少路由约定 |

### Poll Log
| 时间 | 事件 | 决策 |
|------|------|------|
| 14:10 | A1 DONE, A2 NEEDS_CONTEXT | 给 A2 补充 router contract |

### Convergence Summary
- DONE: T012
- IN_REVIEW: T013
- BLOCKED: -
- Verification: `npm test auth` passed; batch e2e pending
```

**调度命令**：用户说"开始执行 sprint"→ 选第一批可并行任务并派发；"轮询 agents"→ 读取/更新 `agents.md` 并收敛结果；"收敛这一批"→ review + 验证 + 更新 backlog/burndown；"停止 agents"→ 不再派发新任务，活跃任务收敛或标 STALE。

## 阶段 6：REPORT 进度汇报

三种汇报格式，按用户要求生成（"给我个 standup" / "sprint 总结" / "给老板看的状态"）：

**Daily standup**（3 行原则）：
```markdown
**昨日**: T012 注册 API 完成（DONE）
**今日**: T013 登录页前端（IN_PROGRESS）
**阻塞**: T015 等第三方 API key（已阻塞 2 天 ⚠️）
```

**Sprint review**：完成 vs 计划点数、DONE 故事列表（对照 AC）、未完成项去向（顺延/砍掉）、本 sprint 决策记录、下 sprint 计划。

**Executive RAG status**（高管视角，一屏读完）：
```markdown
## [项目名] 状态报告 — 2026-06-09
**总体**: 🟡 AMBER
| 维度 | 状态 | 说明 |
|------|------|------|
| 进度 | 🟢 GREEN | Sprint 2/4，燃尽正常 |
| 范围 | 🟡 AMBER | 新增支付需求待评估，可能 +8pt |
| 风险 | 🔴 RED | 第三方 API 审批阻塞关键路径 3 天 |
**需要决策**: 是否接受支付需求顺延到 v1.1？
```

RAG 判定要诚实：有阻塞关键路径的问题就是 RED，不粉饰。

---

## 状态文件结构

```
.dev-workflow/
├── project.md       # 主状态：阶段概览、当前 sprint、燃尽表、决策记录
├── discovery.md     # DISCOVER 产出（含 assumption log）
├── prd.md           # DEFINE 产出
├── backlog.md       # BREAKDOWN 产出 + TRACK 状态列
├── agents.md        # 多 agent 调度、轮询、收敛记录
├── prototypes/      # 原型文件
└── reports/         # sprint review、RAG 报告存档
```

`project.md` 顶部固定结构（新对话恢复上下文的入口）：

```markdown
# [项目名]
**深度**: standard | **当前阶段**: 5-TRACK | **当前 Sprint**: 2/4
**总体状态**: 🟡

## 阶段进度
| 阶段 | 状态 | 完成日期 |
|------|------|---------|
| 1 DISCOVER | ✅ | 2026-06-02 |
| 2 DEFINE | ✅ | 2026-06-04 |
...

## 决策记录
| 日期 | 决策 | 理由 |
```

---

## 核心原则

**匹配深度，不过度设计。** quick 项目用精简 PRD，不要 50 页规格书。文档是为决策服务的，不是为存在而存在。

**可追溯链条。** Task → Story → PRD 故事 ID → discovery 中的用户需求，每一层都能向上追溯。断了链条的工作项要质疑其必要性。

**状态文件是唯一真相。** 所有状态变更、决策、阻塞都落盘。对话里说过但没写进文件的等于没发生。

**80% 就推进，但假设要显式。** 不确定的事写进 assumption log 继续走，不要停下来等完美信息，也不要悄悄拍板。

**阻塞要喊出来。** BLOCKED 状态、燃尽停滞、范围膨胀，主动汇报并给出选项，不等用户来问。

**主控负责收敛。** 多 agent 是执行加速器，不是责任转移。主控 agent 必须轮询、复核、合并、验证，并把结果写回状态文件。

**立即开始。** 规划确认后马上进入 DISCOVER 提第一批问题，不要只交付计划表就停。

## 参考文件

- `references/prd-template.md` — 完整 12 节 PRD 模板 + 按深度裁剪指引（进入 DEFINE 时读）
- `references/discovery-questions.md` — 六类问题库（进入 DISCOVER 时读）
- `references/estimation-guide.md` — 故事点校准例子 + sprint 容量规则（进入 BREAKDOWN 时读）
