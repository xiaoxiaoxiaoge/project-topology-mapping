git grep -E -h "|# 小程序项目检测命令

> Single source of truth for mini-program scanning.
> 适用：微信 / 支付宝 / 抖音 / 百度 / QQ 小程序，以及跨平台（uni-app、Taro、Remax 等）。

---

## 项目类型识别

```bash
# 微信小程序
ls -la app.json project.config.json 2>/dev/null
ls -la miniprogram/ pages/ 2>/dev/null

# 支付宝小程序
ls -la app.json app.acss mini.project.json 2>/dev/null

# 抖音小程序
ls -la project.config.json app.json 2>/dev/null
git grep -h "git grep -E -h "|tt\."git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null | head -n 1

# 百度小程序
ls -la project.swan.json 2>/dev/null
git grep -h "git grep -E -h "|swan\."git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null | head -n 1

# QQ 小程序
git grep -h "git grep -E -h "|qq\."git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null | head -n 1

# 跨平台 (Taro / uni-app / Remax)
ls -la project.config.json 2>/dev/null            # Taro
ls -la src/manifest.json 2>/dev/null              # uni-app
ls -la package.json tsconfig.json 2>/dev/null     # Remax
```

| 特征文件 / API 前缀 | 项目类型 |
|-------------------|---------|
| `app.json` + `wx.` 前缀 | 微信小程序 |
| `app.acss` + `my.` 前缀 | 支付宝小程序 |
| `project.config.json` + `tt.` 前缀 | 抖音小程序 |
| `project.swan.json` + `swan.` 前缀 | 百度小程序 |
| `qq.` 前缀 | QQ 小程序 |
| `tarojs` 依赖 | Taro |
| `@dcloudio/uni-` 依赖 | uni-app |
| `remax` 依赖 | Remax |

---

## 微信小程序检测命令

### 目录结构

```bash
# 1. 项目配置
cat app.json project.config.json 2>/dev/null

# 2. 所有页面
cat app.json | python -c "git grep -E -h "|import json,sys; print('\n'.join(json.load(sys.stdin)['pages']))"git grep -E -h "| 2>/dev/null

# 3. JS / TS / WXML / WXSS / JSON 文件
git ls-files 'miniprogram/**/*.js' 'miniprogram/**/*.ts' 2>/dev/null | head -n 150
git ls-files 'miniprogram/**/*.wxml' 2>/dev/null
git ls-files 'miniprogram/**/*.wxss' 2>/dev/null
git ls-files 'miniprogram/**/*.json' 2>/dev/null

# 4. 自定义组件
git ls-files 'miniprogram/components/**/*.js' 'miniprogram/components/**/*.json' 2>/dev/null

# 5. 工具库
git ls-files 'miniprogram/utils/*' 'miniprogram/lib/*' 2>/dev/null
```

### 关键特性

```bash
# 1. 全局 app.js 入口
cat miniprogram/app.js 2>/dev/null | head -n 30

# 2. 全局 app.wxss
cat miniprogram/app.wxss 2>/dev/null | head -n 20

# 3. 自定义组件列表（从组件目录或引用关系提取）
git ls-files 'miniprogram/components/*/index.json' 2>/dev/null

# 4. 组件被引用情况
git grep -h "git grep -E -h "|usingComponents"git grep -E -h "| -- '*.json' 2>/dev/null | head -n 30

# 5. API 调用（wx.*）
git grep -h "git grep -E -h "|wx\."git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null | \
  sed 's/.*\(wx\.\)\([a-zA-Z]*\).*/\1\2/g' | sort | uniq -c | sort -rn | head -n 30

# 6. 第三方库
cat package.json 2>/dev/null | head -n 40

# 7. 云开发
git grep -h "git grep -E -h "|wx.cloud\|wx-server-sdk"git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null

# 8. 插件 plugin.json
git ls-files 'plugin.json' 'plugin/*/plugin.json' 2>/dev/null
```

### 数据流

```bash
# 1. 网络请求封装
cat miniprogram/utils/request.js 2>/dev/null
cat miniprogram/api/index.js 2>/dev/null

# 2. 状态管理（redux / mobx / zustand 在小程序中的用法）
git grep -h "git grep -E -h "|wx.setStorage\|wx.getStorage\|wx.removeStorage"git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null | head -n 20

# 3. 全局数据 globalData
git grep -h "git grep -E -h "|globalData\."git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null | head -n 20

# 4. 事件总线
git grep -h "git grep -E -h "|getEventBus\|getApp().globalData\|wx.publishHandler"git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null
```

### 差异点检测

```bash
# 1. 页面生命周期使用情况
git grep -h "git grep -E -h "|onLoad\|onShow\|onReady\|onHide\|onUnload\|onPullDownRefresh\|onReachBottom"git grep -E -h "| \
  -- 'miniprogram/pages/**/*.js' 'miniprogram/pages/**/*.ts' 2>/dev/null | sort | uniq -c | sort -rn

# 2. 各页面的 API 调用（差异表）
for page in $(git ls-files 'miniprogram/pages/*/index.js' 2>/dev/null); do
  echo "git grep -E -h "|=== $page ==="git grep -E -h "|
  git grep -h "git grep -E -h "|wx\."git grep -E -h "| "git grep -E -h "|$page"git grep -E -h "| 2>/dev/null
done

# 3. 自定义组件使用差异
git grep -h "git grep -E -h "|<[A-Z]"git grep -E -h "| -- 'miniprogram/**/*.wxml' 2>/dev/null | \
  sed 's/.*<\([A-Z][a-zA-Z]*\).*/\1/g' | sort | uniq -c | sort -rn | head -n 20

# 4. WXML 模板引用
git grep -h "git grep -E -h "|<import\|<include"git grep -E -h "| -- 'miniprogram/**/*.wxml' 2>/dev/null

# 5. 行为统计 / 埋点
git grep -h "git grep -E -h "|wx.reportMonitor\|wx.reportEvent\|wx.getExptInfoSync"git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null
```

