---
name: prd-writing
description: "Write comprehensive Product Requirement Documents (PRD) from meeting notes, user stories, or feature requests. Handles functional module decomposition, Kanban task dispatch, cross-module review, integration, and UI demo design. Trigger: '写PRD', '产品需求文档', 'PRD撰写', '功能模块拆解', '会议纪要转PRD', '需求文档', 'product requirement', 'feature spec'."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [prd, product-requirements, documentation, planning, kanban, ui-design]
    related_skills: [kanban-orchestrator, kanban-worker, claude-design, writing-plans]
---

# PRD Writing Skill

从会议纪要、用户故事或功能需求出发，撰写完整的产品需求文档（PRD）。覆盖功能模块拆解、看板任务分发、交叉评审、整合输出、UI Demo设计全流程。

---

## 触发条件

### 适用场景
用户输入包含以下任一关键词或场景：
- "写PRD" / "产品需求文档" / "PRD撰写"
- "功能模块拆解" / "模块拆解"
- "会议纪要转PRD" / "会议纪要" + "产品"
- "需求文档" / "feature spec" / "product requirement"
- 用户提供会议纪要的文本内容
- 用户提供用户故事（User Story）列表
- 用户描述产品功能需求，需要结构化文档输出

### 不适用场景
以下场景**不应使用**本skill，应使用其他skill：

| 场景 | 原因 | 应使用的Skill |
|------|------|-------------|
| 用户已有PRD，需要输出开发代码 | 本skill只到PRD文档，不涉及代码实现 | `writing-plans` / `subagent-driven-development` |
| 用户需要修复现有代码bug | 本skill不处理代码调试 | `systematic-debugging` |
| 用户要求直接写代码不做需求分析 | 本skill强制先做需求分析 | `writing-plans`（直接实施计划） |
| 用户只需要简单页面原型，无需求文档 | 本skill overhead 过大 | `claude-design`（直接生成HTML） |
| 用户要求做竞品市场调研 | 本skill聚焦需求文档，调研只是输入 | `kanban-worker` + `market-research` profile |

---

## 核心工作流

```
输入分析 → 模块拆解 → 看板分发 → 并行撰写 → 交叉评审 → 整合输出 → UI Demo设计
```

### Phase 1: 输入分析（Input Analysis）

**目标**：理解需求范围，识别信息缺口。

#### Step 1.1: 读取输入
- 若用户提供会议纪要/需求文本 → 直接分析
- 若用户提供文件路径 → 读取文件内容
- 若用户仅给一句话需求（如"做一个AI助手"）→ **进入信息补全流程**（见下方「需求模糊时的处理」）

#### Step 1.2: 提取关键信息
从输入中提取以下要素：

| 要素 | 说明 | 缺失时的处理 |
|------|------|-------------|
| 产品名称 | APP/系统名称 | 使用用户提到的名称，或标注「待确认」 |
| 目标用户 | B2B/B2C/内部用户 | 标注「待确认」 |
| 核心功能 | 主要功能点列表 | 从会议纪要逐条提取 |
| 业务规则 | 定价、权限、流程规则 | 逐条提取并标注优先级 |
| 时间约束 | 上线时间、里程碑 | 标注「待确认」 |
| 技术约束 | 平台（iOS/Android/Web）、技术栈 | 标注「待确认」 |
| 竞品参考 | 是否有对标产品 | 标注「无」 |

#### Step 1.3: 信息缺口检查

若以下**关键要素缺失 ≥2 项**，必须暂停并询问用户：
- 产品名称
- 核心功能（无任何功能描述）
- 目标用户

🔴 **CHECKPOINT · STOP**

> 信息缺口 ≥2 项时，向用户展示缺失清单，询问补充。示例：
> "检测到以下信息缺失，需要您补充：
> 1. 目标用户群体（B2B/B2C？）
> 2. 技术平台（iOS/Android/Web/小程序？）
> 3. 预期上线时间？
>
> 补充后可继续生成PRD，或基于合理假设直接开始（标注假设内容）。"

---

### Phase 2: 功能模块拆解（Module Decomposition）

**目标**：将需求拆分为独立、可并行的功能模块。

#### Step 2.1: 一级模块划分

基于需求内容，按以下维度划分一级模块：

