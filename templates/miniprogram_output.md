# 小程序项目输出模板

> **说明**：本文件是 **Markdown 写作骨架/参考模板**，含占位符（`<name>`、`<project-name>` 等），供 AI 写作时套用。
> **不是** Jinja2/Handlebars 等程序化模板引擎格式，不能被工具直接渲染。
> 使用时将占位符替换为项目真实值，**不要直接复制示例**。

```markdown
# 小程序项目拓扑 - [项目名称]

**生成时间:** YYYY-MM-DD HH:mm
**项目类型:** 小程序 (微信 / 支付宝 / 抖音 / 百度 / 跨端)
**主要语言:** JavaScript / TypeScript / WXML / WXSS / Vue / React
**框架:** 原生 / Taro / uni-app / Remax

## 1. 基础信息

| 项 | 值 |
|----|---|
| 目标平台 | 微信 / 支付宝 / 抖音 / 百度 / 多端 |
| AppID | wx... / my... |
| 入口页面 | pages/index/index |

## 2. 目录结构

```
<tree output>
```

## 3. 模块地图

| 模块 | 页面数 | 组件数 | API 路径前缀 |
|------|-------|-------|------------|
| 首页 | 3 | 5 | /api/home/* |
| 商城 | 12 | 18 | /api/mall/* |
| 个人 | 5 | 6 | /api/user/* |

## 4. 页面路径表

| 路径 | 文件 | 用途 |
|------|------|------|
| pages/index/index | index.{js,ts,wxml,wxss} | 首页 |
| pages/mall/list | list.{js,ts} | 商品列表 |
| pages/user/profile | profile.{js,ts} | 个人中心 |

## 5. 组件关系图

### 5.1 自定义组件
```
src/components/
├── goods-card/        # 商品卡片 (引用 12 次)
├── user-avatar/       # 用户头像 (引用 8 次)
└── empty-state/       # 空状态 (引用 5 次)
```

### 5.2 组件被引用 Top 10
| 组件 | 被引用次数 | 引用方 |
|------|----------|--------|
| goods-card | 12 | mall/list, mall/detail, ... |
| user-avatar | 8 | user/profile, comment/list, ... |

## 6. 跨端差异表

| API 场景 | 微信 (wx.*) | 支付宝 (my.*) | 抖音 (tt.*) | 跨端方案 |
|---------|------------|--------------|------------|---------|
| 网络请求 | `wx.request` | `my.request` | `tt.request` | uni.request |
| 存储 | `wx.setStorage` | `my.setStorage` | `tt.setStorage` | uni.setStorage |
| 跳转 | `wx.navigateTo` | `my.navigateTo` | `tt.navigateTo` | uni.navigateTo |
| 登录 | `wx.login` | `my.getAuthCode` | `tt.login` | uni.login |
| 支付 | `wx.requestPayment` | `my.tradePay` | `tt.pay` | uni.requestPayment |

## 7. 数据流

### 7.1 页面生命周期
```
onLoad → onShow → onReady → 用户操作 → onHide → onUnload
```

### 7.2 全局数据
```
app.js: globalData
  ├── userInfo: { nickName, avatarUrl, ... }
  ├── systemInfo: { ... }
  └── version: '1.0.0'

App 启动时初始化 → 各页面 onLoad 时读取
```

### 7.3 网络请求流
```
页面调用 wx.request / uni.request
  ↓
工具层 request.js (统一拦截)
  ↓
后端 API
  ↓
响应拦截器 (token 刷新 / 错误处理)
  ↓
业务回调
```

## 8. 入口点

| 平台 | 工具 | 步骤 |
|------|------|------|
| 微信 | 微信开发者工具 | 导入项目 → 输入 AppID → 编译 |
| 支付宝 | 小程序开发者工具 | 同上 |
| 抖音 | 抖音开发者工具 | 同上 |
| Taro | `npm run dev:weapp` | 编译到 dist/ → 用微信开发者工具导入 |
| uni-app | `npm run dev:mp-weixin` | 同上 |

## 9. 依赖列表

- 框架: 原生 / Taro 3.X / uni-app X
- 状态管理: Mobx / Redux / 自研
- UI 库: Vant Weapp / TDesign / NutUI
- 网络: wx.request / flyio / axios
- 工具: dayjs / lodash

## 10. 定位速查表

| 页面/功能 | 文件 | 调用 API | 后端服务 |
|---------|------|---------|---------|
| 登录 | pages/login/index | /api/auth/login | user-service |
| 商品列表 | pages/mall/list | /api/mall/list | mall-service |
| 购物车 | pages/cart/index | /api/cart/list | cart-service |
```
