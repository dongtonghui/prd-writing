# PRD Writing Skill

> 从会议纪要、用户故事或功能需求出发，撰写完整的产品需求文档（PRD）。覆盖功能模块拆解、看板任务分发、交叉评审、整合输出、UI Demo 设计全流程。

---

## 简介

**prd-writing** 是 Hermes Agent 的 skill，用于将模糊的产品需求转化为结构化、可执行的产品需求文档（PRD）。

它不仅仅是"写文档"，而是一套完整的产品需求工程工作流：

```
输入分析 → 模块拆解 → 看板分发 → 并行撰写 → 交叉评审 → 整合输出 → UI Demo 设计
```

### 核心能力

| 能力 | 说明 |
|------|------|
| **需求结构化** | 从会议纪要/用户故事中提取关键要素，识别信息缺口 |
| **模块拆解** | 将复杂需求拆分为 3-8 个独立功能模块，定义输入/输出/依赖 |
| **看板分发** | 通过 Kanban 系统将模块任务并行分发给子 agent 执行 |
| **三维度交叉评审** | 流程完整性 × 逻辑一致性 × 方案合理性，三个独立维度并行评审 |
| **整合输出** | 将各模块 PRD 整合为完整文档，含数据模型、API、业务规则 |
| **UI Demo 设计** | 基于 PRD 生成可交互的 HTML Demo（调用 claude-design / popular-web-designs） |

---

## 适用场景

- "写 PRD" / "产品需求文档" / "PRD 撰写"
- "功能模块拆解" / "模块拆解"
- "会议纪要转 PRD" / "会议纪要" + "产品"
- "需求文档" / "feature spec" / "product requirement"
- 用户提供会议纪要的文本内容
- 用户提供用户故事（User Story）列表
- 用户描述产品功能需求，需要结构化文档输出

### 不适用场景

| 场景 | 应使用的 Skill |
|------|---------------|
| 用户已有 PRD，需要输出开发代码 | `writing-plans` / `subagent-driven-development` |
| 用户需要修复现有代码 bug | `systematic-debugging` |
| 用户要求直接写代码不做需求分析 | `writing-plans`（直接实施计划） |
| 用户只需要简单页面原型，无需求文档 | `claude-design`（直接生成 HTML） |
| 用户要求做竞品市场调研 | `kanban-worker` + `market-research` profile |

---

## 安装

### 方式一：通过 Hermes CLI 安装（推荐）

```bash
# 克隆到 Hermes skills 目录
hermes skills add prd-writing https://github.com/dongtonghui/prd-writing.git

# 或直接放入 ~/.hermes/skills/
git clone https://github.com/dongtonghui/prd-writing.git ~/.hermes/skills/prd-writing
```

### 方式二：手动安装

```bash
# 1. 克隆仓库
git clone https://github.com/dongtonghui/prd-writing.git

# 2. 复制到 Hermes skills 目录
cp -r prd-writing ~/.hermes/skills/

# 3. 验证安装
hermes skills list | grep prd-writing
```

---

## 使用方法

在 Hermes Agent 对话中，直接触发关键词即可：

```
用户: "帮我写个 PRD，需求是做一个会员体系..."
→ Agent 自动加载 prd-writing skill 并执行工作流

用户: "这是会议纪要，转成 PRD"
→ Agent 自动分析、拆解、撰写

用户: "功能模块拆解一下"
→ Agent 进入 Phase 2 模块拆解阶段
```

### 完整工作流

#### Phase 1: 输入分析
- 读取会议纪要/需求文本
- 提取关键要素（产品名称、目标用户、核心功能、业务规则、时间约束、技术约束、竞品参考）
- 信息缺口检查（缺失 ≥2 项时暂停询问用户）

#### Phase 2: 功能模块拆解
- 按用户角色/功能域/业务流程/页面界面划分一级模块（3-8 个）
- 每个模块定义：核心功能、输入、输出、依赖模块
- 拆解二级功能点（含优先级 P0/P1/P2、复杂度评估）
- 识别跨模块共享数据实体

#### Phase 3: 看板任务分发
- 每个模块创建一个 Kanban 任务
- 无依赖模块立即并行分发
- 有依赖模块等待依赖完成后分发
- 全局规范/数据模型优先分发