| 维度 | 适用场景 | 示例 |
|------|---------|------|
| 用户角色 | 多角色权限差异大 | 管理员/普通用户/访客 |
| 功能域 | 功能间耦合度低 | 会员体系/支付系统/内容管理 |
| 业务流程 | 流程阶段清晰 | 注册→认证→下单→售后 |
| 页面/界面 | 前端驱动型产品 | 首页/详情页/个人中心 |

**原则**：
- 每个模块边界清晰，模块间通过明确接口交互
- 模块数量建议 3-8 个（过少则拆解不足，过多则管理 overhead 过大）
- 每个模块必须包含：模块名称、核心功能、输入、输出、依赖模块

#### Step 2.2: 二级功能点拆解

对每个一级模块，拆解为具体功能点：

```markdown
### 模块X: [模块名称]

**核心功能**: [一句话描述]
**输入**: [需要什么数据/触发条件]
**输出**: [产生什么结果/数据]
**依赖**: [依赖的其他模块]

#### 功能点清单
| 功能点 | 优先级 | 复杂度 | 备注 |
|--------|--------|--------|------|
| X.1 xxx | P0/P1/P2 | 高/中/低 | |
| X.2 xxx | P0/P1/P2 | 高/中/低 | |
```

#### Step 2.3: 数据模型初步识别

识别跨模块的共享数据实体：

| 实体 | 属性 | 所属模块 | 被哪些模块使用 |
|------|------|---------|--------------|
| User | id, name, role, status | 用户模块 | 所有模块 |
| Order | id, user_id, amount, status | 订单模块 | 支付、物流 |

---

### Phase 3: 看板任务分发（Kanban Dispatch）

**目标**：将模块拆解为可并行的看板任务，创建任务记录并分发给子agent执行。

**本Phase的边界**：
- ✅ 做：创建任务、定义任务内容、确认任务清单
- ❌ 不做：实际撰写PRD内容（那是Phase 4的工作）

#### Step 3.1: 创建看板任务

每个功能模块对应一个看板任务：

```json
{
  "task_id": "prd-[模块名]-[序号]",
  "title": "PRD: [模块名称] 需求文档撰写",
  "profile": "final-synthesis",
  "content": "基于以下需求，撰写[模块名称]的详细PRD子文档...",
  "dependencies": ["依赖的任务ID"]
}
```

**任务分配原则**：
- 无依赖的模块 → 立即分发，并行执行
- 有依赖的模块 → 等待依赖完成后分发
- 数据模型/全局规范 → 优先分发（其他模块依赖它）

#### Step 3.2: 子agent执行

使用 `delegate_task` 或 `kanban-worker` 分发任务：

- **profile**: `final-synthesis`（综合撰写）或 `market-research`（调研型模块）
- **context**: 完整的模块定义 + 全局约束 + 输出格式要求
- **toolsets**: `['web', 'file']`（如需调研）或 `['file']`（纯撰写）

🔴 **CHECKPOINT · STOP**

> 展示看板任务清单，确认后再分发：
> "即将创建以下看板任务并并行执行：
> 1. PRD: 会员体系模块
> 2. PRD: 虚拟货币体系模块
> 3. PRD: 支付与退款模块
> ...
> 确认后分发执行？"

---

### Phase 4: 并行撰写（Parallel Writing）

**目标**：各子agent独立完成模块PRD撰写。

**本Phase的边界**：
- ✅ 做：子agent执行撰写、收集输出、质量检查
- ❌ 不做：模块拆解（Phase 2已完成）、最终整合（Phase 6的工作）

#### Step 4.1: 子agent输出格式

每个子agent必须基于 `templates/prd-module-template.md` 输出以下结构的PRD子文档：

```markdown
# [模块名称] PRD

## 1. 功能概述
## 2. 用户故事
## 3. 功能流程
## 4. 页面/界面设计
## 5. 数据模型
## 6. API接口（如需要）
## 7. 业务规则
## 8. 异常处理
## 9. 验收标准
## 10. 依赖与风险
## 11. 附录
```

> 完整模板含字段说明和示例，见 `templates/prd-module-template.md`。子agent应直接填充模板，不改动结构。

#### Step 4.2: 全局约束传递

向每个子agent传递以下全局约束：
- 产品名称、版本、目标用户
- 技术平台约束
- 术语表（统一用词）
- 与其他模块的接口约定

