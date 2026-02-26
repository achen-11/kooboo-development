---
name: kooboo-development
description: Kooboo CMS 全栈开发指南。用于开发 Kooboo 网站、CMS 模块、电商功能、SSR 页面、Script Vue 后台、Local Vue 应用。提供项目结构、三种前端模式选型、核心 API 参考、数据库操作、电商功能、内容管理、安全加密、模块开发、代码审查清单。开发 Kooboo 站点、新建页面、编写 API、接入电商模块、处理数据库、设计数据模型时请使用此 Skill。
---

# Kooboo 开发指南

本 Skill 提供 Kooboo CMS 全栈开发的完整指导。使用 references 分层结构，按需查阅对应章节。

---

## 使用方式

### Step 1: 明确需求

从用户请求中提取关键信息：
- **前端模式**：SSR（需 SEO）/ Script Vue（后台快速开发）/ Local Vue（复杂 SPA）
- **功能类型**：页面展示 / API 接口 / 电商功能 / 内容管理 / 数据模型
- **是否需要数据库**：是 → 查阅数据库 API；否 → 直接写页面

### Step 2: 选型与结构

- 新项目不知选哪种模式 → 查阅 `references/frontend-modes.md`
- 不确定目录结构 → 查阅 `references/structure.md`
- 路由如何定义 → 查阅 `references/routing.md`

### Step 3: 实现功能

根据功能类型查阅对应 API 参考：

| 功能类型 | 查阅文件 |
|----------|----------|
| HTTP 请求/响应、会话、Cookie | `references/api-core.md` |
| SQLite 数据库、数据模型 | `references/api-database.md` |
| 商品、购物车、订单、分类 | `references/api-commerce.md` |
| CMS 内容增删改查 | `references/api-content.md` |
| 加密、网络请求、邮件发送 | `references/api-security.md` |
| 开发可复用模块 | `references/modules.md` |

### Step 4: 代码示例

需要常用代码模式时，查阅 `references/examples.md` 指向的 `data/code-examples.json`。

### Step 5: 代码审查

交付前对照 `scripts/checklist.md` 进行自查。

---

## 快速索引

### 项目结构
- 目录树与各目录职责 → `references/structure.md`
- 路由定义与动态参数 → `references/routing.md`

### 前端模式选型
- 三种模式对比与适用场景 → `references/frontend-modes.md`

### API 参考
- request / response / session / cookie / state / label → `references/api-core.md`
- SQLite / k_sqlite ORM → `references/api-database.md`
- commerce / product / cart / order → `references/api-commerce.md`
- content / 内容管理 → `references/api-content.md`
- security / net / mail → `references/api-security.md`

### 模块系统
- 模块开发与安装 → `references/modules.md`

### 最佳实践
- 来自 KB_DOC 的实践指南 → `references/best-practices.md`

### 代码示例
- 常用模式速查 → `references/examples.md`

### 代码审查
- 交付前检查清单 → `scripts/checklist.md`

---

## 项目模板参考

- **SSR 网站模板**：`Kooboo-Template/SSR_Template`
- **Script Vue 后台**：`Kooboo-Template/cdnVueTemplate`
- **Local Vue 应用**：`Kooboo-Template/localVueTemplate`

---

## Kooboo 核心概念

Kooboo 是支持三种前端模式的 CMS/开发平台：

1. **SSR** - 服务端渲染，适合需要 SEO 的公开页面
2. **Script Vue** - 页面内嵌入 Vue，适合后台管理界面
3. **Local Vue** - 独立 Vue 3 + Vite 项目，通过 API 与 Kooboo 交互

详细对比与选型建议见 `references/frontend-modes.md`。

---

## 常用参考

- Kooboo 官方文档站：`kooboo-dev-guide/docs`
- Obsidian 知识库：`KB_DOC`（本地）
- Kooboo 核心库：`Kooboo/`
- TypeScript 类型定义：项目根目录 `kooboo.d.ts`
