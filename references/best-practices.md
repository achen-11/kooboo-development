# Kooboo 最佳实践

本文档汇总来自 KB_DOC 知识库的实战最佳实践。

---

## 1. WebSocket 实时通信

### 基本用法

```typescript
// 创建 WebSocket 连接
const ws = k.websocket.connect('wss://example.com/ws')

// 发送消息
ws.send(JSON.stringify({ type: 'message', content: 'Hello' }))

// 接收消息
ws.on('message', (data) => {
    console.log('收到:', data)
})

// 关闭连接
ws.close()
```

### 心跳机制

```typescript
// 定时发送心跳
setInterval(() => {
    if (ws && ws.readyState === WebSocket.OPEN) {
        ws.send(JSON.stringify({ type: 'heartbeat' }))
    }
}, 30000)

// 自动重连
ws.on('close', () => {
    setTimeout(() => {
        connectWebSocket()
    }, 5000)
})
```

---

## 2. 微信支付集成

### 发起支付

```typescript
// api/wechat-pay.ts
k.api.post('create-payment', (body) => {
    const { orderId } = body
    const order = k.commerce.order.get(orderId)

    // 调用微信支付 API
    const payParams = {
        appid: 'wxYOUR_APPID',
        mch_id: 'YOUR_MCHID',
        body: '商品描述',
        out_trade_no: order.orderNumber,
        total_fee: order.total * 100,  // 单位：分
        notify_url: 'https://yoursite.com/api/wechat-callback',
        trade_type: 'JSAPI',
        openid: k.request.headers['x-openid']
    }

    const result = k.net.url.postXml(
        'https://api.mch.weixin.qq.com/pay/unifiedorder',
        payParams
    )

    return {
        success: true,
        data: {
            appId: payParams.appid,
            timeStamp: Math.floor(Date.now() / 1000).toString(),
            nonceStr: result.nonce_str,
            package: 'prepay_id=' + result.prepay_id,
            signType: 'MD5',
            paySign: sign(payParams)
        }
    }
})
```

### 支付回调

```typescript
// api/wechat-callback.ts
k.api.post('wechat-callback', () => {
    const xml = k.request.body
    const data = parseXml(xml)

    // 验证签名
    if (!verifySign(data)) {
        k.response.statusCode(400)
        return '<xml><return_code><![CDATA[FAIL]]></return_code></xml>'
    }

    // 更新订单状态
    k.commerce.order.updateStatus(data.out_trade_no, 'paid')

    return '<xml><return_code><![CDATA[SUCCESS]]></return_code></xml>'
})
```

---

## 3. 发送邮件

### SMTP 发送

```typescript
// api/send-email.ts
k.api.post('send', (body) => {
    const { to, subject, content } = body

    const msg = k.mail.createMessage()
    msg.to = to
    msg.subject = subject
    msg.bodyHtml = content

    const result = k.mail.smtp.send(msg)

    return result
})
```

### 发送 HTML 邮件

```typescript
const msg = k.mail.createMessage()
msg.to = 'user@example.com'
msg.subject = '订单确认'
msg.bodyHtml = `
    <html>
    <body>
        <h1>感谢您的订单！</h1>
        <p>订单号：${order.orderNumber}</p>
        <p>金额：¥${order.total}</p>
    </body>
    </html>
`

k.mail.smtp.send(msg)
```

---

## 4. HTTP 请求转发

### 代理转发

```typescript
// api/proxy.ts
k.api.post('proxy', (body) => {
    const { url, method = 'GET' } = body

    // 验证 URL（防止 SSRF）
    if (!url.startsWith('https://api.example.com/')) {
        k.response.statusCode(400)
        return { error: 'Invalid URL' }
    }

    // 转发请求
    let result
    if (method === 'GET') {
        result = k.net.url.get(url)
    } else {
        result = k.net.url.post(url, body)
    }

    return result
})
```

### 携带认证

```typescript
const response = k.net.url.get(targetUrl, {
    'Authorization': 'Bearer ' + k.session.token,
    'X-API-Key': 'your-api-key'
})
```

---

## 5. 数据模型设计

### 使用 k_sqlite ORM

```typescript
// code/Models/Order.ts
import { ksql, DataTypes } from 'module/k_sqlite'

const Order = ksql.define('orders', {
    orderNumber: {
        type: DataTypes.String,
        required: true,
        unique: true,
        index: true
    },
    userId: {
        type: DataTypes.Number,
        required: true,
        ref: 'users.id'
    },
    total: {
        type: DataTypes.Number,
        required: true,
        default: 0
    },
    status: {
        type: DataTypes.String,
        default: 'pending'
    },
    shippingAddress: {
        type: DataTypes.Object
    },
    items: {
        type: DataTypes.Array
    }
}, {
    timestamps: true,
    softDelete: true
})

export { Order }
```

### 模型关系

