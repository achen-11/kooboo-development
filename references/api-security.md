# Kooboo 安全、网络、档涵盖 `邮件 API

本文k.security`（安全加密）、`k.net`（网络请求）、`k.mail`（邮件发送）三个模块。

---

## k.security — 安全加密

### 哈希

```javascript
// MD5 哈希
const md5Hash = k.security.md5('password')
// 输出: 5f4dcc3b5aa765d61d8327deb882cf99

// SHA256 哈希
const sha256Hash = k.security.sha256('password')
// 输出: 5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8

// SHA512 哈希
const sha512Hash = k.security.sha512('password')
```

### 加密/解密

```javascript
// AES 加密
const encrypted = k.security.encrypt('Hello World', 'my-secret-key')
// 输出: base64 编码的加密字符串

// AES 解密
const decrypted = k.security.decrypt(encrypted, 'my-secret-key')
// 输出: Hello World
```

### Base64

```javascript
// 编码
const encoded = k.security.toBase64('Hello World')
// 输出: SGVsbG8gV29ybGQ=

// 解码
const decoded = k.security.decodeBase64(encoded)
// 输出: Hello World
```

### JWT

```javascript
// 生成 JWT
const token = k.security.jwt.encode(
    { userId: 123, role: 'admin' },
    'secret-key'
)
// 输出: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// 解析 JWT
const payload = k.security.jwt.decode(token)
// 输出: { userId: 123, role: 'admin', iat: ..., exp: ... }

// 验证 JWT
const isValid = k.security.jwt.verify(token, 'secret-key')
// 输出: true 或 false
```

### 随机字符串

```javascript
// 生成随机字符串
const randomStr = k.security.random(16)
// 输出: a8f5f167f44d496e

// 生成随机 UUID
const uuid = k.security.uuid()
// 输出: 550e8400-e29b-41d4-a716-446655440000
```

---

## k.net — 网络请求

### GET 请求

```javascript
// 简单 GET
const html = k.net.url.get('https://example.com')

// GET JSON
const json = k.net.url.getJson('https://api.example.com/data')

// 获取二进制
const binary = k.net.url.getAsBinary('https://example.com/image.png')

// 带请求头
const data = k.net.url.get('https://api.example.com', {
    'Authorization': 'Bearer token',
    'Content-Type': 'application/json'
})

// 基本认证
const data = k.net.url.get('https://api.example.com', 'username', 'password')
```

### POST 请求

```javascript
// POST JSON
const result = k.net.url.post('https://api.example.com/create', {
    name: 'John',
    email: 'john@example.com'
})

// POST 表单
const result = k.net.url.postform('https://api.example.com/submit', {
    name: 'John',
    message: 'Hello'
})

// POST 二进制
const result = k.net.url.postBinary('https://api.example.com/upload', fileData)
```

### 其他 HTTP 方法

```javascript
// PUT
const result = k.net.url.put('https://api.example.com/update/1', data)

// DELETE
const result = k.net.url.delete('https://api.example.com/delete/1')

// PATCH
const result = k.net.url.patch('https://api.example.com/patch/1', data)
```

### 请求配置

```javascript
// 完整配置
const result = k.net.url.get('https://api.example.com', {
    'Authorization': 'Bearer token',  // 请求头
    timeout: 30000,                  // 超时时间（毫秒）
    followRedirects: true            // 是否跟随重定向
})
```

### 响应处理

```javascript
const response = k.net.url.get('https://api.example.com/data')

// 响应状态码
const status = response.statusCode

// 响应头
const contentType = response.headers['content-type']

// 响应体
const body = response.body
```

---

## k.mail — 邮件发送

### SMTP 发送

```javascript
// 创建邮件
const msg = k.mail.createMessage()

// 设置收件人
msg.to = 'user@example.com'
msg.to = 'user1@example.com, user2@example.com'  // 多个收件人

// 设置发件人（默认使用站点配置）
msg.from = 'noreply@example.com'
msg.fromName = '网站名称'

// 设置主题
msg.subject = '欢迎注册'

// 设置正文（纯文本）
msg.body = '欢迎加入我们的网站！'

// 或设置 HTML 正文
msg.bodyHtml = '<h1>欢迎</h1><p>欢迎加入我们的网站！</p>'

// 添加附件
msg.addAttachment('/path/to/file.pdf', 'report.pdf')

// 发送
const result = k.mail.smtp.send(msg)

