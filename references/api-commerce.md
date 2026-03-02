# Kooboo 电商 API

Kooboo 内置完整的电商模块，提供商品、购物车、订单、分类、支付等功能。

---

## k.commerce — 电商核心

### 商品管理

```javascript
// 获取商品列表
const products = k.commerce.product.list()

// 条件查询
const products = k.commerce.product.list({
    category: 'electronics',
})

// 获取单个商品
const product = k.commerce.product.get(idOrSeoName)

// 获取商品变体
const variants = k.commerce.product.get(productId).variants
```

### 商品字段

```javascript
const product = k.commerce.product.get(productId)

// 基本信息
product.id
product.seoName
product.title
product.description

// 图片
product.images
product.featuredImage

// 分类
product.categories

// 标签
product.tags

// 状态
product.active

// 变体
product.variants

// 变体属性
// 价格
product.variants[0].price
```

---

## 分类管理

```javascript
// 获取所有分类
const categories = k.commerce.category.list()

// 按 ID 获取分类
const category = k.commerce.category.get(seoNameOrId)

// 获取分类下的子分类()
const subcategories = k.commerce.category.get(seoNameOrId).children

```

---

## 购物车

### 购物车操作

```javascript
// 创建购物车
const cart = k.commerce.cart.create()

// 添加或更新商品到购物车
k.commerce.cart.addOrUpdateLine(cartId, variantId, quantity)

// 移除商品
k.commerce.cart.removeLine(cartId, variantId)
```

### 购物车项目

```javascript
const cart = k.commerce.cart.get(id)

for (const item of cart.lines) {
    item.variantId           // 变体 ID
    item.quantity     // 数量
}
```

---

## 订单管理

### 创建订单

```javascript
// 创建订单
const order = k.commerce.order.create(cartId)
```

### 订单操作

```javascript
// 更新订单状态
k.commerce.order.pay(orderId, method) // 支付

k.commerce.order.cancel(orderId, reason) // 取消订单
k.commerce.order.delivery(orderId, lineIds)

```

### 查询订单

```javascript
// 获取用户订单列表
const orders = k.commerce.order.list()

// 获取订单详情
const order = k.commerce.order.get(orderId)

// 按状态查询
const paidOrders = k.commerce.order.list({
    customerId, // 客户 ID
    pageIndex,  // 页码
    pageSize,   // 每页数量
    canceled, // 是否已取消
    delivered, // 是否已发货
    paid, // 是否已支付
    startDate, // 开始日期
    endDate, // 结束日期
})
```

### 订单字段

```javascript
const order = k.commerce.order.get(orderId)

order.id
order.clientInfo // 客户信息

order.subtotalAmount   // 商品小计
order.shippingAmount   // 运费
order.shippingAt       // 发货时间
order.taxAmount        // 税费
order.totalAmount      // 订单总额

order.shippingAddress  // 收货地址
order.notes            // 订单备注

order.lines            // 订单商品列表
order.paymentMethod   // 支付方式
order.trackingNumber  // 快递单号

order.paid   // 是否已支付
order.paidAt // 支付时间

order.canceled       // 是否已取消
order.canceledReason // 取消原因
order.canceledAt     // 取消时间

order.createdAt
order.updatedAt
```

---

## 客户管理

```javascript
// 获取当前用户
const customer = k.commerce.customer.get(id)

// 更新客户信息
k.commerce.customer.update({
    firstName: 'John',
    lastName: 'Doe',
    phone: '1234567890',
    email: 'john@example.com'
})

// 获取客户地址
const addresses = k.commerce.customer.get(id).addresses

// 更新地址
addresses.push({

})
k.commerce.customer.updateAddresses(customerId, {
    id,
    firstName,
    lastName,
    phone,
    country,
    province,
    city,
    zip,
    address1,
    address2,
    isDefault,
})
```
// marked: 仅 review 到这里
---

## 支付

### 支付流程

```javascript
// 创建支付
const payment = k.commerce.payment.create(orderId, {
    method: 'alipay',
    returnUrl: 'https://example.com/payment/return',
    notifyUrl: 'https://example.com/payment/notify'
})

// 获取支付链接
const payUrl = payment.url

// 验证支付回调
const verified = k.commerce.payment.verify(callbackData)
```

### 微信支付示例

```javascript
// 发起微信支付
k.api.post('wechat-pay', () => {
    const order = k.commerce.order.get(body.orderId)
    
    const result = k.commerce.payment.create(order.id, {
        method: 'wechat',
        openid: k.request.headers['x-openid'],
        amount: order.total,
        description: 'Order ' + order.orderNumber
    })

    return {
        success: true,
        payInfo: {
            appId: result.appId,
            timeStamp: result.timeStamp,
            nonceStr: result.nonceStr,
            package: result.package,
            signType: result.signType,
            paySign: result.paySign
        }
    }
})
```

---

## 优惠码

```javascript
// 使用优惠码
const discount = k.commerce.discount.apply('SUMMER20')

if (discount) {
    // 优惠码有效
    const discountAmount = discount.amount
    const newTotal = order.total - discountAmount
}

// 获取优惠信息
const discount = k.commerce.discount.get(code)
```

---

## 运费

```javascript
// 计算运费
const shippingFee = k.commerce.shipping.calculate(order)

// 获取可用配送方式
const methods = k.commerce.shipping.methods()

// 常见配送方式
// 'standard' - 标准快递
// 'express' - 快递
// 'pickup' - 自提
```

---

## 货币

```javascript
// 货币转换
const price = k.commerce.currency.convert(100, 'USD', 'EUR')

// 获取当前货币
const currency = k.commerce.currency.current()

// 设置货币
k.commerce.currency.set('CNY')
```

---

## 完整示例：创建订单流程

```javascript
// api/create-order.ts
k.api.post('create-order', () => {
    const cart = k.commerce.cart.get()
    
    if (!cart.items || cart.items.length === 0) {
        k.response.statusCode(400)
        return { success: false, message: '购物车为空' }
    }

    // 检查库存
    for (const item of cart.items) {
        const product = k.commerce.product.get(item.productId)
        if (product.stock < item.quantity) {
            k.response.statusCode(400)
            return { 
                success: false, 
                message: `商品 ${product.name} 库存不足` 
            }
        }
    }

    // 创建订单
    const order = k.commerce.order.create(cart)

    // 清空购物车
    cart.clear()

    return { 
        success: true, 
        orderId: order.id,
        orderNumber: order.orderNumber 
    }
})
```

---

## 快速索引

| 功能 | API |
|------|-----|
| 商品列表 | `k.commerce.product.list()` |
| 商品详情 | `k.commerce.product.get(id)` |
| 分类 | `k.commerce.category.all()` |
| 购物车 | `k.commerce.cart.get()` |
| 加购 | `cart.addItem(productId, qty)` |
| 创建订单 | `k.commerce.order.create(cart)` |
| 订单状态 | `k.commerce.order.updateStatus(id, status)` |
| 客户信息 | `k.commerce.customer.get()` |
| 支付 | `k.commerce.payment.create()` |
| 优惠码 | `k.commerce.discount.apply(code)` |
| 运费 | `k.commerce.shipping.calculate(order)` |
| 货币转换 | `k.commerce.currency.convert(amount, from, to)` |
| 心愿单 | `k.commerce.wishlist.add(productId)` |
