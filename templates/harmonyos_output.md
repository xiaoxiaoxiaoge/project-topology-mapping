# HarmonyOS/OpenHarmony 项目输出模板

```markdown
# 项目拓扑

**生成时间:** YYYY-MM-DD HH:mm
**项目名称:** <name>
**项目类型:** HarmonyOS
**主要语言:** ArkTS/TypeScript, C++
**框架:** HarmonyOS (OpenHarmony), NAPI 原生绑定

## 目录结构

```
<tree output>
```

## 模块地图

| 模块 | 类型 | 功能 |
|------|------|------|
| EntryAbility | Ability | 应用生命周期管理、权限请求 |
| Index | 页面 | 图像Deepfake检测界面 |
| MainPage | 页面 | 主页面(任务/日志/结果管理) |
| DatabaseService | 服务 | 关系型数据库(RDB)封装 |

## 组件关系图

### ArkTS 页面依赖链
```
MainPage (主控)
  ├── PhotoPickerDialog (图片选择弹窗)
  ├── VideoDetectionPage (视频检测)
  │   └── AudioPlayer (音频播放)
  ├── JpgExportPage (JPG导出)
  └── PdfExportPage (PDF导出)
```

### 服务依赖关系
```
DatabaseService (基础服务，所有服务依赖它)
  ├── TaskService
  ├── ResultService
  ├── LogService
  ├── ContactService
  └── TemplateService

SmsService (独立服务，调用HTTP API)
ExportService (导出服务，依赖Task/Result/Log)
```

### Native 绑定架构
```
Index.ets / VideoDetectionPage.ets
  └── testNapi (libdeepfake.so)
        ├── deepfake_detector.cpp (图像检测)
        └── fake_audio.cpp (音频检测)
```

## 入口点

| 页面 | 路由 | 功能 |
|------|------|------|
| MainPage | 主入口 | 任务列表/日志列表/结果列表/设置 |
| Index | 检测 | 图像Deepfake检测 |

## 依赖

- **运行时:** HarmonyOS API 9+
- **数据库:** @kit.ArkData (relationalStore)
- **HTTP:** @kit.NetworkKit (http)
- **媒体:** @kit.MediaKit, @kit.MediaLibraryKit
- **文件:** @kit.CoreFileKit, @ohos.file.fs
- **UI:** @kit.ArkUI
- **Native:** libdeepfake.so (MindSpore Lite + 自研检测模型)

## 系统 API 依赖统计

| Kit | 用途 | 引用次数 |
|-----|------|---------|
| @kit.AbilityKit | Ability生命周期、common | 9 |
| @kit.PerformanceAnalysisKit | hilog日志 | 7 |
| @kit.CoreFileKit | fileIo, picker | 10 |
| @kit.ArkData | relationalStore数据库 | 5 |
| @kit.ArkUI | router, promptAction | 6 |
| @kit.MediaKit | media, image | 6 |
| @kit.MediaLibraryKit | photoAccessHelper | 3 |
| @kit.NetworkKit | http网络 | 2 |
| @kit.BasicServicesKit | BusinessError, print | 5 |
| @kit.ArkWeb | webview | 1 |
```