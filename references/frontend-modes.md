# Kooboo 三种前端模式

Kooboo 支持三种前端开发模式，选择取决于项目需求：

| 模式 | 渲染方式 | SEO | 适用场景 |
|------|----------|-----|----------|
| **SSR** | 服务端渲染 | ✅ 优 | 公开页面、博客、电商产品页 |
| **Script Vue** | 嵌入 Vue | ❌ 差 | 后台管理、快速开发 |
| **Local Vue** | 独立构建 | ❌ 差 | 复杂 SPA、团队协作 |

---

## 1. SSR（服务端渲染）

服务端执行脚本，数据在服务端获取，HTML 完全渲染后返回给浏览器。

### 特点

- **SEO 友好**：搜索引擎可直接抓取完整 HTML
- **首屏快**：服务端渲染，首屏无需等待 JS
- **数据安全**：敏感逻辑可在服务端执行
- **SEO 需求**：产品页、博客、首页等需要搜索引擎收录的页面

### 代码结构

```html
<!-- page/products.html -->
<!-- @k-url /products -->

<!-- 服务端脚本 -->
<script env="server" type="module">
    // 服务端执行，可访问 k 对象
    const products = k.DB.sqlite.products.all()
    const categories = k.commerce.category.all()

    // 传递给客户端
    k.utils.clientJS.setVariable('products', products)
    k.utils.clientJS.setVariable('categories', categories)
</script>

<!-- 模板渲染 -->
<div class="products">
    <h1>产品列表</h1>
    <div v-for="product in products">
        <h2>{{ product.name }}</h2>
        <p>{{ product.price }}</p>
    </div>
</div>
```

### 数据传递

```javascript
// 服务端设置
k.state.set('key', value)           // 供模板使用
k.utils.clientJS.setVariable('key', value)  // 供客户端 JS 使用

// 客户端获取
const data = window.key             // 通过全局变量获取
```

### 适用场景

- 公开网站首页
- 产品详情页
- 博客文章页
- 需要 SEO 的页面
- 内容展示为主的页面

### 项目示例

- `x-music/` - 乐器商城
- `YJL-Custom/` - POD 电商
- `Menoki/` - 贴纸商城

---

## 2. Script Vue（嵌入 Vue）

在 Kooboo 页面中通过 script 标签引入 Vue.js，使用 Vue 的声明式语法开发交互界面。

### 特点

- **快速开发**：无需构建，直接引入 Vue
- **交互丰富**：使用 Vue 生态
- **无 SEO**：客户端渲染，搜索引擎无法抓取
- **适合后台**：内部管理系统

### 代码结构

```html
<!-- page/admin/dashboard.html -->
<!-- @k-url /admin -->

<!-- 引入 Vue 和组件库 -->
<script src="/js/vue.global.js"></script>
<script src="/js/element-plus/index.js"></script>
<script src="/js/http.js"></script>

<!-- 服务端预取数据 -->
<script env="server" type="module">
    // 获取初始数据
    const stats = k.DB.sqlite.orders.count()
    k.utils.clientJS.setVariable('stats', stats)
</script>

<!-- 客户端 Vue 应用 -->
<script>
    const { createApp, ref, onMounted } = Vue

    const App = {
        setup() {
            const orders = ref(window.stats || [])

            onMounted(() => {
                // 客户端逻辑
                fetchOrders()
            })

            return { orders }
        },
        template: `
            <div>
                <h1>订单管理</h1>
                <el-table :data="orders">
                    <el-table-column prop="id" label="订单号"></el-table-column>
                    <el-table-column prop="total" label="金额"></el-table-column>
                </el-table>
            </div>
        `
    }

    createApp(App).use(ElementPlus).mount('#app')
</script>

<div id="app"></div>
```

### definePage 模式

使用 `definePage` 定义页面结构：

```javascript
// 定义页面配置
definePage({
    path: '/admin/products',
    title: '产品管理',
    layout: 'admin',
    components: {
        'product-list': ProductListComponent,
        'product-form': ProductFormComponent
    },
    data: {
        // 预加载数据
        products: 'api/products'
    }
})
```

### 适用场景

- 后台管理界面
- 数据表格页面
- 表单页面
- 内部系统
- 快速原型开发

### 项目示例

- `Kooboo-Template/cdnVueTemplate/` - CDN Vue 后台模板

