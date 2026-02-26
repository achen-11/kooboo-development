# Kooboo 数据库 API

Kooboo 提供两种数据库操作方式：
1. **k.DB.sqlite** - Kooboo 内置 SQLite
2. **k_sqlite 模块** - 更强大的 ORM（推荐）

---

## k.DB.sqlite — 内置 SQLite

### 基本查询

```javascript
// 原生 SQL 查询
const results = k.DB.sqlite.query('SELECT * FROM products WHERE price > ?', [100])

// 查询所有
const products = k.DB.sqlite.products.all()

// 按条件查询
const products = k.DB.sqlite.products.findAll({
    category: 'electronics',
    price: { $gt: 100 }
})

// 单条查询
const product = k.DB.sqlite.products.findOne({ id: 1 })
```

### 操作符

```javascript
const ops = k.DB.sqlite.operators()

// 比较操作符
const products = k.DB.sqlite.products.findAll({
    price: { [ops.GT]: 100 },      // >
    price: { [ops.GTE]: 100 },     // >=
    price: { [ops.LT]: 1000 },     // <
    price: { [ops.LTE]: 1000 },    // <=
    name: { [ops.NE]: 'Admin' }    // !=
})

// 模糊查询
const products = k.DB.sqlite.products.findAll({
    name: { [ops.LIKE]: '%phone%' }
})

// 范围查询
const products = k.DB.sqlite.products.findAll({
    price: { [ops.BETWEEN]: [100, 1000] }
})

// IN 查询
const products = k.DB.sqlite.products.findAll({
    category: { [ops.IN]: ['electronics', 'books', 'clothing'] }
})
```

### 插入/更新/删除

```javascript
// 插入
const id = k.DB.sqlite.products.add({
    name: 'iPhone 15',
    price: 999,
    category: 'electronics'
})

// 更新
k.DB.sqlite.products.update({
    id: 1,
    name: 'iPhone 15 Pro',
    price: 1199
})

// 删除
k.DB.sqlite.products.delete(1)

// 条件删除
k.DB.sqlite.products.deleteWhere({ category: 'deprecated' })
```

---

## k_sqlite 模块 — ORM（推荐）

需要先安装 `k_sqlite` 模块，然后导入使用：

```typescript
import { ksql, DataTypes } from 'module/k_sqlite'
```

### 定义模型

```typescript
const User = ksql.define(
    'users',
    {
        userName: { 
            type: DataTypes.String, 
            required: true 
        },
        email: { 
            type: DataTypes.String, 
            unique: true,
            index: true
        },
        password: { 
            type: DataTypes.String, 
            required: true 
        },
        age: { 
            type: DataTypes.Number 
        },
        isActive: { 
            type: DataTypes.Boolean, 
            default: true 
        },
        profile: {
            type: DataTypes.Object
        }
    },
    {
        timestamps: true,    // 自动添加 createdAt, updatedAt
        softDelete: true    // 启用软删除 isDeleted, deletedAt
    }
)
```

### 数据类型

| 类型 | 说明 | 对应 SQLite |
|------|------|-------------|
| `DataTypes.String` | 字符串 | TEXT |
| `DataTypes.Number` | 数字 | INTEGER |
| `DataTypes.Boolean` | 布尔 | INTEGER (0/1) |
| `DataTypes.Float` | 浮点数 | REAL |
| `DataTypes.Array` | 数组 | TEXT (JSON) |
| `DataTypes.Object` | 对象 | TEXT (JSON) |
| `DataTypes.Timestamp` | 时间戳 | INTEGER |

### 字段选项

```typescript
const Product = ksql.define('products', {
    name: {
        type: DataTypes.String,
        required: true,        // 非空
        default: '未命名'       // 默认值
    },
    code: {
        type: DataTypes.String,
        unique: true,          // 唯一约束
        index: true            // 索引
    },
    price: {
        type: DataTypes.Number,
        default: 0
    },
    category: {
        type: DataTypes.String,
        ref: 'categories.id'   // 外键引用
    }
}, { timestamps: true })
```

---

## CRUD 操作

### 创建

```typescript
// 创建单条
const id = await User.create({
    userName: 'john',
    email: 'john@example.com',
    password: 'hashed_password'
})

// 条件创建（不存在才创建）
const id = await User.createIfNotExists(
    { email: 'john@example.com' },
    { userName: 'john', password: 'hashed' }
)
```

### 查询

```typescript
// 查询单条
const user = await User.findOne({ email: 'john@example.com' })
const user = await User.findById(id)

// 查询多条
const users = await User.findAll({ isActive: true })

// 分页查询
const result = await User.findPaginated(
    { isActive: true },
    { 
        select: ['id', 'userName', 'email'],  // 指定字段
        order: { prop: 'createdAt', order: 'descending' },
        page: 1,
        pageSize: 20
    }
)
// 返回: { list: [...], page: 1, pageSize: 20, total: 100 }
```

### 更新

```typescript
// 更新单条
const id = await User.updateOne(
    { email: 'john@example.com' },
    { userName: 'john_updated' }
)

// 按 ID 更新
const id = await User.updateById(id, { userName: 'new_name' })

// 批量更新
const ids = await User.updateMany(
    { isActive: false },
    { status: 'inactive' }
)

// 保存或创建
const id = await User.updateOrCreate(
    { userName: 'john' },
    { email: 'john@example.com', password: 'xxx' }
)
```

