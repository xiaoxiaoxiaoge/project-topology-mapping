# 桌面端项目检测命令

> Single source of truth for desktop application scanning.
> 适用：Electron / Tauri / Flutter Desktop / Qt / WPF / Wails。

---

## 项目类型识别

```bash
# Electron
ls -la package.json 2>/dev/null
git grep -h "electron\b" -- 'package.json' 2>/dev/null | head -n 3
ls -la main.js main.ts electron.js 2>/dev/null

# Tauri
ls -la src-tauri/ tauri.conf.json 2>/dev/null
cat src-tauri/Cargo.toml 2>/dev/null | head -n 20

# Flutter Desktop
ls -la pubspec.yaml 2>/dev/null
git grep -h "flutter:" -- 'pubspec.yaml' 2>/dev/null | head -n 3
git grep -h "desktop:" -- 'pubspec.yaml' 2>/dev/null | head -n 3

# Qt (C++)
ls -la *.pro CMakeLists.txt *.qrc 2>/dev/null

# WPF / WinForms (.NET)
ls -la *.csproj *.sln 2>/dev/null
git ls-files '*.xaml' 2>/dev/null | head -n 5

# Wails (Go + Web)
ls -la wails.json go.mod 2>/dev/null
git ls-files 'frontend/*' 2>/dev/null | head -n 5
```

| 特征文件 | 项目类型 |
|---------|---------|
| `package.json` + `electron` | Electron |
| `src-tauri/Cargo.toml` + `tauri.conf.json` | Tauri |
| `pubspec.yaml` + `desktop:` | Flutter Desktop |
| `*.pro` / `CMakeLists.txt` | Qt |
| `*.csproj` + `*.xaml` | WPF |
| `wails.json` + `go.mod` | Wails |

---

## Electron 检测命令

### 目录结构

```bash
# 1. main 进程入口
ls main.js main.ts electron.js electron-main.js 2>/dev/null
git ls-files 'main/*' 'electron/*' 2>/dev/null | head -n 20

# 2. renderer 进程
git ls-files 'renderer/*' 'src/renderer/*' 'src/*' 2>/dev/null | head -n 100

# 3. preload 脚本
git ls-files 'preload*' 'src/preload/*' 2>/dev/null

# 4. 配置
cat package.json 2>/dev/null | head -n 50
ls electron-builder.yml electron-forge.config.js 2>/dev/null

# 5. 资源
ls assets/ resources/ build/ 2>/dev/null

# 6. 打包配置
git ls-files 'package.json' 'electron-builder*' 'electron-forge*' 2>/dev/null
```

### 关键特性

```bash
# 1. 进程模型
git grep -h "BrowserWindow\|app.whenReady\|app.on" -- 'main/*' 'main.js' 'main.ts' 2>/dev/null | head -n 20

# 2. IPC 通道（Main 端）
git grep -h "ipcMain\.handle\|ipcMain\.on(" -- 'main/*' '*.js' '*.ts' 2>/dev/null

# 3. IPC 通道（Renderer 端）
git grep -h "ipcRenderer\.invoke\|ipcRenderer\.send" -- 'renderer/*' 'src/*' 2>/dev/null

# 4. preload 暴露的 API
git ls-files 'preload*' 2>/dev/null
git grep -h "contextBridge\.exposeInMainWorld" -- 'preload*' 2>/dev/null

# 5. 窗口管理
git grep -h "new BrowserWindow\|loadFile\|loadURL" -- 'main/*' '*.js' '*.ts' 2>/dev/null | head -n 20

# 6. 菜单 / 托盘
git grep -h "Menu\.buildFromTemplate\|Tray\|nativeImage" -- 'main/*' '*.js' '*.ts' 2>/dev/null

# 7. 系统集成
git grep -h "dialog\.\|shell\.\|clipboard\." -- 'main/*' '*.js' '*.ts' 2>/dev/null

# 8. 自动更新
git grep -h "autoUpdater\|electron-updater" -- 'package.json' 'main/*' 2>/dev/null

# 9. 协议注册
git grep -h "registerFileProtocol\|setAsDefaultProtocolClient" -- 'main/*' 2>/dev/null
```

