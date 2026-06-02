# 桌面端项目输出模板

> **说明**：本文件是 **Markdown 写作骨架/参考模板**，含占位符（`<name>`、`<project-name>` 等），供 AI 写作时套用。
> **不是** Jinja2/Handlebars 等程序化模板引擎格式，不能被工具直接渲染。
> 使用时将占位符替换为项目真实值，**不要直接复制示例**。

```markdown
# 桌面端项目拓扑 - [项目名称]

**生成时间:** YYYY-MM-DD HH:mm
**项目类型:** 桌面端 (Electron / Tauri / Flutter Desktop / Qt / WPF)
**主要语言:** JavaScript/TypeScript / Rust / Dart / C++ / C#
**框架:** ...

## 1. 基础信息

| 项 | 值 |
|----|---|
| 框架 | Electron / Tauri / Flutter Desktop / ... |
| 支持平台 | Windows / macOS / Linux |
| 入口 (Main) | main.js / main.rs / main.go |
| 入口 (Renderer) | index.html |

## 2. 目录结构

```
<tree output>
```

## 3. 进程模型

### Electron

```
┌──────────────────────────────────────┐
│ Main Process (Node.js)               │
│   ├── BrowserWindow 管理              │
│   ├── ipcMain.handle (IPC handlers)   │
│   ├── 系统 API (dialog/menu/tray)     │
│   └── 网络请求 (主进程或 worker)        │
└──────────────┬───────────────────────┘
               │ IPC (invoke/send)
               ↓
┌──────────────────────────────────────┐
│ Renderer Process (Chromium)          │
│   ├── UI (React/Vue/Vanilla)         │
│   ├── ipcRenderer.invoke              │
│   └── 业务逻辑                        │
└──────────────┬───────────────────────┘
               │ contextBridge
               ↓
┌──────────────────────────────────────┐
│ Preload Script (安全桥)                │
│   └── contextBridge.exposeInMainWorld │
└──────────────────────────────────────┘
```

### Tauri

```
┌──────────────────────────────────────┐
│ Frontend (Webview)                    │
│   └── invoke('cmd_name', args)        │
└──────────────┬───────────────────────┘
               │ Tauri IPC
               ↓
┌──────────────────────────────────────┐
│ Rust Backend (Tauri)                  │
│   ├── #[tauri::command] handlers     │
│   ├── State (Arc<Mutex<...>>)         │
│   ├── 事件系统 (emit/listen)          │
│   └── 系统 API (path/dialog/shell)    │
└──────────────────────────────────────┘
```

## 4. IPC 通道清单

| 通道名 | 方向 | 触发方 | 处理方 | 数据 | 用途 |
|--------|------|--------|--------|------|------|
| `app:open-file` | Renderer → Main | FileMenu | main.ts:openFileHandler | filePath | 打开文件 |
| `file:read` | Renderer → Main | Editor | main.ts:readFile | filePath | 读取文件 |
| `config:get` | Renderer → Main | Settings | main.ts:getConfig | - | 读取配置 |
| `file:changed` | Main → Renderer | Watcher | renderer.ts:onFileChange | {path, content} | 文件变更通知 |

## 5. 窗口/标签管理

| 窗口 | 类型 | 加载页面 | 关闭行为 |
|------|------|---------|---------|
| 主窗口 | BrowserWindow | index.html | 关闭时退出 |
| 设置窗口 | BrowserWindow (modal) | settings.html | 关闭时回到主窗口 |
| 关于窗口 | BrowserWindow (modal) | about.html | 关闭时回到主窗口 |

## 6. 模块地图

### Electron

| 模块 | 进程 | 文件 | 职责 |
|------|------|------|------|
| 主进程 | Main | main/* | 应用生命周期、窗口、IPC |
| 预加载 | Preload | preload.ts | 暴露安全 API |
| 渲染 | Renderer | src/renderer/* | UI、业务逻辑 |
| 共享 | - | shared/* | 类型定义、工具 |

### Tauri

| 模块 | 端 | 文件 | 职责 |
|------|----|------|------|
| 前端 | Webview | src/* | UI |
| 命令 | Rust | src-tauri/src/commands/* | 业务逻辑 |
| 状态 | Rust | src-tauri/src/state.rs | 全局状态 |
| 事件 | Rust | src-tauri/src/events.rs | 事件总线 |

## 7. 数据流

### 7.1 完整调用链 (Electron 示例)

```
用户点击 "打开文件" 按钮
  ↓
渲染端: ipcRenderer.invoke('file:open-dialog')
  ↓
preload: contextBridge.exposeInMainWorld('api.openFileDialog', ...)
  ↓
主进程: ipcMain.handle('file:open-dialog', dialog.showOpenDialog)
  ↓
主进程: 拿到 filePath → 读取内容 → 返回
  ↓
渲染端: 拿到 content → 更新 UI
```

### 7.2 状态管理

```
主进程: electron-store / SQLite (持久化)
  ↓
ipcMain.handle('config:get', ...)
  ↓
Renderer: useConfig() hook (Zustand / Redux / Vuex)
  ↓
UI 订阅 store
```

## 8. 系统 API 使用

| API | 用途 | 文件 |
|-----|------|------|
| dialog.showOpenDialog | 打开文件选择 | main/files.ts |
| Menu.buildFromTemplate | 菜单栏 | main/menu.ts |
| Tray | 系统托盘 | main/tray.ts |
| shell.openExternal | 外部链接 | main/shell.ts |
| clipboard | 剪贴板 | main/clipboard.ts |

## 9. 入口点

| 框架 | 启动命令 | 调试 | 打包 |
|------|---------|------|------|
| Electron | `npm start` | DevTools | `npm run build` (electron-builder) |
| Tauri | `cargo tauri dev` | rust-analyzer | `cargo tauri build` |
| Flutter Desktop | `flutter run -d macos` | DevTools | `flutter build macos` |
| Wails | `wails dev` | Delve | `wails build` |

## 10. 依赖列表

### Electron
- electron 28.x
- electron-builder / electron-forge
- electron-store (持久化)
- React/Vue/Svelte (渲染框架)
- IPC 类型: 自研 / typesafe-ipc

### Tauri
- tauri 1.x / 2.x
- 前端框架: Vue / React / Svelte
- 状态: tauri-plugin-store
- 序列号: serde / bincode

## 11. 同类窗口差异（高度/极度差异时必填）

| 窗口 | 加载页 | IPC 数量 | 特殊功能 |
|------|--------|---------|---------|
| 主窗口 | index.html | 15 | 完整功能 |
| 设置窗口 | settings.html | 3 | 仅设置 |
```