### 删除

```typescript
// 物理删除
await User.deleteById(id)
await User.deleteOne({ email: 'john@example.com' })
const count = await User.deleteMany({ isActive: false })

// 软删除（推荐）
await User.removeById(id)
await User.removeOne({ email: 'john@example.com' })
const count = await User.removeMany({ isActive: false })

// 恢复软删除
await User.restoreByIds([id1, id2])
```

---

## 查询选项

### select - 字段选择

```typescript
const users = await User.findAll({}, {
    select: ['id', 'userName', 'email']
})
```

### exclude - 排除字段

```typescript
const users = await User.findAll({}, {
    exclude: ['password', 'token']
})
```

### order - 排序

```typescript
// 升序
const users = await User.findAll({}, {
    order: 'userName'
})

// 降序
const users = await User.findAll({}, {
    order: { prop: 'createdAt', order: 'descending' }
})

// 随机
const users = await User.findAll({}, {
    order: 'RANDOM()'
})
```

### includeDeleted - 包含已删除

```typescript
const users = await User.findAll({}, {
    includeDeleted: true
})
```

---

## 聚合查询

```typescript
// 计数
const count = await User.count({ isActive: true })

// 求和
const total = await Order.sum('amount', { status: 'paid' })

// 最大/最小
const maxPrice = await Product.max('price')
const minPrice = await Product.min('price')
```

---

## 原生 SQL

### 查询

```typescript
const results = await ksql.query(
    'SELECT * FROM users WHERE age > ? AND status = ?',
    [18, 'active']
)
```

### 执行

```typescript
const affected = await ksql.execute(
    'UPDATE users SET status = ? WHERE age < ?',
    ['inactive', 18]
)
```

---

## 模型关系

### 外键引用

```typescript
const Order = ksql.define('orders', {
    userId: {
        type: DataTypes.Number,
        ref: 'users.id'
    },
    total: {
        type: DataTypes.Number
    }
}, { timestamps: true })

// 查找时包含关联数据（需要自行处理联表）
const orders = await Order.findAll()
const ordersWithUser = orders.map(order => ({
    ...order,
    user: await User.findById(order.userId)
}))
```

---

## 最佳实践

1. **启用软删除**：`softDelete: true`，防止误删
2. **添加时间戳**：`timestamps: true`，跟踪创建和更新时间
3. **常用字段加索引**：`index: true`，提升查询性能
4. **分页查询**：大数据量使用 `findPaginated`，避免 `findAll`
5. **指定查询字段**：使用 `select` 避免 `SELECT *`

---

## ⚠️ 重要注意事项

### 1. k_sqlite 是同步操作，不需要 await

Kooboo 的 k_sqlite ORM 是**同步**执行的，不需要使用 `await`：

```typescript
// ✅ 正确：同步调用
const user = User.findById(id)
const users = User.findAll({})
User.updateById(id, { name: 'new' })
User.removeById(id)

// ❌ 错误：不需要 await
const user = await User.findById(id)  // 不需要 await
```

### 2. 主键是 `_id`，不是 `id`

Kooboo 的 sqlite 自动生成的主键字段是 `_id`，不是 `id`：

```typescript
// ✅ 正确：使用 _id
const user = User.findById(userId)
return { id: user._id }

// 在 select 中也要用 _id
const users = User.findAll({}, {
    select: ['_id', 'userName', 'email']
})

// 查询条件中使用 _id
const order = OrderFoodRecord.findOne({ _id: orderId })
```

### 3. Model 定义不需要手动定义 id 字段

k_sqlite 会自动生成 `_id` 字段，Model 定义时不需要写 id：

```typescript
// ✅ 正确：不需要定义 id
const User = ksql.define('users', {
    userName: { type: DataTypes.String, required: true },
    email: { type: DataTypes.String }
}, { timestamps: true })

// ❌ 错误：不需要手动定义 id
const User = ksql.define('users', {
    id: { type: DataTypes.Number, primaryKey: true },  // 不需要
    userName: { type: DataTypes.String }
})
```

### 4. 关联查询使用 map 而非 Promise.all

由于是同步操作，关联查询使用 map 即可：

```typescript
// ✅ 正确：同步 map
const orders = Order.findAll({})
const ordersWithDetails = orders.map(order => ({
    ...order,
    details: OrderDetail.findAll({ orderId: order._id })
}))

// ❌ 错误：不需要 Promise.all
const ordersWithDetails = await Promise.all(
    orders.map(async order => { ... })  // 不需要 async/await
)
```

---

## 快速索引

| 操作 | k.DB.sqlite | k_sqlite ORM |
|------|-------------|---------------|
| 查询所有 | `.all()` | `.findAll()` |
| 条件查询 | `.findAll({})` | `.findAll(where)` |
| 单条 | `.findOne({})` | `.findOne(where)` |
| 插入 | `.add({})` | `.create({})` |
| 更新 | `.update({})` | `.updateOne(where, {})` |
| 删除 | `.delete(id)` | `.removeById(id)` |
| 分页 | 无 | `.findPaginated()` |
| 原生 SQL | `.query()` | `.query()` / `.execute()` |