### 数据流

```bash
# 1. Main 端网络请求（通常在主进程或 worker）
git grep -h "fetch\|axios\|http\." -- 'main/*' 2>/dev/null | head -n 20

# 2. 渲染端 → 主端 → 后端 典型链路
git grep -h "ipcRenderer\.invoke" -- 'src/*' 2>/dev/null
git grep -h "ipcMain\.handle" -- 'main/*' 2>/dev/null

# 3. 状态持久化（electron-store / sqlite / lowdb）
git grep -h "electron-store\|better-sqlite3\|lowdb\|nedb" -- 'package.json' 2>/dev/null
```

### 差异点检测

```bash
# 1. BrowserWindow 数量（多窗口应用 vs 单窗口）
git grep -h "new BrowserWindow" -- 'main/*' 2>/dev/null

# 2. 各窗口加载的页面
git grep -B1 -A1 "new BrowserWindow" -- 'main/*' 2>/dev/null

# 3. 菜单项
git grep -h "label:" -- 'main/*' 2>/dev/null | head -n 30

# 4. IPC handler 数量
git grep -c "ipcMain\.handle" -- 'main/*' 2>/dev/null

# 5. preload 暴露的 API 数量
git grep -c "contextBridge\.exposeInMainWorld" -- 'preload*' 2>/dev/null

# 6. 不同进程的依赖
git grep -h "from ['\"]electron" -- 'main/*' 'src/*' 2>/dev/null | sort -u | head -n 20
```

### 阶段 7 检查

```bash
# 主进程 vs 渲染进程的职责划分
git grep -l "BrowserWindow\|ipcMain" -- 'main/*' 2>/dev/null
git grep -l "ipcRenderer\|document\." -- 'src/*' 'renderer/*' 2>/dev/null
```

---

## Tauri 检测命令

### 目录结构

```bash
# 1. Rust 端源码
git ls-files 'src-tauri/src/*.rs' 2>/dev/null | head -n 50

# 2. 前端源码
git ls-files 'src/*' 2>/dev/null | head -n 100
git ls-files 'dist/*' 'build/*' 2>/dev/null | head -n 20

# 3. 配置
cat src-tauri/tauri.conf.json 2>/dev/null
cat src-tauri/Cargo.toml 2>/dev/null

# 4. tauri 命令清单
git grep -h "#\[tauri::command\]" -- 'src-tauri/src/*.rs' 2>/dev/null
```

### 关键特性

```bash
# 1. Tauri 命令（Rust 端）
git grep -B1 "fn " -- 'src-tauri/src/*.rs' 2>/dev/null | \
  grep -A1 "tauri::command" | head -n 30

# 2. 前端调用 Tauri
git grep -h "invoke(" -- 'src/*' 2>/dev/null | head -n 20

# 3. 事件系统
git grep -h "emit\|listen" -- 'src-tauri/src/*.rs' 'src/*' 2>/dev/null | head -n 20

# 4. 窗口管理
git grep -h "WindowBuilder\|window\.set_title\|window\." -- 'src-tauri/src/*.rs' 2>/dev/null | head -n 20

# 5. 状态管理插件
git grep -h "tauri-plugin-" -- 'src-tauri/Cargo.toml' 2>/dev/null

# 6. 系统集成
git grep -h "tauri::api::path\|tauri::api::dialog\|tauri::api::shell" \
  -- 'src-tauri/src/*.rs' 2>/dev/null | head -n 20
```

### 差异点检测

