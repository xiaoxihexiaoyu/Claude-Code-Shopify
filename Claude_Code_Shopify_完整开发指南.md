# Claude Code + Shopify 全栈开发完整指南

> **Claude Code + Shopify** = 用 Claude Code 作为 AI 结对编程工具，高效开发 Shopify 项目

---

## 目录

1. [Claude Code 速览](#一claude-code-速览)
2. [Shopify 开发基础（完整版）](#二shopify-开发基础完整版)
3. [Hydrogen 完全指南](#三hydrogen-完全指南)
4. [Claude Code + Shopify 工作流](#四claude-code--shopify-工作流)
5. [第三方服务接入](#五第三方服务接入)
6. [实战场景案例](#六实战场景案例)
7. [最佳实践与技巧](#七最佳实践与技巧)
8. [学习资源](#八学习资源)

---

## 一、Claude Code 速览

### 1.1 什么是 Claude Code？

```
┌───────────────────────────────────────────────────────────┐
│                    Claude Code 简介                    │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  Claude Code 是 Anthropic 官方推出的 CLI 工具      │
│                                                           │
│  核心特性：                                                │
│  • 命令行工具: 在终端中与 Claude 对话式开发      │
│  • 代码库感知: 自动读取/理解你的项目结构       │
│  • 多模型驱动: Opus 4.6 / Sonnet / Haiku         │
│  • Agent 系统: 自动化复杂任务                       │
│  • Git 集成: 自动提交代码、管理分支                  │
│  • Skills 系统: 可扩展的命令集 (类似插件)       │
│                                                           │
│  简单理解: 一个能理解你整个项目的 AI 助手      │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### 1.2 安装与启动

```bash
# 安装 Claude Code
npm install -g @anthropic-ai/claude-code

# 启动
claude

# 查看版本
claude --version
```

### 1.3 与 ChatGPT/Claude Web 的区别

| 维度 | ChatGPT/Claude Web | Claude Code |
|------|-------------------|-------------|
| **代码库感知** | 需手动粘贴代码 | 自动读取整个项目 |
| **上下文** | 对话级记忆 | 项目级持久记忆 |
| **执行能力** | 只能生成代码 | 可直接读写文件、运行命令 |
| **Git 集成** | 无 | 深度集成 (commit/PR/分支) |
| **工作流** | 复制粘贴 | 对话式协作 |
| **适用场景** | 一次性问题 | 持续开发项目 |

---

## 二、Shopify 开发基础（完整版）

### 2.1 Shopify 生态概览

```
┌───────────────────────────────────────────────────────────┐
│                   Shopify 生态系统                    │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  ┌─────────────┐        ┌─────────────┐             │
│  │              │        │              │             │
│  │ Shopify 后台  │  ←→   │  Storefront API │  ←→   │ 你的前端     │
│  │ (Admin)      │        │  (只读)       │  │             │
│  │              │        │              │  │             │
│  └─────────────┘        └─────────────┘             │
│         ↓                       ↓                    │
│    产品/内容               产品数据              用户浏览器  │
│    订单/客户               博客文章              (React/Liquid)│
│    设置/政策               页面内容                          │
│         ↑                                                │
│         │                                                │
│  └─────────────┘                                           │
│  Admin API (读写)                                         │
│  (脚本/应用可调用)                                        │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### 2.2 Shopify 三大 API

#### 2.2.1 Storefront API（只读）

```
┌───────────────────────────────────────────────────────────┐
│                   Storefront API                       │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  用途：前端获取产品、内容、购物车数据                     │
│                                                           │
│  认证方式：Storefront API Access Token                │
│                                                           │
│  特点：                                                    │
│  • 只读操作（不能写数据）                            │
│  • 公开可暴露（相对安全）                           │
│  • 有访问限流                                         │
│                                                           │
│  能获取的数据：                                            │
│  ✓ 产品信息/价格/库存                                │
│  ✓ Collection 列表                                    │
│  ✓ Blog/Article 内容                                  │
│  ✓ Page 内容                                         │
│  ✓ 购物车 (配合 Cart API)                             │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

**示例查询：**
```graphql
query @inContext(language: EN, country: US) {
  product(handle: "my-product") {
    id
    title
    description
    priceRange {
      minVariantPrice {
        amount
        currencyCode
      }
    }
    images(first: 5) {
      nodes {
        url
        altText
      }
    }
  }
}
```

#### 2.2.2 Admin API（读写）

```
┌───────────────────────────────────────────────────────────┐
│                   Admin API                           │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  用途：后台管理、自动化脚本、第三方应用                   │
│                                                           │
│  认证方式：OAuth 2.0 或 API Credentials            │
│                                                           │
│  特点：                                                    │
│  • 完整 CRUD（创建/读取/更新/删除）                     │
│  • 需要认证                                              │
│  • 全量权限可配置                                        │
│                                                           │
│  能力：                                                    │
│  ✓ 产品/变体/图片 CRUD                                │
│  ✓ 订单管理                                            │
│  ✓ 客户资料管理                                         │
│  ✓ 折扣码创建                                           │
│  ✓ 库存调整                                            │
│  ✓ Metafield 管理                                     │
│  ✓ 政策更新                                            │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

**认证方式对比：**

| 方式 | 适用场景 | Token 有效期 | 权限范围 |
|------|---------|-------------|---------|
| **Admin API Access** | 后台集成、内部工具 | 可选永不过期 | 全部权限 |
| **OAuth 2.0** | 公开应用 | 需 refresh 刷新 | 用户授权 |
| **Client Credentials** | 自动化脚本 | 24 小时 | 预配置 scope |

#### 2.2.3 Liquid/Theme API

```
┌───────────────────────────────────────────────────────────┐
│              Liquid/Theme API                       │
├───────────────────────────────────────────────────────────┤
│                                                           │
│  用途：修改默认主题（如 Dawn）的外观和行为                 │
│                                                           │
│  技术栈：                                                  │
│  • Liquid 模板语言（Shopify 自研）                   │
│  • HTML/CSS/JavaScript                                │
│  • JSON Schema（配置 schema）                         │
│                                                           │
│  常用对象：                                                │
│  • {{ product }} — 产品数据                         │
│  • {{ cart }} — 购物车                              │
│  • {{ collection }} — 商品集合                      │
│  • {{ page }} — 页面内容                              │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### 2.3 三种开发模式对比

```
┌─────────────────────────────────────────────────────────────────┐
│                     Shopify 开发模式对比                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  模式                    │ Liquid 主题  │ Hydrogen     │ 全栈自定义     │
│  ──────────────────────────┼─────────────┼─────────────┼─────────────┤
│  技术栈                  │ Liquid+HTML/JS│ React+Vite   │ Next.js/Nuxt   │
│  自由度                   │ 低           │ 高          │ 极高         │
│  开发门槛                 │ 低           │ 中          │ 高           │
│  性能控制                 │ 有限         │ 高          │ 极高         │
│  SEO 控制                 │ 有限         │ 高          │ 极高         │
│  品牌差异化             │ 困难         │ 容易        │ 完全自由     │
│  适合场景                 │ 快速开店     │ DTC 品牌     │ 复杂应用       │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 2.4 Shopify 数据模型详解

```
┌─────────────────────────────────────────────────────────────────┐
│                    Shopify 数据模型                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Shop (商店)                                                        │
│  ├─ name (商店名称)                                                 │
│  ├─ description (描述)                                            │
│  ├─ metafields (自定义数据)                                        │
│  └─ policies (政策)                                               │
│                                                                         │
│  Product (产品)                                                     │
│  ├─ title (标题)                                                  │
│  ├─ description (描述)                                            │
│  ├─ variants (变体) ← ProductVariant                            │
│  │   ├─ price (价格)                                            │
│  │   ├─ sku (库存单位)                                         │
│  │   ├─ inventoryQuantity (库存数量)                            │
│  │   └─ availableForSale (是否可售)                             │
│  ├─ images (图片) ← ProductImage                                  │
│  └─ collections (所属合集)                                      │
│                                                                         │
│  Collection (商品集合)                                            │
│  ├─ title (标题)                                                  │
│  ├─ description (描述)                                            │
│  ├─ products (包含的产品)                                         │
│  └─ rules (筛选规则)                                              │
│                                                                         │
│  Customer (客户)                                                   │
│  ├─ email (邮箱)                                                 │
│  ├─ firstName (名)                                              │
│  ├─ lastName (姓)                                               │
│  ├─ orders (订单) ← Order                                        │
│  └─ addresses (地址) ← CustomerAddress                            │
│                                                                         │
│  Order (订单)                                                      │
│  ├─ orderNumber (订单号)                                         │
│  ├─ lineItems (订单项) ← LineItem                                  │
│  ├─ customer (客户)                                             │
│  ├─ shippingAddress (配送地址)                                    │
│  ├─ financialStatus (财务状态)                                  │
│  └─ fulfillments (履约信息) ← Fulfillment                        │
│                                                                         │
│  Blog (博客)                                                       │
│  ├─ title (标题)                                                 │
│  └─ articles (文章) ← Article                                      │
│                                                                         │
│  Article (文章)                                                   │
│  ├─ title (标题)                                                 │
│  ├─ content (HTML内容)                                           │
│  ├─ excerpt (摘要)                                              │
│  ├─ tags (标签)                                                 │
│  └─ comments (评论)                                             │
│                                                                         │
│  Page (页面)                                                       │
│  ├─ title (标题)                                                 │
│  └─ body (HTML内容)                                             │
│                                                                         │
│  Metafield (自定义字段)                                          │
│  ├─ namespace (命名空间)                                       │
│  ├─ key (键)                                                   │
│  ├─ value (值)                                                 │
│  └─ type (类型)                                                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 2.5 GraphQL 基础

```
┌─────────────────────────────────────────────────────────────────┐
│              REST vs GraphQL (Shopify)                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  REST API:                                                       │
│  • 多个端点，每个端点返回固定数据                             │
│  • 过度获取（over-fetching）或获取不足                          │
│  • 需要多次请求才能获取完整数据                               │
│                                                                         │
│  GraphQL API:                                                    │
│  • 单个端点，精确请求需要的数据                                │
│  • 按需获取，减少数据传输                                     │
│  • 类型安全，强类型系统                                        │
│  • Fragment 复用，减少重复                                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

**基础查询示例：**
```graphql
# 获取产品详情
query getProduct($handle: String!) {
  product(handle: $handle) {
    id
    title
    description
    productType
    totalInventory
    priceRange {
      minVariantPrice {
        amount
        currencyCode
      }
      maxVariantPrice {
        amount
        currencyCode
      }
    }
    variants(first: 10) {
      nodes {
        id
        title
        price {
          amount
          currencyCode
        }
        availableForSale
      }
    }
    images(first: 5) {
      nodes {
        url
        altText
        width
        height
      }
    }
  }
}
```

### 2.6 Metafield 完全指南

```
┌─────────────────────────────────────────────────────────────────┐
│                    Metafield 完全指南                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  什么是 Metafield？                                              │
│  Metafield 是 Shopify 的自定义数据系统，允许你在任何资源上   │
│  (产品、客户、订单、变体等)添加自定义字段。                         │
│                                                                         │
│  常见用途：                                                      │
│  ✓ 存储产品额外信息（尺寸、材质、Care 说明等）                │
│  ✓ 存储客户偏好（兴趣标签、生日、会员等级等）                  │
│  ✓ 存储全局配置（社交链接、品牌设置等）← Shop Metafield    │
│                                                                         │
│  Metafield 类型：                                                  │
│  ────────────────────────────────────────────────────────────  │
│  • single_line_text — 单行文本                                   │
│  • multi_line_text — 多行文本                                    │
│  • number_integer — 整数                                         │
│  • number_decimal — 小数                                         │
│  • date — 日期                                                   │
│  • date_time — 日期时间                                         │
│  • url — 网址链接                                               │
│  • json — JSON 对象                                              │
│  • boolean — 是/否                                               │
│  • color — 颜色选择器                                            │
│  • file — 文件上传                                              │
│  • money — 金额                                                 │
│  • rating — 评分 (1-5 星)                                         │
│  • dimension — 尺寸                                             │
│  • volume — 体积                                                │
│  • weight — 重量                                                │
│  • reference — 引用其他资源                                       │
│  • list.single_line_text — 单选列表                             │
│                                                                         │
│  创建 Metafield 步骤：                                            │
│  ────────────────────────────────────────────────────────────  │
│  1. 后台：设置 → 自定义数据 → 选择资源类型                   │
│  2. 点击"添加定义"                                               │
│  3. 填写命名空间、键、选择类型                                   │
│  4. 选择应用范围（所有资源/特定资源）                           │
│                                                                         │
│  读取 Metafield (GraphQL):                                         │
│  ────────────────────────────────────────────────────────────  │
│  query {                                                           │
│    product(handle: "my-product") {                               │
│      metafield(namespace: "custom", key: "care_instructions") { │
│        value                                                       │
│      }                                                             │
│    }                                                               │
│  }                                                                 │
│                                                                         │
│  写入 Metafield (Admin API):                                        │
│  ────────────────────────────────────────────────────────────  │
│  mutation {                                                         │
│    productUpdate(                                                 │
│      input: {                                                      │
│        id: "gid://shopify/Product/123"                            │
│        metafields: [                                               │
│          {                                                          │
│            namespace: "custom"                                │
│            key: "care_instructions"                             │
│            value: "Hand wash only"                             │
│            type: SINGLE_LINE_TEXT                               │
│          }                                                          │
│        ]                                                           │
│      }                                                             │
│    ) {                                                             │
│      product {                                                   │
│        id                                                         │
│      }                                                            │
│    }                                                             │
│  }                                                                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 2.7 Shopify Policy 完全指南

```
┌─────────────────────────────────────────────────────────────────┐
│                    Shopify Policy 完全指南                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  政策类型                                                        │
│  ────────────────────────────────────────────────────────────  │
│  • PRIVACY_POLICY — 隐私政策                                   │
│  • REFUND_POLICY — 退款政策                                    │
│  • TERMS_OF_SERVICE — 服务条款                                 │
│  • SHIPPING_POLICY — 发货政策                                  │
│  • SUBSCRIPTION_POLICY — 订阅政策                             │
│  • CONTACT_INFORMATION — 联系信息                             │
│  • LEGAL_NOTICE — 法律声明                                    │
│                                                                         │
│  后台位置                                                        │
│  ────────────────────────────────────────────────────────────  │
│  设置 → 政策                                                     │
│                                                                         │
│  查询方式 (GraphQL)                                              │
│  ────────────────────────────────────────────────────────────  │
│  query {                                                           │
│    shop {                                                          │
│      privacyPolicy { body }                                       │
│      refundPolicy { body }                                         │
│      termsOfService { body }                                       │
│      shippingPolicy { body }                                      │
│    }                                                               │
│  }                                                                 │
│                                                                         │
│  更新方式 (Admin API)                                            │
│  ────────────────────────────────────────────────────────────  │
│  mutation {                                                         │
│    shopPolicyUpdate(                                             │
│      shopPolicy: {                                                │
│        type: SHIPPING_POLICY                                     │
│        body: "<p>您的发货政策内容...</p>"                     │
│      }                                                           │
│    ) {                                                             │
│      shopPolicy { type body }                                   │
│    }                                                                 │
│  }                                                                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 2.8 Shopify Webhook 完全指南

```
┌─────────────────────────────────────────────────────────────────┐
│                    Shopify Webhook 完全指南                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  什么是 Webhook？                                                  │
│  Webhook 允许 Shopify 在特定事件发生时向你配置的 URL 发送   │
│  HTTP POST 请求，实现实时数据同步。                                 │
│                                                                         │
│  常见 Webhook 事件                                                  │
│  ────────────────────────────────────────────────────────────  │
│                                                                         │
│  订单相关                                                        │
│  • orders/create — 订单创建                                       │
│  • orders/updated — 订单更新                                       │
│  • orders/cancelled — 订单取消                                     │
│  • orders/fulfilled — 订单履约                                     │
│  • orders/paid — 订单支付                                         │
│                                                                         │
│  产品相关                                                        │
│  • products/create — 产品创建                                     │
│  • products/updated — 产品更新                                     │
│  • products/delete — 产品删除                                     │
│                                                                         │
│  客户相关                                                        │
│  • customers/create — 客户创建                                   │
│  • customers/updated — 客户更新                                   │
│  • customers/delete — 客户删除                                   │
│                                                                         │
│  购物车相关                                                      │
│  • carts/create — 购物车创建                                     │
│  • carts/update — 购物车更新                                     │
│                                                                         │
│  应用相关                                                        │
│  • app/uninstalled — 应用卸载                                   │
│                                                                         │
│  Webhook 处理流程                                                  │
│  ────────────────────────────────────────────────────────────  │
│  1. 接收 POST 请求                                               │
│  2. 验证 HMAC 签名（确保请求来自 Shopify）                   │
│  3. 解析 JSON 数据                                             │
│  4. 处理业务逻辑                                               │
│  5. 返回 200 OK                                                │
│                                                                         │
│  HMAC 验证示例 (Node.js)                                             │
│  ────────────────────────────────────────────────────────────  │
│  import crypto from 'crypto';                                     │
│                                                                         │
│  function verifyWebhook(data, hmac, webhookSecret) {           │
│    const computedHmac = crypto                                   │
│      .createHmac('sha256', webhookSecret)                     │
│      .update(data, 'utf8')                                       │
│      .digest('base64');                                          │
│    return crypto.timingSafeEqual(Buffer.from(hmac), computedHmac); │
│  }                                                                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 2.9 Shopify App 开发基础

```
┌─────────────────────────────────────────────────────────────────┐
│                 Shopify App 开发基础                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  App 类型                                                        │
│  ────────────────────────────────────────────────────────────  │
│  • Public App — 公开应用商店，所有商家可用                     │
│  • Custom App — 单独开发，仅安装到特定商店                    │
│  • Organization App — 组织级应用，多商店共享                    │
│                                                                         │
│  App 技术栈                                                      │
│  ────────────────────────────────────────────────────────────  │
│  • 后端 — Node.js / Ruby / PHP 等                                │
│  • 前端 — React / Vue / 等 (嵌入 iframe)                       │
│  • 数据库 — Shopify (通过 API) 或 自建数据库                       │
│  • 托管 — Heroku / Vercel / AWS 等                                 │
│                                                                         │
│  App 安装流程                                                    │
│  ────────────────────────────────────────────────────────────  │
│  1. 商家在应用商店点击"添加应用"                                 │
│  2. 重定向到你的托管服务器（确认 URL）                             │
│  3. 你的服务器生成安装链接并重定向回 Shopify                     │
│  4. Shopify 接收安装请求，生成 Access Token                         │
│  5. 你的服务器存储 Token 并返回确认                               │
│  6. Shopify 显示安装成功                                          │
│                                                                         │
│  OAuth 2.0 流程                                                   │
│  ────────────────────────────────────────────────────────────  │
│  1. 获取 API Key                                                    │
│  2. 用户访问安装 URL                                               │
│  3. 重定向到授权页面                                               │
│  4. 用户确认授权                                                 │
│  5. 重定向回你的服务器，带上 code                                │
│  6. 用 code 换取 Access Token                                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 2.10 Shopify Scripts 完全指南

```
┌─────────────────────────────────────────────────────────────────┐
│                Shopify Scripts (CLI 工具) 完全指南               │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Shopify CLI 是一个命令行工具，用于自动化 Shopify 开发任务     │
│                                                                         │
│  安装                                                           │
│  ────────────────────────────────────────────────────────────  │
│  npm install -g @shopify/cli @shopify/cli-kit                  │
│                                                                         │
│  常用命令                                                        │
│  ────────────────────────────────────────────────────────────  │
│  shopify login              # 登录 Shopify                       │
│  shopify logout             # 登出                               │
│                                                                         │
│  shopify theme push         # 上传主题                            │
│  shopify theme pop          # 下载主题                            │
│  shopify theme watch         # 监听文件变化并自动上传             │
│                                                                         │
│  shopify function create     # 创建 Function                       │
│  shopify function deploy     # 部署 Function                       │
│                                                                         │
│  shopify project start       # 启动项目                           │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 2.11 Shopify Flow 完全指南

```
┌─────────────────────────────────────────────────────────────────┐
│                  Shopify Flow (自动化工作流) 完全指南              │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  什么是 Flow？                                                    │
│  Flow 是 Shopify 的自动化工作流工具，可自动化重复性任务         │
│                                                                         │
│  Flow 类型                                                        │
│  ────────────────────────────────────────────────────────────  │
│  • Scheduled Flow — 定时触发                                   │
│  • Scheduled Task — 定时任务                                   │
│  • Scheduled Script — 定时脚本                                 │
│                                                                         │
│  常见用途                                                        │
│  ────────────────────────────────────────────────────────────  │
│  • 自动标记库存低的商品                                           │
│  • 定时发送邮件报告                                               │
│  • 自动更新订单状态                                               │
│  • 批量更新产品价格                                               │
│                                                                         │
│  创建方式                                                        │
│  ────────────────────────────────────────────────────────────  │
│  后台: Automation → Flow                                       │
│                                                                         │
│  API: scheduledFlowCreate mutation                             │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 三、Hydrogen 完全指南

### 3.1 什么是 Hydrogen？

```
┌─────────────────────────────────────────────────────────────────┐
│                    Hydrogen 框架介绍                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Hydrogen 是 Shopify 官方的 React 框架，用于构建完全自定义的   │
│  前端体验（Headless Commerce）。                                     │
│                                                                         │
│  核心特性                                                        │
│  ────────────────────────────────────────────────────────────  │
│  • 基于 React — 使用熟悉的 React 生态                        │
│  • 内置 TypeScript 类型安全                                  │
│  • 文件系统路由 — 类似 Next.js App Router                    │
│  • Vite 构建 — 快速的开发体验                                  │
│  • 内置组件 — Image, Cart, Product 等                          │
│  • Oxygen 部署 — Shopify 官方的托管平台                        │
│                                                                         │
│  与传统 Liquid 主题的区别                                        │
│  ────────────────────────────────────────────────────────────  │
│  • 完全控制前端代码和架构                                     │
│  • 通过 Storefront API 获取数据                                │
│  • 可以使用任何 React 库                                      │
│  • 更好的性能控制和 SEO 优化                                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Hydrogen 项目结构详解

```
my-hydrogen-app/
├── app/
│   ├── routes/                    # 路由目录（文件系统路由）
│   │   ├── ($locale)._index.tsx  # 首页
│   │   ├── ($locale).products.$handle.tsx  # 产品页
│   │   ├── ($locale).collections.$handle.tsx  # 系列页
│   │   ├── ($locale).cart.tsx  # 购物车
│   │   └── ($locale).account.*  # 账户相关
│   │
│   ├── components/  # React 组件
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   ├── ProductCard.tsx
│   │   └── Cart.tsx
│   │
│   ├── lib/  # 工具函数
│   │   ├── shopify.ts  # Storefront API 封装
│   │   ├── fragments.ts  # GraphQL fragments
│   │   └── seo.ts  # SEO 工具
│   │
│   ├── styles/  # 样式文件
│   │   └── tailwind.css  # Tailwind CSS
│   │
│   └── root.tsx  # 根组件
│
├── public/  # 静态资源
│
├── shopify.app.toml  # Oxygen 配置
│
├── .env  # 环境变量
│
└── package.json
```

### 3.3 Hydrogen 核心组件详解

#### 3.3.1 Image 组件

```tsx
import {Image} from '@shopify/hydrogen';

function ProductImage({src, alt, width, height}) {
  return (
    <Image
      src={src}
      alt={alt}
      width={width}
      height={height}
      loading="lazy"
      crop="center"
    />
  );
}
```

**Image 组件特性：**
- 自动格式转换（WebP/AVIF）
- 响应式图片
- 懒加载
- 裁裁控制
- CDN 优化

#### 3.3.2 Money 组件

```tsx
import {Money} from '@shopify/hydrogen';

function ProductPrice({amount, currencyCode}) {
  return (
    <Money
      withoutTrailingZeros
      data={{amount, currencyCode}}
    />
  );
}
```

#### 3.3.3 CartForm 组件

```tsx
import {CartForm} from '@shopify/hydrogen';

function AddToCart({variantId}) {
  return (
    <CartForm
      route="/cart"
      inputs={{
        lines: [{merchandiseId: variantId, quantity: 1}]
      }}
    >
      <button type="submit">Add to Cart</button>
    </CartForm>
  );
}
```

### 3.4 Hydrogen Hooks 详解

```
┌─────────────────────────────────────────────────────────────────┐
│                    Hydrogen Hooks 详解                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  useAction                                                       │
│  ────────────────────────────────────────────────────────────  │
│  用于触发 actions（表单提交、API 调用等）                    │
│                                                                         │
│  useLoader                                                       │
│  ────────────────────────────────────────────────────────────  │
│  访问路由 loader 数据（server 数据）                             │
│                                                                         │
│  useRouteLoaderData                                              │
│  ────────────────────────────────────────────────────────────  │
│  访问其他路由的 loader 数据                                   │
│                                                                         │
│  useNavigation                                                   │
│  ────────────────────────────────────────────────────────────  │
│  编程式导航                                                     │
│                                                                         │
│  useAnalytics                                                   │
│  ────────────────────────────────────────────────────────────  │
│  发送分析事件到 Analytics Provider                               │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 3.5 Oxygen 部署详解

```
┌─────────────────────────────────────────────────────────────────┐
│                    Oxygen 部署详解                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  什么是 Oxygen？                                                  │
│  Oxygen 是 Shopify 官方的 Vercel/Netlify 替代品，专门用于     │
│  部署 Hydrogen 项目。                                              │
│                                                                         │
│  特性                                                            │
│  ────────────────────────────────────────────────────────────  │
│  • 全球边缘网络部署                                           │
│  • 自动 HTTPS                                                  │
│  • 环境变量管理                                              │
│  • 与 Shopify 深度集成                                        │
│  • 自动 CI/CD                                                    │
│                                                                         │
│  部署命令                                                        │
│  ────────────────────────────────────────────────────────────  │
│  npm run deploy                                                  │
│                                                                         │
│  环境变量                                                        │
│  ────────────────────────────────────────────────────────────  │
│  SHOPIFY_STORE_ID=xxx                                            │
│  SHOPFUL_STOREFRONT_API_TOKEN=xxx                               │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 四、Claude Code + Shopify 工作流

### 4.1 典型开发会话

```bash
# 1. 启动 Claude Code
claude

# 2. 自然语言描述需求
你: "帮我在首页添加一个 Newsletter 订阅组件"

# 3. Claude Code 自动:
#    - 读取首页文件结构
#    - 找到合适的位置
#    - 创建组件文件
#    - 更新导入和样式
#    - 运行类型检查

# 4. 你: "提交这次改动"
# Claude Code 自动:
#    - git add 相关文件
#    - 生成 commit message
#    - 创建 commit
```

### 4.2 Claude Code 工具能力

| 工具 | 功能 | Shopify 场景示例 |
|------|------|-------------------|
| **Read** | 读取文件内容 | 读取组件、配置、GraphQL 查询 |
| **Write** | 创建新文件 | 新建 React 组件、页面路由 |
| **Edit** | 编辑现有文件 | 修改组件逻辑、更新样式 |
| **Bash** | 执行终端命令 | `npm run dev`、`git commit`、部署 |
| **Glob** | 文件模式匹配 | 查找所有 `*.liquid` 或 `*.tsx` 文件 |
| **Grep** | 代码搜索 | 查找特定函数或变量 |
| **Task** | 启动 Agent | 复杂任务的自动化处理 |

### 4.3 Agent 系统

```
┌─────────────────────────────────────────────────────────────────┐
│                 Claude Code Agent 系统                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Agent = 专门处理某类任务的 AI 子进程                            │
│                                                                         │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐   │
│  │   Explore       │ │   Plan          │ │  Code Review   │   │
│  │   探索代码库     │ │   规划实现方案   │ │   代码审查      │   │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘   │
│                                                                         │
│  使用场景：                                                       │
│  • "帮我探索这个 Shopify 主题的结构" → Explore Agent         │
│  • "规划如何添加购物车功能" → Plan Agent                        │
│  • "审查我刚写的代码" → Code Review Agent                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 五、第三方服务接入

### 5.1 常见服务矩阵

```
┌─────────────────────────────────────────────────────────────────┐
│                   第三方服务接入矩阵                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐   │
│  │  邮件营销       │ │  数据分析        │ │  客服系统        │   │
│  │                 │ │                 │ │                 │   │
│  │  • Klaviyo     │ │  • GA4         │ │  • Gorgias     │   │
│  │  • Mailchimp   │ │  • Pixel       │ │  • Re:amaze    │   │
│  │  • Omnisend    │ │  • Hotjar      │ │  • Chatwoot   │   │
│  │                 │ │  • Clarity     │ │                 │   │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘   │
│                                                                         │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐   │
│  │  支付增强       │ │  物流配送        │ │  用户评价        │   │
│  │                 │ │                 │ │                 │   │
│  │  • Stripe      │ │  • ShipStation │ │  • Judge.me    │   │
│  │  • PayPal     │ │  • EasyPost    │ │  • Yotpo       │   │
│  │  • Afterpay    │ │  • Shippo      │ │  • Loox        │   │
│  │                 │ │                 │ │                 │   │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘   │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Klaviyo 集成详解

```
┌─────────────────────────────────────────────────────────────────┐
│                   Klaviyo API 集成详解                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  API 基础 URL                                                    │
│  https://a.klaviyo.com/                                         │
│                                                                         │
│  API 版本                                                         │
│  2024-10-15                                                      │
│                                                                         │
│  主要端点                                                         │
│  ────────────────────────────────────────────────────────────  │
│  POST /api/profile-subscription-bulk-create-jobs/               │
│    创建/更新档案并订阅到列表                                   │
│                                                                         │
│  POST /api/events/                                             │
│    创建服务端事件                                               │
│                                                                         │
│  POST /api/coupon-codes/                                        │
│    生成唯一折扣码                                              │
│                                                                         │
│  POST /api/profile-import-jobs/                                 │
│    批量更新档案属性                                            │
│                                                                         │
│  认证头                                                           │
│  Authorization: Klaviyo-API-Key {private_api_key}               │
│  revision: 2024-10-15                                       │
│  Content-Type: application/vnd.api+json                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 5.3 GA4 集成详解

```
┌─────────────────────────────────────────────────────────────────┐
│                   GA4 (Google Analytics 4) 集成详解                │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  获取 MEASUREMENT_ID                                               │
│  ────────────────────────────────────────────────────────────  │
│  1. 登录 https://analytics.google.com/                       │
│  2. 创建账号/属性                                               │
│  3. 创建媒体资源                                               │
│  4. 获取 MEASUREMENT_ID (格式: G-XXXXXXXXXX)                  │
│                                                                         │
│  前端集成                                                         │
│  ────────────────────────────────────────────────────────────  │
│  <!-- Google tag (gtag.js) -->                                   │
│  <script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script> │
│  <script>                                                       │
│    window.dataLayer = window.dataLayer || [];              │
│    function gtag(){dataLayer.push(arguments);}            │
│    gtag('js', new Date());                                      │
│  </script>                                                      │
│                                                                         │
│  电商事件                                                         │
│  ────────────────────────────────────────────────────────────  │
│  gtag('event', 'purchase', {                                       │
│    transaction_id: 'T_12345',                                     │
│    value: 50.0,                                                │
│    currency: 'USD',                                            │
│    items: [...]                                                 │
│  });                                                             │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 5.4 Chatwoot 集成详解

```
┌─────────────────────────────────────────────────────────────────┐
│                   Chatwoot 集成详解                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  什么是 Chatwoot？                                                 │
│  开源客服平台，支持多渠道（网站、邮件、WhatsApp 等）       │
│                                                                         │
│  主要特性                                                         │
│  ────────────────────────────────────────────────────────────  │
│  • 多渠道统一收件箱                                             │
│  • 实时聊天                                                    │
│  • 机器人/自动化                                               │
│  • 知识库                                                      │
│  • 团队协作                                                    │
│  • API 集成                                                     │
│                                                                         │
│  部署方式                                                         │
│  ────────────────────────────────────────────────────────────  │
│  • Self-hosted (Docker)                                       │
│  • Cloud                                                       │
│                                                                         │
│  Webhook 集成                                                      │
│  ────────────────────────────────────────────────────────────  │
│  1. 用户发送消息                                              │
│  2. Chatwoot 发送 Webhook 到你的服务器                          │
│  3. 你的服务器处理逻辑（如查询 Shopify 订单）                  │
│  4. 返回消息给 Chatwoot                                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 六、实战场景案例

### 6.1 场景一：快速创建组件

```bash
# 你的输入
"创建一个 Newsletter 订阅组件，包含邮箱输入和订阅按钮，
使用 Tailwind 样式，符合品牌色调"

# Claude Code 自动执行:
1. 读取现有组件结构
2. 创建 Newsletter.tsx
3. 添加 TypeScript 类型
4. 运行类型检查
```

### 6.2 场景二：批量操作

```bash
# 你的输入
"把所有组件中的硬编码社交链接改成从 metafield 读取"

# Claude Code 自动执行:
1. Grep 查找所有包含 "instagram.com" 的文件
2. 读取 root.tsx 中的 metafield 提取逻辑
3. 批量修改每个文件
4. 验证类型正确性
```

### 6.3 场景三：API 集成

```bash
# 你的输入
"接入 Klaviyo 订阅 API，需要处理超时和错误"

# Claude Code 自动执行:
1. 查找现有的 API 路由结构
2. 创建 /api/subscribe 路由
3. 编写 Klaviyo API 调用逻辑
4. 添加错误处理和超时控制
5. 更新前端表单组件
```

### 6.4 场景四：代码审查

```bash
# 你的输入
"审查我刚修改的文件，检查是否有安全问题"

# Claude Code 自动执行:
1. 运行 git diff 查看改动
2. 分析安全问题 (XSS、注入等)
3. 检查是否符合 Shopify 最佳实践
4. 提供改进建议
```

---

## 七、最佳实践与技巧

### 7.1 提示词技巧

| 场景 | 好的提示词 | 差的提示词 |
|------|-----------|-----------|
| 功能开发 | "在 Header 组件中添加搜索按钮，点击打开搜索弹窗，参考现有的 Aside 组件模式" | "加个搜索功能" |
| Bug 修复 | "用户反馈在 Safari 中按钮点击无响应，请检查事件绑定和兼容性" | "按钮坏了" |
| 代码优化 | "优化 Hero 组件的加载性能，考虑懒加载和代码分割" | "让代码更快" |
| API 集成 | "接入 Klaviyo Subscribe API，需要处理超时和错误，参考现有的 klaviyo.ts" | "加个邮件订阅" |

### 7.2 项目配置：CLAUDE.md

```
┌─────────────────────────────────────────────────────────────────┐
│                   CLAUDE.md 项目指南                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  CLAUDE.md 是项目的"说明书"，Claude Code 会优先读取它        │
│                                                                         │
│  典型内容：                                                       │
│  • 项目背景和品牌定位                                             │
│  • 技术栈说明                                                   │
│  • 开发规范 (Dos/Don'ts)                                         │
│  • 项目结构说明                                                   │
│  • API 集成方式                                                   │
│  • 环境变量配置                                                   │
│  • 常用命令                                                       │
│                                                                         │
│  好处：                                                           │
│  ✓ Claude Code 快速理解项目                                     │
│  ✓ 减少重复解释                                                 │
│  ✓ 团队共享知识                                                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 7.3 工作流建议

```
┌─────────────────────────────────────────────────────────────────┐
│                 Claude Code 工作流建议                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. 首次使用: 写好 CLAUDE.md                                     │
│     • 让 Claude Code 快速理解项目                                 │
│     • 减少后续沟通成本                                           │
│                                                                         │
│  2. 复杂任务: 用 Agent 系统                                       │
│     • 探索代码库 → Explore Agent                                  │
│     • 规划方案 → Plan Agent                                         │
│     • 审查代码 → Code Review Agent                                  │
│                                                                         │
│  3. 批量操作: 提供具体文件或目录                                 │
│     • "检查 sections/ 目录下所有 Liquid 模板的语法"        │
│     • "批量更新 assets/ 目录下的样式文件"                    │
│                                                                         │
│  4. Git 操作: 利用 Git 集成                                       │
│     • "提交今天的改动，commit message 要说明新增的组件"      │
│     • "创建 PR 到 production 分支"                              │
│                                                                         │
│  5. 调试问题: 提供完整上下文                                     │
│     • "首页在移动端布局错乱，我用的 iPhone 尺寸，相关文件是 theme.liquid" │
│                                                                         │
│  6. 学习项目: 让 Claude Code 解释                                   │
│     • "解释一下这个 Shopify 主题的数据流"                       │
│     • "这段 Liquid 模板代码在做什么？"                          │
│                                                                         │
└─────────────────────────────────────────────────────────────────┘
```

### 7.4 常见问题解决

| 问题 | 解决方案 |
|------|---------|
| Claude 无法理解项目 | 完善 CLAUDE.md，提供更多上下文 |
| 生成代码有语法错误 | 明确要求"运行类型检查"后再输出 |
| 不了解 Shopify 特性 | 在 CLAUDE.md 中添加 Shopify API 文档链接 |
| 修改影响其他文件 | 明确要求"检查所有受影响的文件" |

---

## 八、学习资源

| 资源 | 链接 |
|------|------|
| Claude Code 文档 | [docs.anthropic.com/claude-code](https://docs.anthropic.com/claude-code) |
| Shopify 开发文档 | [shopify.dev/docs](https://shopify.dev/docs) |
| Hydrogen 指南 | [shopify.dev/custom-storefronts](https://shopify.dev/docs/custom-storefronts) |
| GraphQL API | [shopify.dev/docs/api/admin-graphql](https://shopify.dev/docs/api/admin-graphql) |
| Liquid 语言 | [shopify.dev/docs/api/liquid](https://shopify.dev/docs/api/liquid) |
| App 开发 | [shopify.dev/docs/apps/admin-api](https://shopify.dev/docs/apps/admin-api) |

---

## 总结

**Claude Code + Shopify** 的核心价值：

1. **项目级理解** — Claude Code 懂你的整个代码库，不是一次性问答
2. **自然语言开发** — 用中文描述需求，AI 自动执行
3. **自动化流程** — 从创建组件到提交代码，一气呵成
4. **减少上下文切换** — 不用在编辑器和浏览器之间来回切换
5. **知识沉淀** — CLAUDE.md 成为项目的"活文档"

**适合人群：**
- 独立开发者
- 小型团队
- 需要快速迭代的 Shopify 项目
- 希望提高开发效率的前端开发者

**Shopify 的核心价值：**
1. **后端托管** — 支付、物流、订单、库存等重资产由 Shopify 承担
2. **前端自由** — 品牌、交互、体验等轻资产完全可控
3. **数据主控** — 第一方数据收集与分析，不依赖平台黑盒
4. **扩展灵活** — API 复用，支持多前端、多渠道

**适合团队：**
- 有稳定的前端开发资源
- 品牌差异化是核心战略
- 愿意为长期价值投入初期成本

---

*文档版本：2026-02-15*
