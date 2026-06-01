# PRD Writing 关联 Skill 使用指南

本文件详细说明 `prd-writing` skill 在哪些 Phase 调用哪些关联 skill，以及具体调用方式。

---

## kanban-orchestrator

**调用时机**: Phase 3 看板任务分发
**用途**: 将模块拆解结果编排为看板任务队列

### 调用方式

```python
# 方式1: 使用 kanban CLI（如已安装）
terminal("kanban task create --title 'PRD: 会员体系' --profile final-synthesis --content '...'")

# 方式2: 使用 todo 工具（Hermes 内置）
todo(todos=[
    {"id": "prd-001", "content": "PRD: 会员体系模块", "status": "pending"},
    {"id": "prd-002", "content": "PRD: 虚拟货币体系模块", "status": "pending"},
])
```

### 参数说明

| 参数 | 必填 | 说明 |
|------|------|------|
| title | ✓ | 任务标题，格式: "PRD: [模块名称]" |
| profile | ✓ | 执行profile: `final-synthesis`(撰写) / `market-research`(调研) / `logic-review`(评审) |
| content | ✓ | 任务详细描述，包含模块定义+全局约束+输出格式 |
| dependencies | ✗ | 依赖的任务ID列表，有依赖的任务需等依赖完成后再分发 |

---

## kanban-worker

**调用时机**: Phase 3-4 子agent执行
**用途**: 实际执行看板任务，完成模块PRD撰写

### 调用方式

```python
delegate_task(
    goal="基于以下需求撰写[模块名称]的PRD子文档",
    context="""
    ## 模块定义
    [模块名称]: [核心功能]
    输入: [输入数据]
    输出: [输出结果]
    依赖: [依赖模块]

    ## 全局约束
    - 产品名称: [名称]
    - 目标用户: [用户群体]
    - 术语表: [统一术语]

    ## 输出格式
    必须包含: 功能概述/用户故事/功能流程/页面设计/数据模型/API接口/业务规则/异常处理/验收标准
    """,
    toolsets=["file", "web"]  # web仅当需要调研时启用
)
```

---

## claude-design

**调用时机**: Phase 7 UI Demo设计
**用途**: 生成独立HTML格式的UI Demo

### 调用方式

```python
# 调用 claude-design skill 生成单页面Demo
delegate_task(
    goal="基于PRD页面描述生成UI Demo HTML",
    context="""
    页面名称: [名称]
    功能描述: [PRD中的页面描述]
    设计偏好: [品牌色/风格/参考产品]
    技术要求: 零依赖单文件HTML，可直接浏览器打开

    输出路径: /workspace/reports/prd-demos/[页面名].html
    """,
    toolsets=["file"]
)
```

### 输出规范

- 单文件HTML，零外部依赖（CSS/JS内联）
- 响应式布局（适配移动端+桌面端）
- 保存路径: `/workspace/reports/prd-demos/[页面名].html`

---

## popular-web-designs

**调用时机**: Phase 7 UI Demo设计（风格选择阶段）
**用途**: 提供设计系统参考（配色、字体、组件规范）

### 调用方式

```python
# 读取设计系统模板
skill_view(name="popular-web-designs", file_path="templates/linear.app.md")
# 或
skill_view(name="popular-web-designs", file_path="templates/stripe.com.md")
```

### 常用模板

| 模板 | 风格 | 适用场景 |
|------|------|---------|
| linear.app.md | 深色+紫渐变，现代SaaS | B2B工具类产品 |
| stripe.com.md | 简洁+卡片，企业级 | 支付/金融类产品 |
| vercel.com.md | 极简+黑白，开发者 | 技术/开发者产品 |

---

## writing-plans

**调用时机**: PRD完成后，用户需要输出开发实施计划时
**用途**: 将PRD转化为可执行的开发任务计划

### 调用方式

```python
# 在PRD交付后，如用户说"输出开发计划"
skill_view(name="software-development/writing-plans")
# 然后基于PRD内容，使用 writing-plans 的 Implementation Plans 章节生成开发计划
```

### 与 prd-writing 的关系

| 阶段 | 使用 Skill | 输出 |
|------|-----------|------|
| 需求分析 → PRD | prd-writing | PRD文档 |
| PRD → 开发计划 | writing-plans | 实施计划（含代码、测试、任务拆分） |
| UI Demo | claude-design / popular-web-designs | HTML Demo |
| 任务执行 | kanban-orchestrator / kanban-worker | 看板任务 |