---

### Phase 5: 交叉评审（Cross Review）

**目标**：确保模块间一致性，发现遗漏和冲突。

#### Step 5.1: 整合初稿

收集所有子agent输出，整合为单一文档：

```markdown
# [产品名称] PRD v1.0

## 目录
## 1. 产品概述
## 2. 功能架构图
## 3. 模块PRD（按模块组织）
## 4. 数据模型总览
## 5. 全局业务规则
## 6. 非功能需求
## 7. 附录（术语表、竞品分析等）
```

#### Step 5.2: 一致性检查清单

| 检查项 | 检查方法 | 不通过时的处理 |
|--------|---------|---------------|
| 术语统一 | 搜索同一概念的不同表述 | 统一为术语表定义 |
| 数据模型冲突 | 对比各模块的数据定义 | 召开「数据模型对齐」子任务 |
| 接口一致性 | 检查模块间调用约定 | 修正为统一接口 |
| 权限逻辑一致 | 检查各模块的权限判断 | 统一权限体系 |
| 页面流程闭环 | 检查用户操作流程是否完整 | 补充缺失步骤 |

#### Step 5.3: 评审子agent

dispatch `logic-review` profile 的子agent执行评审：
- 输入：整合后的完整PRD
- 输出：问题清单 + 修改建议

---

### Phase 6: 整合输出（Final Integration）

**目标**：输出最终PRD文档。

#### Step 6.1: 修正与完善

根据评审结果修正PRD：
- 高优先级问题 → 立即修正
- 中优先级问题 → 标注「待确认」
- 低优先级问题 → 记录到「已知问题」

#### Step 6.2: 输出交付物

| 交付物 | 格式 | 保存路径 |
|--------|------|---------|
| 完整PRD文档 | Markdown | `/workspace/reports/[产品名]-prd-v1-[语言].md` |
| 功能模块清单 | Markdown表格 | 内嵌于PRD |
| 数据模型图 | 文本/Mermaid | 内嵌于PRD |
| API接口文档 | Markdown表格 | 内嵌于PRD |

#### Step 6.3: 用户确认

🔴 **CHECKPOINT · STOP**

> "PRD v1.0 已完成，包含以下模块：
> 1. 会员体系（X页）
> 2. 虚拟货币体系（X页）
> ...
> 
> 是否需要：
> - (a) 直接输出完整文档
> - (b) 先输出目录结构确认
> - (c) 针对某模块深入补充"

---

### Phase 7: UI Demo设计（UI Demo Design）

**目标**：基于PRD输出可交互的UI Demo HTML。

#### Step 7.1: 确定Demo范围

与用户确认：
- 哪些页面需要Demo？（建议核心流程页面）
- 设计风格偏好？（参考现有产品/品牌指南）
- 交互深度？（静态展示 / 可点击跳转 / 完整交互）

#### Step 7.2: 调用设计Skill

使用 `claude-design` 或 `popular-web-designs` skill 生成UI Demo：

- 输入：PRD中的页面描述 + 设计偏好
- 输出：独立HTML文件（零依赖，单文件可运行）
- 保存：`/workspace/reports/prd-demos/[页面名].html`

#### Step 7.3: Demo整合

生成 `index.html` 作为Demo导航页：

```html
<!-- 包含所有Demo页面的链接、PRD摘要、更新日志 -->
```

---

## 需求模糊时的处理（信息补全流程）

当用户输入过于模糊（如"做一个AI助手"），执行以下流程：

### Step A: 识别缺失信息

| 必需要素 | 缺失时的影响 |
|---------|-------------|
| 产品名称 | 无法统一术语 |
| 核心功能 | 无法拆解模块 |
| 目标用户 | 无法定义用户故事 |
| 技术平台 | 无法确定技术约束 |

### Step B: 提问策略

一次性列出所有缺失要素，不逐条追问：

> "为了写出准确的PRD，需要补充以下信息：
> 1. 产品名称和定位？
> 2. 目标用户是谁（B2B/B2C/内部）？
> 3. 核心功能有哪些？
> 4. 技术平台（iOS/Android/Web/小程序）？
> 5. 预期上线时间？
> 6. 是否有竞品参考？
>
> 您可以直接回复补充，或说'基于假设开始'（我会标注假设内容）。"