```bash
# 1. Tauri Command 列表
git grep -A1 "#\[tauri::command\]" -- 'src-tauri/src/*.rs' 2>/dev/null | \
  grep "pub fn\|pub async fn" | sort -u

# 2. 前端 invoke 调用方
git grep -h "invoke(['\"]" -- 'src/*' 2>/dev/null | sort -u | head -n 20

# 3. Rust 端与前端的依赖关系
git ls-files 'src-tauri/src/*.rs' 2>/dev/null
git ls-files 'src/*.tsx' 'src/*.ts' 2>/dev/null

# 4. 持久化（tauri-plugin-store / sqlite）
git grep -h "tauri-plugin-store\|rusqlite\|sqlx" -- 'src-tauri/Cargo.toml' 2>/dev/null
```

---

## Flutter Desktop 检测命令

```bash
# 1. 桌面端配置
git grep -A5 "flutter:" -- 'pubspec.yaml' 2>/dev/null
git grep -A3 "desktop:" -- 'pubspec.yaml' 2>/dev/null

# 2. 平台判断
git grep -h "Platform.isWindows\|Platform.isMacOS\|Platform.isLinux" -- 'lib/*.dart' 'lib/**/*.dart' 2>/dev/null

# 3. 窗口管理（window_manager）
git grep -h "WindowManager\|windowManager" -- 'lib/*.dart' 2>/dev/null

# 4. 系统集成
git grep -h "dart:io\|Process\.run\|File\(" -- 'lib/*.dart' 2>/dev/null | sort -u | head -n 20

# 5. 菜单 / 托盘
git grep -h "MenuBar\|SystemTray" -- 'lib/*.dart' 2>/dev/null
```

---

## Qt / WPF / 其他

### Qt (C++)

```bash
# 1. 项目文件
cat *.pro CMakeLists.txt 2>/dev/null

# 2. 头文件 / 源文件
git ls-files '*.h' '*.hpp' '*.cpp' 2>/dev/null | head -n 50

# 3. UI 文件
git ls-files '*.ui' '*.qrc' 2>/dev/null

# 4. 信号槽
git grep -h "connect(\|Q_OBJECT\|signals:" -- '*.cpp' '*.h' 2>/dev/null | head -n 20

# 5. QML
git ls-files '*.qml' 2>/dev/null
```

### WPF (.NET)

```bash
# 1. XAML 文件
git ls-files '*.xaml' 2>/dev/null

# 2. 视图模型
git ls-files '*ViewModel.cs' 'ViewModels/*' 2>/dev/null

# 3. 命令
git grep -h "ICommand\|RelayCommand" -- '*.cs' 2>/dev/null | head -n 20

# 4. 数据绑定
git grep -h "{Binding" -- '*.xaml' 2>/dev/null | head -n 20
```

### Wails (Go + Web)

```bash
# 1. Go 端
cat main.go app.go 2>/dev/null

# 2. Wails 绑定方法
git grep -h "func.*ctx context\.Context" -- '*.go' 2>/dev/null | head -n 20

# 3. 前端
git ls-files 'frontend/*' 2>/dev/null

# 4. 跨端调用
git grep -h "window\.go\." -- 'frontend/*' 2>/dev/null
```

---

## 桌面端 IPC 通道表（阶段 4 输出模板必填）

```markdown
### IPC 通道清单

| 通道名 | 方向 | 触发方 | 处理方 | 数据 |
|--------|------|--------|--------|------|
| `app:open-file` | Renderer → Main | FileMenu | main.ts:openFileHandler | filePath |
| `file:read` | Renderer → Main | Editor | main.ts:readFile | filePath |
| `config:get` | Renderer → Main | Settings | main.ts:getConfig | - |
```

---

## 入口点

| 框架 | 启动命令 | 调试 |
|------|---------|------|
| Electron | `npm start` / `electron .` | DevTools / VSCode |
| Tauri | `cargo tauri dev` / `npm run tauri dev` | rust-analyzer |
| Flutter Desktop | `flutter run -d macos/linux/windows` | DevTools |
| Wails | `wails dev` | Delve |
| Qt | `qmake && make` / `cmake --build` | Qt Creator |
| WPF | `dotnet run` / VS | VS Debugger |
