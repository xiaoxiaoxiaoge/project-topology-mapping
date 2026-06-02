# 移动端项目输出模板

> **说明**：本文件是 **Markdown 写作骨架/参考模板**，含占位符（`<name>`、`<project-name>` 等），供 AI 写作时套用。
> **不是** Jinja2/Handlebars 等程序化模板引擎格式，不能被工具直接渲染。
> 使用时将占位符替换为项目真实值，**不要直接复制示例**。

```markdown
# 移动端项目拓扑 - [项目名称]

**生成时间:** YYYY-MM-DD HH:mm
**项目类型:** 移动端 (Android / iOS / React Native / Flutter)
**主要语言:** Kotlin/Java / Swift/ObjC / TypeScript / Dart
**框架:** ...

## 1. 基础信息

| 项 | 值 |
|----|---|
| 平台 | Android / iOS / RN / Flutter |
| 最低系统版本 | Android X / iOS X |
| 包名 / Bundle ID | com.example.xxx |
| 入口 Activity / ViewController | MainActivity / MainViewController |

## 2. 目录结构

```
<tree output>
```

## 3. 模块地图

### Android
| 模块 | 包路径 | Activity 数 | Fragment 数 |
|------|--------|------------|------------|
| 用户模块 | com.example.user | 2 | 3 |
| 订单模块 | com.example.order | 1 | 2 |

### iOS
| 模块 | 路径 | ViewController 数 | Storyboard 数 |
|------|------|------------------|--------------|
| 用户模块 | Sources/User/ | 3 | 1 |
| 订单模块 | Sources/Order/ | 2 | 1 |

### RN / Flutter
| 模块 | 路径 | Screen 数 | Widget 数 |
|------|------|---------|----------|
| 用户模块 | lib/screens/user/ | 4 | 8 |
| 订单模块 | lib/screens/order/ | 3 | 6 |

## 4. 组件关系图

### 4.1 页面导航图
```
HomeScreen
  ├── LoginScreen (未登录时跳转)
  ├── ProfileScreen
  └── SettingsScreen
```

### 4.2 状态管理
```
- ViewModel / Provider / Riverpod / Bloc
- 全局: UserSession / AppConfig
- 局部: ListState / DetailState
```

### 4.3 原生模块调用表（RN/Flutter）

| 模块 | 调用方 | 平台 | 用途 |
|------|-------|------|------|
| CameraModule | CameraScreen | iOS/Android | 调用相机 |
| FilePicker | UploadScreen | iOS/Android | 选择文件 |

## 5. 数据流

### 5.1 页面渲染流
```
Screen → Widget Tree → State → Render
```

### 5.2 状态管理流
```
用户操作 → ViewModel/Bloc → State → UI 重渲染
                  ↓
                Repository → API/DB
```

### 5.3 路由守卫流
```
路由 → 拦截器 → 登录态校验 → 放行/重定向到 Login
```

## 6. 入口点

| 命令 | 用途 | 端口 |
|------|------|------|
| `./gradlew installDebug` | Android 安装到设备 | adb 5037 |
| `xcodebuild` | iOS 编译 | - |
| `npx react-native run-android` | RN 启动 Android | Metro 8081 |
| `flutter run` | Flutter 启动 | Observatory 8181 |

## 7. 依赖列表

### Android
- 框架: Android SDK X / Kotlin X
- UI: Jetpack Compose / Material Design
- 网络: Retrofit / OkHttp
- 数据库: Room / SQLite
- 异步: Coroutine / RxJava

### iOS
- 框架: UIKit / SwiftUI
- 依赖管理: CocoaPods / SPM
- 网络: Alamofire / URLSession
- 持久化: CoreData / Realm

### RN/Flutter
- 框架: React Native 0.7X / Flutter 3.X
- 状态: Redux / Provider / Riverpod / Bloc
- 网络: axios / Dio

## 8. 定位速查表

| 页面 | 路由 | 核心组件 | 后端 API |
|------|------|---------|---------|
| 登录 | /login | LoginScreen, LoginForm | /api/auth/login |
| 首页 | /home | HomeScreen, BannerList | /api/home/feed |
| 个人中心 | /profile | ProfileScreen, UserInfo | /api/user/profile |

## 9. 同类页面差异（高度/极度差异时必填）

### [模块名] 页面差异分析

| 页面 | 路由 | 核心组件 | 状态管理 | 差异原因 |
|------|------|---------|---------|---------|
| Home | / | HomeScreen | useHomeStore | 默认入口 |
| Profile | /profile | ProfileScreen | useUserStore | 需要登录 |
```