#### Phase 4: 并行撰写
- 子 agent 按统一模板输出模块 PRD
- 模板包含 11 个标准章节：功能概述、用户故事、功能流程、页面设计、数据模型、API 接口、业务规则、异常处理、验收标准、依赖与风险、附录
- 主 agent 传递全局约束（术语表、接口约定、技术平台）

#### Phase 5: 三维度交叉评审（核心亮点）

同时 spawn **3 个独立子 agent**，彼此隔离、互不干扰：

| 维度 | 聚焦目标 | 评审 Checklist |
|------|---------|---------------|
| **A. 流程图完整性** | 所有功能流程是否完整、无断点 | 主流程完整性、分支覆盖度、异常流程覆盖、页面流程闭环、跨模块衔接、状态机完整性 |
| **B. 逻辑一致性** | 模块间数据、术语、权限、接口是否一致 | 术语统一、数据模型冲突、接口一致性、权限逻辑一致、业务规则冲突、数值常量一致 |
| **C. 方案合理性** | 方案设计是否合理、可落地 | 目标对齐度、复杂度合理性、技术可行性、用户体验合理性、性能风险、安全合规、扩展性 |

**汇总裁决**：
- 综合评分 ≥ 75 分且无阻塞项 → ✅ 直接通过
- 综合评分 ≥ 60 分且阻塞项 ≤ 3 个 → ⚠️ 有条件通过（修复后通过）
- 综合评分 < 60 分或阻塞项 > 3 个 → ✗ 不通过（需重新设计）

#### Phase 6: 整合输出
- 根据评审结果修正 PRD
- 输出完整 Markdown 文档到 `/workspace/reports/`
- 含功能模块清单、数据模型图（Mermaid）、API 接口文档

#### Phase 7: UI Demo 设计（可选）
- 调用 `claude-design` 或 `popular-web-designs` 生成核心页面 Demo
- 输出独立 HTML 文件（零依赖，单文件可运行）

---

## 文件结构

```
prd-writing/
├── SKILL.md                          # 技能主文档（完整工作流、规范、反例）
├── README.md                         # 本文件（快速入门）
├── references/
│   ├── skill-usage-guide.md          # 关联 skill 的详细调用参数与代码示例
│   ├── three-dim-review-playbook.md  # 三维度交叉评审实战案例与模板
│   └── subagent-path-isolation.md    # 子 agent 路径隔离问题与解决方案
├── templates/
│   └── prd-module-template.md        # 模块 PRD 的标准 11 章节模板
└── scripts/                          # 辅助脚本（可选）
```

---

## 关联 Skill

| Skill | 用途 | 调用时机 |
|-------|------|---------|
| [kanban-orchestrator](https://github.com/dongtonghui/kanban-orchestrator) | 看板任务编排 | Phase 3 任务分发 |
| [kanban-worker](https://github.com/dongtonghui/kanban-worker) | 看板任务执行 | Phase 3-4 子 agent 执行 |
| [claude-design](https://github.com/dongtonghui/claude-design) | UI Demo 设计 | Phase 7 Demo 生成 |
| [popular-web-designs](https://github.com/dongtonghui/popular-web-designs) | 设计系统参考 | Phase 7 风格选择 |
| [writing-plans](https://github.com/dongtonghui/writing-plans) | 实施计划撰写 | PRD 完成后输出开发计划 |

---

## 核心设计原则

1. **先分析再写，不盲目产出** — Phase 1 信息缺口检查是红线
2. **模块边界清晰** — 3-8 个模块，过多则管理 overhead 爆炸
3. **评审不可跳过** — 三维度交叉评审是质量保证的核心
4. **评审必须隔离** — 3 个评审 agent 独立运行，结论互不污染
5. **假设必须标注** — 所有未确认内容用 `[假设: xxx]` 显式标注
6. **失败有 fallback** — 子 agent 失败时主 agent 直接接手，不阻塞进度

---

## 贡献

欢迎通过 Issue 或 PR 贡献改进：

- 发现 skill 使用中的问题
- 补充更多行业/场景的 PRD 模板
- 优化三维度评审 checklist
- 改进子 agent 的上下文传递策略

---

## License

MIT © Hermes Agent