if (result.success) {
    console.log('邮件发送成功')
}
```

### 完整示例

```javascript
// api/send-welcome-email.ts
k.api.post('send-welcome', (body) => {
    const { email, name } = body

    const msg = k.mail.createMessage()
    msg.to = email
    msg.subject = '欢迎注册'
    msg.bodyHtml = `
        <h1>欢迎 ${name}！</h1>
        <p>感谢您注册我们的网站。</p>
        <p>点击以下链接验证邮箱：</p>
        <a href="https://example.com/verify?email=${encodeURIComponent(email)}">验证邮箱</a>
    `

    const result = k.mail.smtp.send(msg)

    if (result.success) {
        return { success: true, message: '邮件已发送' }
    } else {
        k.response.statusCode(500)
        return { success: false, message: '邮件发送失败' }
    }
})
```

### AWS SES 发送

```javascript
// 使用 AWS SES
const mail = k.mail.amazonses.createEmail({
    accessKeyId: 'AKIAIOSFODNN7EXAMPLE',
    secretAccessKey: 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY',
    region: 'us-east-1'
})

mail.to = 'user@example.com'
mail.subject = 'Test'
mail.body = 'Hello'

const result = mail.send()
```

---

## k.site — 网站信息

### 基本信息

```javascript
// 网站 ID
const siteId = k.site.webSite.id

// 网站信息
const info = k.site.info
info.host           // 域名
info.name           // 网站名称
info.baseUrl        // 基础 URL
info.culture        // 当前语言

// 多语言
const currentLang = k.site.multilingual.currentCulture
const availableLangs = k.site.multilingual.cultures
```

### 资源管理

```javascript
// 页面
k.site.pages.all()
k.site.pages.get('page-id')

// 视图
k.site.views.all()

// 布局
k.site.layouts.all()

// 脚本
k.site.scripts.all()

// 样式
k.site.styles.all()

// 图片/媒体
k.site.images.all()
```

---

## 完整示例：调用外部 API

```typescript
// api/fetch-products.ts
k.api.get('fetch-products', () => {
    // 从外部 API 获取商品数据
    const response = k.net.url.getJson('https://api.example.com/products', {
        'Authorization': 'Bearer ' + k.session.token
    })

    if (!response || response.error) {
        k.response.statusCode(502)
        return { success: false, message: '获取商品失败' }
    }

    // 处理数据并返回
    const products = response.data.map(p => ({
        id: p.id,
        name: p.name,
        price: p.price
    }))

    return { success: true, data: products }
})
```

---

## 完整示例：用户密码处理

```typescript
// 注册时密码哈希
k.api.post('register', (body) => {
    const { username, password, email } = body

    // 检查用户是否已存在
    const existing = k.DB.sqlite.users.findOne({ email })
    if (existing) {
        k.response.statusCode(400)
        return { success: false, message: '邮箱已被注册' }
    }

    // 密码哈希存储
    const passwordHash = k.security.sha256(password + 'salt')

    // 创建用户
    k.DB.sqlite.users.add({
        username,
        email,
        password: passwordHash
    })

    return { success: true, message: '注册成功' }
})

// 登录时验证密码
k.api.post('login', (body) => {
    const { email, password } = body

    const user = k.DB.sqlite.users.findOne({ email })
    if (!user) {
        k.response.statusCode(401)
        return { success: false, message: '用户不存在' }
    }

    const passwordHash = k.security.sha256(password + 'salt')
    if (user.password !== passwordHash) {
        k.response.statusCode(401)
        return { success: false, message: '密码错误' }
    }

    // 生成 JWT
    const token = k.security.jwt.encode(
        { userId: user.id, email: user.email },
        'secret-key'
    )

    return { success: true, token }
})
```

---

## 快速索引

### 安全 (k.security)

| 功能 | API |
|------|-----|
| MD5 | `k.security.md5(str)` |
| SHA256 | `k.security.sha256(str)` |
| 加密 | `k.security.encrypt(data, key)` |
| 解密 | `k.security.decrypt(encrypted, key)` |
| Base64 编码 | `k.security.toBase64(str)` |
| Base64 解码 | `k.security.decodeBase64(str)` |
| JWT 生成 | `k.security.jwt.encode(payload, secret)` |
| JWT 解析 | `k.security.jwt.decode(token)` |
| JWT 验证 | `k.security.jwt.verify(token, secret)` |
| 随机字符串 | `k.security.random(len)` |

### 网络 (k.net)

| 功能 | API |
|------|-----|
| GET | `k.net.url.get(url)` |
| GET JSON | `k.net.url.getJson(url)` |
| POST | `k.net.url.post(url, data)` |
| POST 表单 | `k.net.url.postform(url, data)` |
| PUT | `k.net.url.put(url, data)` |
| DELETE | `k.net.url.delete(url)` |

### 邮件 (k.mail)

| 功能 | API |
|------|-----|
| 创建邮件 | `k.mail.createMessage()` |
| 发送 | `k.mail.smtp.send(msg)` |
| AWS SES | `k.mail.amazonses.createEmail(config)` |