### 阶段 7 检查

```bash
# 同类页面模块差异
git ls-files 'miniprogram/pages/*/index.js' 2>/dev/null

# 跨端差异代码
git grep -h "git grep -E -h "|process.env.TARO_ENV\|#ifdef MP-WEIXIN\|#ifdef MP-ALIPAY"git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null
```

---

## 支付宝小程序检测命令

```bash
# 1. 配置
cat app.json app.acss mini.project.json 2>/dev/null

# 2. 页面（my.* API）
git grep -h "git grep -E -h "|my\."git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null | \
  sed 's/.*\(my\.\)\([a-zA-Z]*\).*/\1\2/g' | sort | uniq -c | sort -rn | head -n 30

# 3. 跨端差异
git grep -h "git grep -E -h "|my\.\|wx\."git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null | \
  sed 's/.*\(my\.\|wx\.\)\([a-zA-Z]*\).*/\1\2/g' | sort | uniq -c | sort -rn

# 4. 组件库
git grep -h "git grep -E -h "|<[a-z]-[a-z]"git grep -E -h "| -- '*.axml' 2>/dev/null | head -n 20
```

---

## 抖音 / 百度 / QQ 小程序检测命令

```bash
# 抖音
git grep -h "git grep -E -h "|tt\."git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null | \
  sed 's/.*\(tt\.\)\([a-zA-Z]*\).*/\1\2/g' | sort | uniq -c | sort -rn | head -n 20

# 百度
git grep -h "git grep -E -h "|swan\."git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null | \
  sed 's/.*\(swan\.\)\([a-zA-Z]*\).*/\1\2/g' | sort | uniq -c | sort -rn | head -n 20

# QQ
git grep -h "git grep -E -h "|qq\."git grep -E -h "| -- '*.js' '*.ts' 2>/dev/null | \
  sed 's/.*\(qq\.\)\([a-zA-Z]*\).*/\1\2/g' | sort | uniq -c | sort -rn | head -n 20
```

---

## 跨平台框架 (Taro / uni-app / Remax)

### Taro

```bash
# 1. 项目配置
cat project.config.json config/index.js 2>/dev/null

# 2. 页面
git ls-files 'src/pages/*' 2>/dev/null

# 3. 跨端条件
git grep -h "git grep -E -h "|process.env.TARO_ENV"git grep -E -h "| -- '*.tsx' '*.jsx' '*.vue' 2>/dev/null | head -n 20

# 4. Taro 组件使用
git grep -h "git grep -E -h "|from ['\"git grep -E -h "|]@tarojs"git grep -E -h "| -- '*.tsx' '*.jsx' '*.vue' 2>/dev/null | sort | uniq -c | sort -rn
```

### uni-app

```bash
# 1. manifest
cat src/manifest.json pages.json 2>/dev/null

# 2. 跨端条件
git grep -h "git grep -E -h "|#ifdef MP-WEIXIN\|#ifdef H5\|#ifdef APP-PLUS\|#ifdef MP-ALIPAY"git grep -E -h "| -- '*.vue' 2>/dev/null

# 3. uni API
git grep -h "git grep -E -h "|uni\."git grep -E -h "| -- '*.js' '*.ts' '*.vue' 2>/dev/null | \
  sed 's/.*\(uni\.\)\([a-zA-Z]*\).*/\1\2/g' | sort | uniq -c | sort -rn | head -n 20
```

### Remax

```bash
# 1. 配置
cat package.json tsconfig.json 2>/dev/null

# 2. 跨端渲染
git ls-files 'src/pages/*' 2>/dev/null

# 3. 平台 API 抽象
git grep -h "git grep -E -h "|import { useRemax" -- '*.tsx' '*.ts' 2>/dev/null | sort -u
```

---

## 跨平台差异表（阶段 7 必填）

```markdown
### 跨端 API 差异分析

| API 场景 | 微信 (wx.*) | 支付宝 (my.*) | 抖音 (tt.*) | 百度 (swan.*) | 跨端方案 |
|---------|------------|--------------|------------|-------------|---------|
| 网络请求 | `wx.request` | `my.request` | `tt.request` | `swan.request` | uni.request / Taro.request |
| 存储 | `wx.setStorage` | `my.setStorage` | `tt.setStorage` | `swan.setStorage` | uni.setStorage |
| 跳转 | `wx.navigateTo` | `my.navigateTo` | `tt.navigateTo` | `swan.navigateTo` | uni.navigateTo |
| 登录 | `wx.login` | `my.getAuthCode` | `tt.login` | `swan.login` | uni.login |
| 支付 | `wx.requestPayment` | `my.tradePay` | `tt.pay` | `swan.requestPolymerPayment` | uni.requestPayment |
```

---

## 入口点

| 平台 | 命令 | 工具 |
|------|------|------|
| 微信 | `微信开发者工具 → 导入项目` | 微信开发者工具 |
| 支付宝 | `小程序开发者工具 → 导入项目` | 支付宝开发者工具 |
| 抖音 | `抖音开发者工具 → 导入项目` | 抖音开发者工具 |
| 百度 | `百度开发者工具 → 导入项目` | 百度开发者工具 |
| Taro | `npm run dev:weapp` | 各家开发者工具 |
| uni-app | `npm run dev:mp-weixin` | 微信开发者工具 |
| uni-app (H5) | `npm run dev:h5` | 浏览器 |