---

## 3. Local Vue（独立应用）

独立的 Vue 3 + Vite 项目，通过 HTTP API 与 Kooboo 交互。

### 特点

- **完整构建**：使用 Vite 构建，开发者体验最佳
- **现代框架**：Vue 3 + Composition API + TypeScript
- **团队协作**：适合大型团队，代码分离
- **独立部署**：可以部署到任意静态服务器或 CDN

### 项目结构

```
my-vue-app/
├── src/
│   ├── views/
│   │   ├── HomeView.vue
│   │   └── ProductView.vue
│   ├── components/
│   ├── api/
│   │   └── index.ts       # API 封装
│   ├── stores/
│   │   └── product.ts     # Pinia store
│   ├── router/
│   │   └── index.ts
│   └── main.ts
├── index.html
├── vite.config.ts
└── package.json
```

### 与 Kooboo 交互

#### 方式一：通过 k.api 定义端点

```typescript
// Kooboo 端：api/products.ts
k.api.get('products', () => {
    return k.DB.sqlite.products.all()
})

// Vue 端：api/index.ts
export const getProducts = () => fetch('/api/products/products').then(r => r.json())
```

#### 方式二：通过 k.net 服务端代理

```typescript
// Kooboo 端：api/external.ts
k.api.get('fetch-products', () => {
    const response = k.net.url.getJson('https://api.example.com/products')
    return response
})

// Vue 端直接调用自己的 API
export const getProducts = () => fetch('/api/products/fetch-products').then(r => r.json())
```

### Vue 组件示例

```vue
<!-- src/views/ProductsView.vue -->
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useProductStore } from '../stores/product'

const store = useProductStore()

onMounted(() => {
    store.fetchProducts()
})
</script>

<template>
    <div class="products">
        <h1>产品列表</h1>
        <div v-for="product in store.products" :key="product.id">
            {{ product.name }}
        </div>
    </div>
</template>
```

### 适用场景

- 复杂单页应用
- 需要高级交互
- 大型团队项目
- 需要 CI/CD 流程
- 移动端优先的应用

### 项目示例

- `erp/front-end/` - ERP 系统前端
- `task-banner/frontend/` - 任务管理前端

---

## 模式对比与选型

### 选型决策树

```
需要 SEO？
    │
    ├─ 是 → SSR
    │
    └─ 否 → 项目规模？
              │
              ├─ 小/快 → Script Vue
              │
              └─ 大/复杂 → Local Vue
```

### 对比表

| 特性 | SSR | Script Vue | Local Vue |
|------|-----|------------|-----------|
| SEO | ✅ | ❌ | ❌ |
| 首屏速度 | 快 | 中 | 懒加载 |
| 开发体验 | 中 | 中 | 优 |
| 构建工具 | 无 | 无 | Vite |
| 团队协作 | 差 | 中 | 优 |
| 部署方式 | Kooboo | Kooboo | 任意 |
| 代码分离 | 差 | 中 | 优 |
| TypeScript | 可选 | 可选 | 推荐 |

### 混合使用

实际项目中可以混合使用：

- **公开页面**：SSR（首页、产品页）
- **后台**：Script Vue 或 Local Vue
- **管理后台**：Local Vue

```html
<!-- 公开页：SSR -->
<!-- @k-url / -->

<!-- 后台：Script Vue -->
<!-- @k-url /admin -->
<script src="/js/vue.global.js"></script>
```

---

## 开发工作流

### SSR 开发

1. 创建 `page/*.html`
2. 添加 `<!-- @k-url /path -->`
3. 编写 `<script env="server">` 获取数据
4. 使用模板语法渲染
5. `kb sync` 同步到服务器

### Script Vue 开发

1. 引入 Vue 和组件库
2. 编写 Vue 应用代码
3. 调试运行
4. `kb sync` 同步

### Local Vue 开发

1. 使用 `localVueTemplate` 创建项目
2. `npm run dev` 本地开发
3. `npm run build` 构建
4. 构建产物放入 Kooboo 或单独部署

---

## 模板参考

- SSR 模板：`Kooboo-Template/SSR_Template/`
- Script Vue 模板：`Kooboo-Template/cdnVueTemplate/`
- Local Vue 模板：`Kooboo-Template/localVueTemplate/`
