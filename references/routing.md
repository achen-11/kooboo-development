# Kooboo 路由系统

## 路由定义

### 基本语法

在 HTML 文件顶部使用注释定义路由：

```html
<!-- @k-url / -->
<!-- @k-url /about -->
<!-- @k-url /products -->
```

路由定义必须在 HTML 文件的第一行或前几行。

### 动态路由

使用 `{}` 定义动态参数：

```html
<!-- @k-url /products/{category} -->
<!-- @k-url /product/{productId} -->
<!-- @k-url /user/{userId}/profile -->
```

动态参数可通过 `k.request.queryString` 或 `k.request.get()` 获取：

```javascript
// URL: /product/123
const productId = k.request.queryString.productId
// 或
const productId = k.request.get('productId')
```

### 路由约束

| 约束类型 | 语法 | 示例 |
|----------|------|------|
| 数字 | `:number` | `/product/{id:number}` |
| 字母 | `:alpha` | `/code/{code:alpha}` |
| 字母数字 | `:alphanum` | `/tag/{tag:alphanum}` |
| 通配 | `*` | `/files/{*path}` |

---

## 页面与路由

### page/ 目录

页面文件放在 `page/` 目录下：

```
page/
├── index.html          # → /
├── about.html          # → /about
├── products.html       # → /products
├── product/
│   └── {productId}.html  # → /product/{productId}
└── user/
    └── {userId}/
        └── profile.html  # → /user/{userId}/profile
```

### 路由查找规则

1. Kooboo 首先查找精确匹配
2. 然后查找动态路由（按定义顺序）
3. 最后查找通配路由

---

## 视图与布局

### 引用视图

```html
<view id="common.header"></view>
<view id="components.product-card"></view>
<view id="homepage.hero-banner"></view>
```

### 使用布局

```html
<layout id="app.main">
    <placeholder id="Main">
        <!-- 页面内容 -->
    </placeholder>
</layout>
```

### 布局传参

```html
<!-- 布局中定义 placeholder -->
<placeholder id="Title">
    <slot name="default">
        <h1>默认标题</h1>
    </slot>
</placeholder>

<!-- page 中使用 -->
<layout id="app.main">
    <placeholder id="Title">
        <h1 slot="default">自定义标题</h1>
    </placeholder>
    <placeholder id="Main">
        <!-- 内容 -->
    </placeholder>
</layout>
```

---

## 视图属性传递

### 定义可传参的视图

```html
<!-- view 定义 -->
<view id="components.product-card">
    <template k-slot-define:product="{ product }">
        <div class="card">
            <h3>{{ product.name }}</h3>
            <p>{{ product.price }}</p>
        </div>
    </template>
</view>
```

### 使用时传参

```html
<!-- page 中使用并传参 -->
<view id="components.product-card">
    <template k-slot-insert:product>
        {
            "name": "iPhone 15",
            "price": 999
        }
    </template>
</view>
```

---

## 路由与 API

### 页面路由 vs API 路由

- **页面路由**：`<!-- @k-url /path -->` 定义页面 URL
- **API 路由**：在 `api/*.ts` 文件中使用 `k.api.get('/path', ...)`，**并且必须**在文件顶部添加 `// @k-url /api/xxx/{action}` 声明

### API 路由示例

```typescript
// @k-url /api/product/{action}
// api/products.ts
import { Product } from 'code/Models'

k.api.get('products', () => {
    const products = Product.findAll({})
    return { success: true, data: products }
})

k.api.get('product-detail', () => {
    const id = k.request.queryString.id
    const product = Product.findById(id)
    return { success: true, data: product }
})

// 访问：/api/products/products
// 访问：/api/products/product-detail?id=1
```

### ⚠️ 重要：API 文件必须添加 @k-url 声明

```typescript
// ✅ 正确：必须添加 @k-url 声明
// @k-url /api/user/{action}
import { User } from 'code/Models'
k.api.get('info', () => { ... })

// ❌ 错误：缺少 @k-url 声明
import { User } from 'code/Models'
k.api.get('info', () => { ... })
```

---

## 路由重定向

### 服务端重定向

```javascript
k.response.redirect('/new-path')
```

### 条件重定向

```javascript
// 未登录重定向到登录页
if (!k.account.isLogin) {
    k.response.redirect('/login?returnurl=' + k.request.url)
    return
}
```

---

## 路由守卫

### 服务端路由守卫

```html
<script env="server" type="module">
    // 每个页面加载时都会执行
    if (!k.account.isLogin && k.request.url !== '/login') {
        k.response.redirect('/login?returnurl=' + k.request.url)
    }
</script>
```

---

## 路由最佳实践

1. **路由命名**：使用小写字母和连字符
   - ✅ `/product-detail`、`/user-profile`
   - ❌ `/ProductDetail`、`/User_Profile`

2. **动态参数**：参数名使用有意义的名称
   - ✅ `/order/{orderId}`
   - ❌ `/order/{id}`

3. **路由顺序**：静态路由放在动态路由前面
   ```html
   <!-- @k-url /products -->
   <!-- @k-url /products/{category} -->
   ```

4. **SEO**：需要 SEO 的页面使用 SSR 模式，不需要的使用 Script Vue

5. **404 处理**：创建 `page/404.html` 处理未匹配路由
