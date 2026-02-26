# Kooboo 模块系统

Kooboo 模块是可安装的插件，可以被打包、发布和在 Kooboo 站点上安装使用。

---

## 模块结构

```
module_name/
├── root/
│   ├── Module.config    # 必需：模块配置
│   ├── Event.js        # 模块生命周期事件
│   └── Readme.md      # 模块说明文档
├── view/               # 模块视图（可选）
├── code/               # 模块代码（可选）
├── api/                # 模块 API（可选）
└── package.json        # 依赖配置（可选）
```

---

## Module.config

模块配置文件，必须放在 `root/` 目录下：

```json
{
    "name": "my_module",
    "description": "模块的简短描述",
    "version": "1.0.0",
    "author": "Your Name",
    "authorEmail": "you@example.com",
    "website": "https://example.com",
    "menu": {
        "name": "我的模块",
        "icon": "icon.svg",
        "url": "/_modules/my_module"
    },
    "config": {
        "key1": "value1",
        "key2": "value2"
    },
    "dependencies": {
        "other_module": ">=1.0.0"
    }
}
```

### 配置字段说明

| 字段 | 必需 | 说明 |
|------|------|------|
| name | 是 | 模块名称（英文、kebab-case） |
| description | 是 | 模块描述 |
| version | 是 | 版本号（语义化版本） |
| author | 否 | 作者名称 |
| authorEmail | 否 | 作者邮箱 |
| website | 否 | 作者网站 |
| menu | 否 | 后台菜单配置 |
| config | 否 | 模块配置项 |
| dependencies | 否 | 依赖的其他模块 |

---

## Event.js

模块生命周期事件处理：

```javascript
// root/Event.js

// 模块安装时执行
exports.onInstall = function (moduleContext) {
    console.log('模块已安装')
    
    // 可以在这里初始化数据库表
    // 创建默认数据等
}

// 模块卸载时执行
exports.onUninstall = function (moduleContext) {
    console.log('模块已卸载')
    
    // 清理数据
}

// 模块启用时执行
exports.onEnable = function (moduleContext) {
    console.log('模块已启用')
}

// 模块禁用时执行
exports.onDisable = function (moduleContext) {
    console.log('模块已禁用')
}

// 模块升级时执行
exports.onUpgrade = function (moduleContext, oldVersion, newVersion) {
    console.log('模块从 ' + oldVersion + ' 升级到 ' + newVersion)
    
    // 数据迁移
}
```

---

## 模块间通信

### 引用其他模块

```typescript
// 引用 k_sqlite 模块
import { ksql, DataTypes } from 'module/k_sqlite'

// 引用 http_request 模块
import { http } from 'module/http_request'
```

### 暴露模块 API

```typescript
// code/index.ts - 模块导出
export function myFunction(data) {
    // 实现
    return result
}
```

---

## 模块视图

### 创建视图

```
module_name/
├── view/
│   ├── index.html      # 模块首页
│   ├── settings.html   # 设置页面
│   └── components/
│       └── ...
```

### 视图使用

```html
<!-- view/index.html -->
<layout id="module.main">
    <placeholder id="Main">
        <h1>模块首页</h1>
        <p>这是我的自定义模块</p>
    </placeholder>
</layout>
```

### 访问模块视图

```
https://site.com/_modules/module_name
https://site.com/_modules/module_name/settings
```

---

## 模块 API

### 定义 API

```
module_name/
├── api/
│   ├── index.ts
│   └── users.ts
```

```typescript
// api/users.ts
k.api.get('users', () => {
    return k.DB.sqlite.module_users.all()
})

k.api.post('create-user', (body) => {
    return k.DB.sqlite.module_users.add(body)
})
```

### 访问模块 API

```
https://site.com/_api/module_name/users
https://site.com/_api/module_name/create-user
```

---

## 模块打包

### 打包为 ZIP

1. 在 Kooboo 后台，进入「模块管理」
2. 选择要导出的模块
3. 点击「导出」
4. 系统会生成 `.zip` 文件

### 手动打包

```
module_name/
├── root/
│   ├── Module.config
│   ├── Event.js
│   └── Readme.md
├── view/
├── code/
├── api/
└── package.json
```

将上述目录打包为 `module_name.zip`。

---

## 模块安装

### 后台安装

1. 登录 Kooboo 管理后台
2. 进入「模块管理」
3. 点击「安装模块」
4. 上传 `.zip` 文件
5. 完成安装

### 导入安装

```typescript
// 使用模块导入 API
k.module.import('/path/to/module.zip')
```

---

## 常用模块

### k_sqlite

SQLite ORM 模块，提供数据库操作能力：

```typescript
import { ksql, DataTypes } from 'module/k_sqlite'

const User = ksql.define('users', {
    name: { type: DataTypes.String, required: true }
}, { timestamps: true })
```

### http_request

HTTP 请求模块：

```typescript
import { http } from 'module/http_request'

const response = await http.get('https://api.example.com')
const result = await http.post('https://api.example.com', { data: 'xxx' })
```

### sqlite_gui

SQLite 数据库图形化管理界面：

- 可视化浏览数据表
- 执行 SQL 查询
- 数据增删改查

---

## 模块开发最佳实践

1. **命名规范**：使用 kebab-case（如 `my-custom-module`）
2. **版本管理**：遵循语义化版本（semver）
3. **配置分离**：使用 `Module.config` 存储配置，避免硬编码
4. **清理数据**：在 `onUninstall` 中清理不再需要的数据
5. **数据迁移**：在 `onUpgrade` 中处理版本升级时的数据迁移
6. **文档完善**：提供 `Readme.md` 说明模块用途和使用方法

---

## 快速索引

| 操作 | 说明 |
|------|------|
| 创建模块 | 新建 `module_name/root/Module.config` |
| 定义生命周期 | 在 `root/Event.js` 导出事件函数 |
| 打包导出 | Kooboo 后台「导出」或手动打包 ZIP |
| 安装模块 | 后台「安装模块」上传 ZIP |
| 引用模块 | `import { xxx } from 'module/module_name'` |
| 模块视图 | 放在 `view/` 目录，访问 `/_modules/name` |
| 模块 API | 放在 `api/` 目录，访问 `/_api/name` |
