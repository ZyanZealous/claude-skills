---
name: ambition
description: 大规模软件开发全生命周期管理器，覆盖 DISCOVER（需求采集）→ DEFINE（PRD/用户故事）→ PROTOTYPE（原型）→ BREAKDOWN（需求拆分）→ TRACK（状态管理）→ REPORT（进度汇报）六个阶段。两种模式：（1）自动规划——用户给出简单描述和项目深度，自动规划补足中间步骤；（2）自定义——用户定义流程，AI 补充细节。在以下情况使用：用户描述想开发的项目、提到"需求""PRD""规划""拆分任务""user story""sprint""项目进度""开发流程"、要求生成需求文档或原型、或开始任何多阶段开发工作。即使用户没说"工作流"，只要在规划或管理开发项目就应触发。
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
| 5 | **TRACK** 状态管理 | 持续推进、暴露阻塞 | 任务状态机 + sprint burndown |
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

**立即开始。** 规划确认后马上进入 DISCOVER 提第一批问题，不要只交付计划表就停。

## 参考文件

- `references/prd-template.md` — 完整 12 节 PRD 模板 + 按深度裁剪指引（进入 DEFINE 时读）
- `references/discovery-questions.md` — 六类问题库（进入 DISCOVER 时读）
- `references/estimation-guide.md` — 故事点校准例子 + sprint 容量规则（进入 BREAKDOWN 时读）
