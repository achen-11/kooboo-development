# Kooboo Skill 反馈约束 — 实现计划

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 根据 feedback.md 的 5 类问题，更新 kooboo-ai-plugins 的 references 与 kooboo-api skill，并新增 anti-patterns，使 AI 生成代码时遵守约定。

**Architecture:** 先改 references（api-security、structure、routing、api-core、examples、checklist-common），再新增 anti-patterns.md，最后在 kooboo-api/SKILL.md 增加「编写时务必遵守」小节。设计细节见 `docs/plans/2025-03-02-kooboo-skill-feedback-design.md`。

**Tech Stack:** Markdown 文档，路径均相对于 `skill/kooboo-development/kooboo-ai-plugins/`。

---

### Task 1: api-security.md — 补充 k.security.encode / decode

**Files:**
- Modify: `kooboo-ai-plugins/references/api-security.md`

**Step 1:** 在合适位置（如「加密与编码」相关小节）新增「Token 编解码」小节。

**Step 2:** 写明：
- `k.security.encode(obj)`：传入对象，返回 token 字符串。
- `k.security.decode(token)`：传入 token 字符串；返回对象：
  - 有效：`{ code: 0, value: <encode 时传入的 obj> }`
  - 无效：`{ code: 1, message: 'token invalid' }`  
若已有 JWT 小节，加一句：需要上述约定返回格式时用 encode/decode；标准 JWT 用 `k.security.jwt.*`。

**Step 3:** 保存后确认无错误引用。

---

### Task 2: structure.md — Model 引用与 Services 位置

**Files:**
- Modify: `kooboo-ai-plugins/references/structure.md`

**Step 1:** 全文将 `import { User } from 'code/Models'` 改为「有聚合时用聚合文件路径（如 `code/Models/index`），无聚合时用 `import { User } from 'code/Models/User'`」；禁止使用 `code/Models`（无文件名）。

**Step 2:** 在「code/ — 业务代码」或「数据模型目录」旁增加一句：引用 Model 时禁止写 `code/Models`，须写聚合文件路径或具体 `code/Models/模型名`。

**Step 3:** 在「code/ — 业务代码」中明确：业务逻辑层放在 `code/Services/xxx.ts`（如 `code/Services/auth.ts`）；不得在 `code/` 根下创建 `code/auth.ts` 等；引用时使用 `code/Services/xxx`。

---

### Task 3: routing.md — API 合并与 id 用 query

**Files:**
- Modify: `kooboo-ai-plugins/references/routing.md`

**Step 1:** 在「API 路由」或「路由最佳实践」中增加两条约定：
- 同一模块的多个接口放在一个 api 文件中，使用 `@k-url /api/模块名/{action}`，文件内多个 `k.api.get/post('actionName', ...)`。错误示例：api/order-list.ts、api/order-create.ts；正确：api/order.ts + @k-url /api/order/{action}。
- 类似 id 的单资源标识建议用 query 参数，如 `@k-url /api/order?id={id}`，在 handler 中从 query 取 id；避免 `@k-url /api/order/{id}`。

**Step 2:** 若有按 id 的示例，改为 query 写法并注明「建议 id 用 query」。

---

### Task 4: api-core.md — 同上约定与示例

**Files:**
- Modify: `kooboo-ai-plugins/references/api-core.md`

**Step 1:** 在 API 相关小节增加与 Task 3 一致的两条约定（同一模块一文件、id 用 query）。

**Step 2:** 将示例中的 `import { User } from 'code/Models'` 改为 `code/Models/index` 或 `code/Models/User`（与上下文一致）。

**Step 3:** 若有「按 id 获取」的 API 示例，改为通过 query 传 id。

---

### Task 5: examples.md — Model 与 API 示例

**Files:**
- Modify: `kooboo-ai-plugins/references/examples.md`

**Step 1:** 所有 `code/Models` 改为 `code/Models/index` 或 `code/Models/User`（根据示例是否体现聚合择一）。

**Step 2:** 若有「按 action 拆成多个 api 文件」的示例，改为「同一模块一个 api 文件、多个 action」。

**Step 3:** 若有按 id 的 API 示例，改为 query 传 id，并加简短注释「建议 id 用 query」。

---

### Task 6: checklist-common.md — 检查项与 anti-patterns 引用

**Files:**
- Modify: `kooboo-ai-plugins/references/checklist-common.md`

**Step 1:** 将「Model 引用路径」改为：使用聚合文件路径或 `code/Models/模型名`，禁止 `code/Models`。

**Step 2:** 新增检查项（或合并为一条「见 anti-patterns」）：
- Services 在 `code/Services/`，不在 `code/` 根。
- 同一模块 API 合并在一个文件。
- 单资源 id 使用 query，非路径参数。
- 使用 token 时：`k.security.encode/decode` 及约定返回格式（若适用）。

**Step 3:** 在清单末尾增加：交付前对照 `references/anti-patterns.md` 避免 5 类常见错误。

---

### Task 7: 新增 anti-patterns.md

**Files:**
- Create: `kooboo-ai-plugins/references/anti-patterns.md`

**Step 1:** 标题：Kooboo 常见错误与正确写法。

**Step 2:** 5 条，每条包含「错误」「正确」「说明」：
1. encode/decode：错误用法 vs 正确 `k.security.encode(obj)` / `k.security.decode(token)`；说明 decode 有效时 `value` 为 encode 时传入的 obj。
2. Model 引用：错误 `code/Models`；正确：有聚合用聚合路径（如 code/Models/index），无聚合用 code/Models/User。
3. Services：错误 code/auth.ts；正确 code/Services/auth.ts，引用 code/Services/xxx。
4. API 拆分：错误 多文件 per action；正确 同一模块一个 api 文件、@k-url /api/模块/{action}。
5. id 参数：错误 路径 /{id}；正确 query ?id=。

**Step 3:** 保存后确认各 skill 若引用 anti-patterns 路径正确（如 kooboo-ai-plugins/references/anti-patterns.md）。

---

### Task 8: kooboo-api/SKILL.md — 「编写时务必遵守」

**Files:**
- Modify: `kooboo-ai-plugins/skills/kooboo-api/SKILL.md`

**Step 1:** 在「何时使用」之后新增小节「编写时务必遵守」（或「约束」）。

**Step 2:** 内容：5 条一句话约束（与 anti-patterns 一致），或写「交付前与编写时请对照 `kooboo-ai-plugins/references/anti-patterns.md`」，并列出 5 条标题：encode/decode 用法与返回、Model 引用路径、Services 目录、API 按模块合并、id 用 query。

**Step 3:** 保存后通读 SKILL.md 确保与 references 无冲突。

---

## 执行选项

计划已保存至 `docs/plans/2025-03-02-kooboo-skill-feedback-impl.md`。

**1. 本会话内按任务执行** — 按 Task 1～8 顺序编辑文件，每完成一两项可自检或交你过目。  
**2. 新会话中执行** — 在新会话中打开本仓库，用 executing-plans 按任务批量执行并在节点检查。

你更倾向哪种？
