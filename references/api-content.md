# Kooboo 内容管理 API

Kooboo 内容管理（Content）模块用于管理非结构化内容，如博客文章、新闻、产品描述等。

---

## 基本概念

### 内容类型

内容管理基于**内容类型**（Content Type），每种类型有自定义字段：

```javascript
//假设有一个  "blog" 内容类型
const posts = k.content.blog.all()

// 假设有一个 "news" 内容类型
const news = k.content.news.all()

// 假设有一个 "homepage" 内容类型
const homepage = k.content.Homepage.get()
```

### 内容字段

内容字段在 Kooboo 后台定义，支持类型：
- 文本（Text）
- 富文本（RichText）
- 图片（Media）
- 数字（Number）
- 布尔（Boolean）
- 日期（DateTime）
- 引用（Reference）

---

## 查询内容

### 获取所有内容

```javascript
// 获取类型下所有内容
const posts = k.content.blog.all()

// 带分页
const result = k.content.blog.findPaginated({}, { page: 1, pageSize: 10 })
```

### 按条件查询

```javascript
// 简单条件
const posts = k.content.blog.find("title = 'Featured'")

// 多条件
const posts = k.content.blog.find("category = 'news' AND isPublished = true")

// 使用 findAll
const posts = k.content.blog.findAll({ category: 'news' })
```

### 使用操作符

```javascript
const ops = k.content.operators()

// 大于
const posts = k.content.blog.find({
    date: { [ops.GT]: '2024-01-01' }
})

// 小于等于
const posts = k.content.blog.find({
    views: { [ops.LTE]: 1000 }
})

// 包含
const posts = k.content.blog.find({
    title: { [ops.LIKE]: '%Kooboo%' }
})

// 在范围内
const posts = k.content.blog.find({
    category: { [ops.IN]: ['news', 'tech', 'life'] }
})
```

### 操作符列表

| 操作符 | 说明 | 示例 |
|--------|------|------|
| `eq` | 等于 | `{ [ops.EQ]: 'value' }` |
| `ne` | 不等于 | `{ [ops.NE]: 'value' }` |
| `gt` | 大于 | `{ [ops.GT]: 100 }` |
| `gte` | 大于等于 | `{ [ops.GTE]: 100 }` |
| `lt` | 小于 | `{ [ops.LT]: 100 }` |
| `lte` | 小于等于 | `{ [ops.LTE]: 100 }` |
| `like` | 包含 | `{ [ops.LIKE]: '%word%' }` |
| `in` | 在数组中 | `{ [ops.IN]: ['a', 'b'] }` |
| `between` | 范围 | `{ [ops.BETWEEN]: [min, max] }` |

### 获取单条内容

```javascript
// 按 ID 获取
const post = k.content.blog.get(id)

// 按字段获取
const post = k.content.blog.findOne({ slug: 'my-first-post' })
```

---

## 创建内容

```javascript
// 创建内容
const newPost = k.content.blog.add({
    title: '我的第一篇博客',
    content: '这是博客内容...',
    category: 'tech',
    author: 'John',
    isPublished: true,
    date: new Date()
})
```

---

## 更新内容

```javascript
// 更新内容
k.content.blog.update({
    id: postId,
    title: '更新的标题',
    content: '更新后的内容...'
})
```

---

## 删除内容

```javascript
// 按 ID 删除
k.content.blog.delete(postId)

// 按条件删除
k.content.blog.deleteWhere({ isPublished: false })
```

---

## 排序与分页

```javascript
// 排序
const posts = k.content.blog.findAll({}, {
    order: { field: 'date', direction: 'desc' }
})

// 分页
const result = k.content.blog.findAll({}, {
    page: 1,
    pageSize: 20
})

// 返回格式
// {
//   list: [...],
//   total: 100,
//   page: 1,
//   pageSize: 20
// }
```

---

## 字段选择

```javascript
// 只返回指定字段
const posts = k.content.blog.findAll({}, {
    fields: ['id', 'title', 'date']
})

// 排除某些字段
const posts = k.content.blog.findAll({}, {
    exclude: ['content', 'draft']
})
```

---

## 获取关联内容

```javascript
// 假设 "post" 类型有一个 "category" 引用字段
const post = k.content.blog.get(postId)

// 获取关联的分类
const category = k.content.category.get(post.categoryId)

// 获取文章的所有评论（假设有引用）
const comments = k.content.comments.find({ postId: post.id })
```

---

## 完整示例：博客列表 API

```typescript
// api/blog-list.ts
k.api.get('blog-list', () => {
    const { page = 1, pageSize = 10, category, keyword } = k.request.queryString

    // 构建查询条件
    const conditions = {}
    if (category) {
        conditions.category = category
    }
    if (keyword) {
        conditions.title = { [k.content.operators().LIKE]: `%${keyword}%` }
    }

    // 查询
    const result = k.content.blog.findAll(conditions, {
        order: { field: 'date', direction: 'desc' },
        page: parseInt(page),
        pageSize: parseInt(pageSize),
        fields: ['id', 'title', 'summary', 'coverImage', 'date', 'category']
    })

    return result
})
```

---

## 完整示例：博客详情 API

```typescript
// api/blog-detail.ts
k.api.get('blog-detail', () => {
    const { id } = k.request.queryString

    if (!id) {
        k.response.statusCode(400)
        return { success: false, message: '缺少 id 参数' }
    }

    const post = k.content.blog.get(id)

    if (!post) {
        k.response.statusCode(404)
        return { success: false, message: '文章不存在' }
    }

    // 增加浏览量（假设有 views 字段）
    k.content.blog.update({
        id: post.id,
        views: (post.views || 0) + 1
    })

    return { success: true, data: post }
})
```

---

## 内容类型与数据库的区别

| 特性 | 内容管理 (k.content) | 数据库 (k.DB / k_sqlite) |
|------|---------------------|--------------------------|
| 用途 | 非结构化内容 | 结构化业务数据 |
| 字段定义 | Kooboo 后台配置 | 代码定义模型 |
| 灵活性 | 高 | 中 |
| 查询能力 | 强 | 强 |
| 适用场景 | 文章、新闻、页面 | 订单、用户、库存 |

---

## 快速索引

| 操作 | API |
|------|-----|
| 获取所有 | `k.content.type.all()` |
| 条件查询 | `k.content.type.find(conditions)` |
| 分页查询 | `k.content.type.findPaginated(cond, options)` |
| 单条查询 | `k.content.type.get(id)` |
| 创建 | `k.content.type.add(data)` |
| 更新 | `k.content.type.update(data)` |
| 删除 | `k.content.type.delete(id)` |
| 操作符 | `k.content.operators()` |
