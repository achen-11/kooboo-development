问题 1:
错误使用 k.security.encode 和 k.security.decode 方法，
1. encode 方法直接 `k.security.encode({})`即可
2. decode 方案直接 `k.security.decode(token)`即可
   1. 该方法返回 string 类型
   2. 解析后返回对象
      1. 若 token 有效返回 `{code: 0, value: {}}`，
      2. 否则返回 `{code: 1, message: 'token invalid'}`
问题 2:
Models 使用错误
1. 正确创建了 Models, 然后通过`code/Models/index.ts`聚合, 这个操作是没有问题的, 但在引用时, 路径使用错误
```ts
// 错误
import { User } from 'code/Models'
// 正确
import { User } from 'code/Models/index'
```

问题 3:
正确拆分了 api, services, models 层, 但services 没有正确定义
```ts
// 错误
// 创建了 /code/auth.ts
import { xx } from 'code/auth'
// 正确
// 创建 /code/Services/auth.ts
import { xx } from 'code/Services/auth'
```

问题 4
过份拆分 api, 同一个模块的 api 应该尽量放置在一个文件即可
- 错误
  - 创建了 api/order-create.ts
  - 创建了 api/order-list.ts
- 正确
  - 创建 api/order.ts
```ts
// @k-url /api/order/{action}
k.api.get("list", ()=>{
    //...
})

k.api.post("create", (body: { productId: string, quantity: number }) => {
    //...
})
```

问题 5
类似 id 这样的参数, 建议使用 query 参数, 而不是路径参数
```ts
//错误
// @k-url /api/order/{id}
k.api.get("get", (id: string) => {
    //...
})
// 正确
// @k-url /api/order?id={id}
k.api.get("get", (id: string) => {
    //...
})
```