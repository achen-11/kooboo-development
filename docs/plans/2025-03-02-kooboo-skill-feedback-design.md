# Kooboo Skill 反馈约束 — 设计说明

根据测试反馈，在 plugin 的 references 与 skill 中补充 5 类约束，避免 AI 重复同类错误。

---

## 修正说明（用户确认）

- **问题 1**：`k.security.decode(token)` 有效时返回 `{ code: 0, value: <encode 时传入的 obj> }`，即 **value 即为 encode 时传入的对象**。
- **问题 2**：Model 引用**禁止**使用 `code/Models`（无文件名）。正确二选一：**有聚合**则用聚合文件路径（如 `code/Models/index`，以实际聚合文件名为准）；**无聚合**则直接引用具体模型文件，如 `import { User } from 'code/Models/User'`。
- **问题 4**：示例保持「同一模块 API 合并在一个文件」；若涉及聚合/单文件说明，与问题 2 同理区分。

---

## 1. 问题与落点

| 问题 | 主要修改文件 | 修改要点 |
|------|--------------|----------|
| 1. k.security.encode/decode | api-security.md | 补充 encode(obj) / decode(token)；decode 返回格式：有效 `{code:0, value: encode 时的 obj}`，无效 `{code:1, message:'token invalid'}` |
| 2. Models 引用路径 | structure.md, api-core.md, examples.md, checklist-common | 禁止 code/Models；有聚合用聚合路径，无聚合用 code/Models/User 等 |
| 3. Services 位置 | structure.md | 业务逻辑在 code/Services/xxx.ts，禁止 code/xxx.ts |
| 4. API 按模块合并 | routing.md, api-core.md, examples.md | 同一模块一个 api 文件，@k-url /api/模块/{action}，多 action 同文件 |
| 5. id 用 query | routing.md, api-core.md / examples | 单资源 id 建议 query ?id=，避免 /{id} |

---

## 2. 具体修改

### 2.1 api-security.md
- 新增「Token 编解码（k.security.encode / decode）」：`k.security.encode(obj)` 返回 token 字符串；`k.security.decode(token)` 解析后：有效 `{ code: 0, value: 原 obj }`，无效 `{ code: 1, message: 'token invalid' }`。与 JWT 小节并存时注明适用场景。

### 2.2 structure.md
- Model 引用：全文改为「有聚合用聚合文件路径（如 code/Models/index），无聚合用 code/Models/User」；禁止写 `code/Models`。
- code/ 小节：明确业务逻辑在 `code/Services/xxx.ts`，不得在 `code/` 根下建业务服务文件；引用用 `code/Services/xxx`。

### 2.3 routing.md + api-core.md
- 约定：同一模块多个接口放在一个 api 文件，使用 @k-url /api/模块名/{action}。
- 约定：类似 id 的单资源标识建议 query（如 /api/order?id=），避免路径 /api/order/{id}；示例改为 query 取 id。

### 2.4 examples.md
- Model 引用改为聚合路径或 code/Models/User，不得出现 code/Models。
- API 示例：同一模块一个文件、多 action；单资源用 query id。

### 2.5 checklist-common.md
- Model 项改为「使用聚合路径或 code/Models/模型名，禁止 code/Models」。
- 新增或引用：Services 在 code/Services/、API 按模块合并、id 用 query、token 用 encode/decode 及返回格式。

### 2.6 新增 anti-patterns.md
- 5 条「错误 vs 正确 vs 说明」，含上述两点修正（decode 的 value、Model 两种正确写法）。

### 2.7 kooboo-api/SKILL.md
- 增加「编写时务必遵守」：5 条一句话约束或指向 anti-patterns，并列出 5 条标题。

---

## 3. 实施顺序

1. 修改 references（api-security, structure, routing, api-core, examples, checklist-common）。
2. 新增 references/anti-patterns.md。
3. 修改 kooboo-api/SKILL.md。
