---
name: kooboo-workflow
description: Kooboo 需求驱动开发工作流。根据用户需求类型（新建页面/开发 API/接入电商/数据模型/功能扩展），自动指引对应的开发路径和步骤。
---

# Kooboo 需求驱动开发工作流

本 Skill 根据用户需求类型，自动匹配最佳开发路径。

---

## Step 0: 识别需求类型

从用户请求中识别开发类型：

| 需求类型 | 特征关键词 | 开发路径 |
|----------|-----------|----------|
| 新建网站/页面 | "新建"、"创建页面"、"做个网站"、"首页"、"落地页" | → 路径 A |
| 开发 API 接口 | "API"、"接口"、"接口对接"、"回调" | → 路径 B |
| 接入电商模块 | "电商"、"商品"、"购物车"、"订单"、"支付" | → 路径 C |
| 设计数据模型 | "数据库"、"数据表"、"模型"、"表结构" | → 路径 D |
| 内容管理需求 | "CMS"、"内容"、"文章"、"新闻"、"富文本" | → 路径 E |
| 安全/加密需求 | "加密"、"签名"、"验签"、"安全" | → 路径 F |
| 模块开发 | "模块"、"复用"、"发布模块" | → 路径 G |
| 功能维护/扩展 | "修改"、"扩展"、"优化"、"Bug" | → 路径 H |

---

## 路径 A: 新建网站/页面

适用场景：创建一个新的 Kooboo 站点或页面

### Step A1: 前端模式选型

| 问题 | 选择 | 指引 |
|------|------|------|
| 页面需要 SEO？ | 是 → SSR | `references/ssr-mode.md` |
| 是后台管理界面？ | 是 → Script Vue | `references/script-vue-mode.md` |
| 是复杂 SPA 应用？ | 是 → Local Vue | `references/local-vue-mode.md` |
| 不确定？ | 查阅 | `references/frontend-modes.md` |

### Step A2: 确定项目结构

- 查阅目录结构 → `references/structure.md`
- 定义路由 → `references/routing.md`

### Step A3: 实现页面

根据选择的前端模式：

| 模式 | 实现文档 |
|------|----------|
| SSR | `references/ssr-mode.md` |
| Script Vue | `references/script-vue-mode.md` |
| Local Vue | `references/local-vue-mode.md` |

### Step A4: 数据绑定（如需）

| 数据类型 | 指引文档 |
|----------|----------|
| 结构化数据（SQLite） | `references/api-database.md` |
| 非结构化内容（CMS） | `references/api-content.md` |

### Step A5: 代码审查

→ `scripts/checklist.md`

---

## 路径 B: 开发 API 接口

适用场景：创建 Kooboo 站点内部 API 或对外接口

### Step B1: 定义接口规范

- HTTP 方法与路由 → `references/api-core.md`
- 请求/响应格式 → `references/api-core.md`

### Step B2: 实现业务逻辑

- Session/Cookie 处理 → `references/api-core.md`
- 状态管理 → `references/api-core.md`

### Step B3: 数据处理

| 数据需求 | 指引文档 |
|----------|----------|
| SQLite 操作 | `references/api-database.md` |
| 电商数据 | `references/api-commerce.md` |
| 内容数据 | `references/api-content.md` |

### Step B4: 安全检查

- 加密/签名 → `references/api-security.md`
- 权限校验 → `references/api-security.md`

### Step B5: 测试与审查

→ `scripts/checklist.md`

---

## 路径 C: 接入电商模块

适用场景：需要商品、购物车、订单、支付等功能

### Step C1: 确认电商功能范围

| 功能 | 指引文档 |
|------|----------|
| 商品管理 | `references/api-commerce.md` |
| 购物车 | `references/api-commerce.md` |
| 订单处理 | `references/api-commerce.md` |
| 支付集成 | `references/api-commerce.md` + `references/api-security.md` |

### Step C2: 数据模型设计

→ `references/api-commerce.md`

### Step C3: 前端集成

根据页面需求选择前端模式，参考路径 A

### Step C4: 测试与审查

→ `scripts/checklist.md`

---

## 路径 D: 设计数据模型

适用场景：需要创建新的 SQLite 数据表或修改表结构

### Step D1: 需求分析

- 数据结构设计
- 字段类型定义

→ `references/api-database.md`

### Step D2: 创建数据表

- k_sqlite ORM 使用 → `references/api-database.md`
- 迁移脚本编写

### Step D3: 验证与审查

→ `scripts/checklist.md`

---

## 路径 E: 内容管理需求

适用场景：需要 CMS 内容管理功能

### Step E1: 确定内容模型

- 内容类型定义
- 字段配置

→ `references/api-content.md`

### Step E2: 实现 CRUD

- 内容增删改查 → `references/api-content.md`
- 富文本编辑 → `references/api-content.md`

### Step E3: 前端展示

根据页面需求选择前端模式，参考路径 A

### Step E4: 审查

→ `scripts/checklist.md`

---

## 路径 F: 安全/加密需求

适用场景：需要加密、签名、权限校验等安全功能

### Step F1: 确定安全需求

| 需求类型 | 指引文档 |
|----------|----------|
| 数据加密 | `references/api-security.md` |
| 接口签名 | `references/api-security.md` |
| 权限控制 | `references/api-security.md` |
| HTTPS/SSL | `references/api-security.md` |

### Step F2: 实现安全功能

→ `references/api-security.md`

### Step F3: 审查

→ `scripts/checklist.md`

---

## 路径 G: 模块开发

适用场景：开发可复用的 Kooboo 模块

### Step G1: 模块设计

→ `references/modules.md`

### Step G2: 开发模块

- 模块结构
- 生命周期
- API 暴露

→ `references/modules.md`

### Step G3: 测试与发布

→ `scripts/checklist.md`

---

## 路径 H: 功能维护/扩展

适用场景：修改现有功能、Bug 修复、性能优化

### Step H1: 需求理解

- 理解现有代码结构
- 定位需要修改的位置

### Step H2: 修改实现

根据修改的功能类型，参考对应路径

### Step H3: 回归测试

→ `scripts/checklist.md`

---

## 通用审查清单

无论哪个路径，完成后都应检查：

→ `scripts/checklist.md`

---

## 快速索引表

| 需求 | 首选路径 | 关键文档 |
|------|----------|----------|
| 新建页面 | A | `frontend-modes.md` |
| 开发 API | B | `api-core.md` |
| 电商功能 | C | `api-commerce.md` |
| 数据库设计 | D | `api-database.md` |
| CMS 内容 | E | `api-content.md` |
| 安全加密 | F | `api-security.md` |
| 模块开发 | G | `modules.md` |
| 功能修改 | H | 视情况而定 |