```typescript
// 用户模型
const User = ksql.define('users', {
    name: { type: DataTypes.String, required: true },
    email: { type: DataTypes.String, unique: true }
}, { timestamps: true })

// 订单模型（外键关联）
const Order = ksql.define('orders', {
    userId: { type: DataTypes.Number, ref: 'users.id' },
    total: { type: DataTypes.Number }
}, { timestamps: true })

// 查询时手动联表
const orders = await Order.findAll()
const ordersWithUser = await Promise.all(
    orders.map(async (order) => ({
        ...order,
        user: await User.findById(order.userId)
    }))
)
```

---

## 6. 文件上传下载

### 文件上传

```typescript
// api/upload.ts
k.api.post('upload', () => {
    if (!k.request.files || k.request.files.length === 0) {
        k.response.statusCode(400)
        return { success: false, message: 'No file uploaded' }
    }

    const file = k.request.files[0]
    const fileName = `${Date.now()}_${file.fileName}`

    // 保存到 media 目录
    file.save('uploads/' + fileName)

    return {
        success: true,
        url: '/media/uploads/' + fileName,
        fileName: file.fileName,
        size: file.size
    }
})
```

### 文件下载

```typescript
// api/download.ts
k.api.get('download', () => {
    const { file } = k.request.queryString

    // 安全检查：防止路径遍历
    if (file.includes('..')) {
        k.response.statusCode(403)
        return { error: 'Invalid path' }
    }

    k.response.file('uploads/' + file)
})
```

---

## 7. 用户登录与权限

### 登录验证

```typescript
// api/login.ts
k.api.post('login', (body) => {
    const { email, password } = body

    const user = k.DB.sqlite.users.findOne({ email })
    if (!user) {
        k.response.statusCode(401)
        return { success: false, message: '用户不存在' }
    }

    const hash = k.security.sha256(password + user.salt)
    if (hash !== user.password) {
        k.response.statusCode(401)
        return { success: false, message: '密码错误' }
    }

    // 设置会话
    k.session.userId = user.id
    k.session.userName = user.name

    return {
        success: true,
        user: { id: user.id, name: user.name, email: user.email }
    }
})
```

### 权限检查中间件

```typescript
// code/middleware/auth.ts
export function requireAuth() {
    if (!k.session.userId) {
        k.response.statusCode(401)
        return false
    }
    return true
}

export function requireRole(roles: string[]) {
    if (!requireAuth()) return false

    const user = k.DB.sqlite.users.findById(k.session.userId)
    if (!roles.includes(user.role)) {
        k.response.statusCode(403)
        return false
    }
    return true
}
```

---

## 8. 电商购物车和订单

### 购物车操作

```typescript
// api/cart.ts
k.api.get('cart', () => {
    const cart = k.commerce.cart.get()
    return cart
})

k.api.post('add-item', (body) => {
    const { productId, quantity = 1 } = body

    const product = k.commerce.product.get(productId)
    if (!product) {
        k.response.statusCode(404)
        return { success: false, message: '商品不存在' }
    }

    const cart = k.commerce.cart.get()
    cart.addItem(productId, quantity)

    return { success: true, cart }
})
```

### 创建订单

```typescript
k.api.post('create-order', () => {
    const cart = k.commerce.cart.get()

    if (!cart.items || cart.items.length === 0) {
        k.response.statusCode(400)
        return { success: false, message: '购物车为空' }
    }

    const order = k.commerce.order.create(cart)
    cart.clear()

    return { success: true, orderId: order.id }
})
```

---

## 9. Vue 快速开发

### 环境搭建

```bash
# 使用模板创建项目
npm create vite@latest my-vue-app -- --template vue-ts

# 安装依赖
npm install

# 安装 Kooboo 相关包
npm install axios pinia vue-router element-plus @element-plus/icons-vue
```

### Vue 文件结构

```
src/
├── views/
│   └── HomeView.vue
├── components/
│   └── HelloWorld.vue
├── api/
│   └── index.ts
├── stores/
│   └── user.ts
├── router/
│   └── index.ts
└── App.vue
```

### Kooboo API 封装

```typescript
// api/index.ts
const baseURL = '/api'

export const api = {
    get: (url: string) => fetch(baseURL + url).then(r => r.json()),
    post: (url: string, data: any) => fetch(baseURL + url, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
    }).then(r => r.json())
}
```

---

## 10. 国际化

### 服务端

```typescript
// 使用 k.label
const title = k.label('page.title')
const greeting = k.t('Hello {name}', { name: 'John' })
```

### 客户端

```javascript
// 在 HTML 中引入语言文件
<script src="/js/i18n/zh-CN.js"></script>

// 使用
window.t('page.title')
```

---

## 快速索引

| 场景 | 参考 |
|------|------|
| WebSocket 实时通信 | `k.websocket.connect()` |
| 微信支付 | `k.commerce.payment.create()` |
| 发送邮件 | `k.mail.smtp.send()` |
| HTTP 代理 | `k.net.url.post()` |
| 数据模型 | `ksql.define()` |
| 文件上传 | `k.request.files[].save()` |
| 用户登录 | `k.session.set()` |
| 权限控制 | 中间件函数 |
| Vue 开发 | Vite + Element Plus |
| 国际化 | `k.label()` / `k.t()` |