### Step C: 基于假设执行

若用户选择「基于假设开始」：
- 所有假设内容用 `[假设: xxx]` 标注
- 在PRD「已知问题」章节列出所有假设
- 建议用户后续确认假设内容

---

## 反例与黑名单（Don't Do This）

| # | 反模式 | 后果 | 正确做法 |
|---|--------|------|---------|
| 1 | **直接输出PRD不确认需求** | 产出与用户意图不符，返工成本高 | Phase 1必须做信息缺口检查 |
| 2 | **模块拆解过细（>10个模块）** | 管理 overhead 爆炸，看板任务过多 | 控制在3-8个模块，相关功能合并 |
| 3 | **跳过交叉评审直接整合** | 模块间数据模型冲突、术语不一致 | Phase 5必须执行一致性检查 |
| 4 | **不标注假设直接写** | 用户误以为PRD基于确认事实 | 所有假设用 `[假设: xxx]` 显式标注 |
| 5 | **UI Demo覆盖全部页面** | 工作量过大，核心流程被稀释 | 只Demo核心流程页面（3-5个） |
| 6 | **子agent不带全局约束** | 各模块术语、数据定义不一致 | Phase 4必须传递术语表和接口约定 |
| 7 | **评审问题不分类优先级** | 低优先级问题阻塞交付 | 分高/中/低三级，高优先级才阻塞 |
| 8 | **输出后不确认直接结束** | 用户可能有补充需求未表达 | Phase 6.3必须暂停确认 |

---

## 异常与边界条件

| 场景 | 触发条件 | 处理动作 |
|---|---|---|
| 用户输入非中文 | 检测到主要语言非中文 | 用用户语言输出PRD，但保留中文术语对照表 |
| 需求中途变更 | 用户在看板执行中补充新需求 | 记录到「需求变更日志」，评估是否影响已分发任务；影响大则暂停重新规划 |
| 子agent超时/失败 | delegate_task 返回错误 | 1. 记录失败任务ID和错误信息<br>2. 重试1次（相同context）<br>3. 仍失败 → 主agent直接撰写该模块，标注 `[fallback: 主agent撰写]`<br>4. 在「已知问题」记录子agent失败原因 |
| 模块间依赖循环 | A依赖B，B依赖A | 打破循环：提取公共依赖为独立模块，或标记为「需架构评审」 |
| 用户要求跳过某Phase | 用户说"直接写"或"跳过评审" | 告知风险（如"跳过评审可能导致模块不一致"），用户坚持则执行并标注 |
| PRD体积过大（>200KB） | 整合后文件过大 | 拆分为「主PRD+模块子文档」，主PRD保留摘要和链接 |
| 无看板系统可用 | kanban-worker 无法连接 | 退化为顺序执行：主agent逐模块撰写，不并行 |

---

## 速查表（TL;DR）

```
用户给需求 → Phase 1 分析（缺口≥2则暂停询问）
         → Phase 2 拆模块（3-8个，含输入/输出/依赖）
         → Phase 3 看板分发（确认后并行）
         → Phase 4 子agent撰写（传全局约束）
         → Phase 5 交叉评审（一致性检查清单）
         → Phase 6 整合输出（修正+确认）
         → Phase 7 UI Demo（可选，核心页面）

红线：不确认需求直接写 / 模块>10个 / 跳过评审 / 不标假设
```

---

## 关联Skill

| Skill | 用途 | 调用时机 | 详细用法 |
|-------|------|---------|---------|
| kanban-orchestrator | 看板任务编排 | Phase 3 任务分发 | 见 `references/skill-usage-guide.md` §kanban-orchestrator |
| kanban-worker | 看板任务执行 | Phase 3-4 子agent执行 | 见 `references/skill-usage-guide.md` §kanban-worker |
| claude-design | UI Demo设计 | Phase 7 Demo生成 | 见 `references/skill-usage-guide.md` §claude-design |
| popular-web-designs | 设计系统参考 | Phase 7 风格选择 | 见 `references/skill-usage-guide.md` §popular-web-designs |
| writing-plans | 实施计划撰写 | PRD完成后，如需输出开发计划 | 见 `references/skill-usage-guide.md` §writing-plans |

> 每个关联skill的具体调用参数、代码示例、输出规范详见 `references/skill-usage-guide.md`。
