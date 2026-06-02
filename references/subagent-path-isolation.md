# Session Reference: Subagent Path Isolation & Direct File Output

## Issue 1: Subagent Cannot Access Skill Directory Templates

### Symptom
When using `delegate_task` to spawn subagents for parallel PRD module writing, subagents fail with:
```
Error: File not found: templates/prd-module-template.md
```

### Root Cause
Subagents run in isolated contexts with their own working directory (typically `/workspace` or a temp dir). They **cannot access** the skill directory at `~/.hermes/skills/prd-writing/templates/` because:
- Skill directories are outside the subagent's filesystem view
- Subagents do not inherit the parent agent's skill loading context
- The `file_path` parameter in `skill_view` resolves relative to the subagent's cwd, not the parent's skill dir

### Reproduction
```python
# In parent agent (works):
skill_view(name="prd-writing", file_path="templates/prd-module-template.md")

# In subagent context (fails):
# Subagent tries to read "templates/prd-module-template.md" from /workspace
# File does not exist there → Error
```

### Workaround (Used Successfully)
**Option A: Inline template in context (Recommended)**
```python
delegate_task(
    goal="撰写模块PRD",
    context="""
    ## 输出格式（11章节标准模板）
    必须严格按以下结构输出：
    ### 1. 功能概述
    - 一句话描述
    - 用户价值
    - 业务目标
    ### 2. 用户故事（表格：编号/作为/想要/以便于/优先级）
    ### 3. 功能流程
    - 主流程（步骤表格）
    - 分支流程
    - 异常流程
    ### 4. 页面/界面设计
    - 页面清单表格
    - 关键页面原型描述（布局/核心元素/空状态/加载状态/错误状态）
    ### 5. 数据模型
    - 表格：字段/类型/必填/默认值/说明
    ### 6. API接口
    - 表格：接口/方法/描述/认证
    ### 7. 业务规则
    - 表格：规则ID/规则描述/触发条件/处理逻辑/优先级
    ### 8. 异常处理
    - 表格：异常ID/场景/触发条件/系统处理/用户提示/日志记录
    ### 9. 验收标准
    - 功能验收（表格）
    - 非功能验收（表格）
    ### 10. 依赖与风险
    - 依赖模块表格
    - 风险矩阵（可能性/影响/缓解措施）
    ### 11. 附录
    - 术语表
    - 变更日志
    """
)
```

**Option B: Parent agent writes directly (Fallback)**
When subagent fails after 1 retry, parent agent writes the module directly. This was the approach used in the MayGrove PRD v2 session — parent agent wrote all 7 modules sequentially after subagent failed.

**Option C: Copy template to workspace before dispatch**
```python
# Parent agent copies template to workspace
cp ~/.hermes/skills/prd-writing/templates/prd-module-template.md /workspace/prd-template.md
# Subagent can now read /workspace/prd-template.md
```

### Recommendation
Update SKILL.md Phase 3.2 to always use Option A (inline context) as primary, with Option B as fallback. Option C adds filesystem side effects that may not be desirable.

---

## Issue 2: User Preference for Direct File Output

### Signal
User said: "功能模块输出为md文档"

### Interpretation
User wants the deliverable written directly to a `.md` file on disk, **not** presented as inline markdown in the chat response. This is a common preference for:
- Large documents (>50KB) that are unwieldy in chat
- Formal deliverables that need to be saved, shared, or version-controlled
- Users who prefer file-based workflows over chat-based consumption

### Correct Response Pattern
```
# BAD: Inline full document in chat
> Here's the PRD:
> # Product Requirements Document
> ... (2000 lines of markdown) ...

# GOOD: Write to file, summarize in chat
write_file(path="/workspace/reports/product-prd.md", content="...")
> PRD已写入 `/workspace/reports/product-prd.md`
> - 行数: 1,886 | 大小: 98KB | 模块数: 7
> - 关键变更: ①种子分失效机制调整 ②经验值重新定义
> - 待确认问题: 3项（见文档§9.1）
```

### When to Apply
Apply direct file output when:
- User explicitly says "输出为md/文档/文件"
- User says "不用展示" / "直接写文件"
- Document is expected to be >500 lines or >30KB
- The deliverable is a formal document (PRD, spec, report) rather than a quick answer

### When NOT to Apply
- Small snippets (<50 lines) — inline is fine
- Interactive clarification questions
- Code review comments
- Quick lookups or summaries

---

## Session Log: MayGrove PRD v2.1

| Event | Action | Result |
|-------|--------|--------|
| Subagent dispatched for module writing | `delegate_task` with `file_path="templates/prd-module-template.md"` | Failed: file not found |
| Fallback activated | Parent agent wrote all 7 modules directly | Success: 1,886 lines, 98KB |
| User requested md output | `write_file` to `/workspace/reports/...` | Success |
| User requested currency system changes | Patched 8 sections across 4 modules | Success |

### Files Modified in Session
- `~/.hermes/skills/prd-writing/SKILL.md` — Patched 3 locations
- `~/.hermes/skills/prd-writing/references/subagent-path-isolation.md` — This file
- `/workspace/reports/maygrove-subscription-prd-v2-zh.md` — v2.0 → v2.1
