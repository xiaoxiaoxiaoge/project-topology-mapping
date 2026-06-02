git grep -E -h "|# 阶段 7: 同类模块差异识别

**⚠️ 重要警告：这是最容易被遗漏的检查点**

当项目中有**多个相似名称的子模块**时（如 `<module>/<submodule-1>`、`<module>/<submodule-2>`、`<module>/<submodule-3>`），即使它们看起来结构相似，**内部逻辑可能完全不同**。必须显式识别，否则 AI 会假设"git grep -E -h "|所有子模块逻辑相同"git grep -E -h "|导致后续修改出错。

> 前置：阶段 6 校验全绿。
> 后置：本 skill 执行完毕。

---

## 7.1 何时触发

满足以下任一条件时**必须**执行阶段 7：

- 存在 ≥ 3 个同级子模块（如 `pages/trace/{file-flow,word,picture,video,...}`）
- 子模块命名相似但属于不同业务领域
- 子模块虽然结构相似但流程步骤不同
- 阶段 2 的差异点预检判定为"git grep -E -h "|高度变异"git grep -E -h "|或"git grep -E -h "|极度变异"git grep -E -h "|

**跳过条件：**

- 项目只有 1 个模块 / 没有同级子模块
- 所有子模块逻辑完全一致（极少见）
- 用户明确说"git grep -E -h "|不需要差异表"git grep -E -h "|

---

## 7.2 变异程度判定

参考阶段 2 的预检结果：

| 变异程度 | 判断标准 | 阶段 3 采样 | 阶段 7 差异表 |
|---------|---------|------------|--------------|
| **无变异** | 所有子模块的配置/逻辑相同 | 1 个 | 可选 |
| **轻度变异** | 有差异但步骤流程相似 | 2-3 个 | 推荐 |
| **高度变异** | 存在动态生成、关键组件不同、步骤数差异大 | 每个子模块 | **必填** |
| **极度变异** | 多种不同类型，每种步骤数差异大 | 每个子模块 | **必填** |

---

## 7.3 前端项目检查命令

```bash
# 1. 列出所有子模块的 config.ts
find src/pages/<module> -name "git grep -E -h "|config.ts"git grep -E -h "| -exec echo "git grep -E -h "|=== {} ==="git grep -E -h "| \; -exec cat {} \;

# 2. 对比 totalStep 值
grep -rh "git grep -E -h "|totalStep"git grep -E -h "| src/pages/<module>/*/config.ts

# 3. 检查是否有 stepsFilter 或动态步骤
grep -rh "git grep -E -h "|stepsFilter\|steps\["git grep -E -h "| src/pages/<module>/*/config.ts

# 4. 对比关键组件差异
grep -rh "git grep -E -h "|ImgCrop\|ImgRotate\|ImgCorrect\|ImgExtract\|DocExtract"git grep -E -h "| \
  src/pages/<module>/*/trace.tsx | sort | uniq -c | sort -rn

# 5. API 调用差异（不同子模块调用的 services 是否不同）
grep -rh "git grep -E -h "|from ['\"git grep -E -h "|]@/services/"git grep -E -h "| src/pages/<module>/*/list.tsx | \
  sed "git grep -E -h "|s/.*from ['\"git grep -E -h "|]@\/services\///g; s/['\"git grep -E -h "|].*//g"git grep -E -h "| | \
  sort | uniq -c | sort -rn

# 6. 路由/跳转逻辑差异
grep -rh "git grep -E -h "|router\.\|navigate\|useNavigate\|history\."git grep -E -h "| src/pages/<module>/*/trace.tsx | \
  sort | uniq

# 7. 表单处理差异
grep -rh "git grep -E -h "|onFinish\|onSubmit\|handleSubmit"git grep -E -h "| src/pages/<module>/*/trace.tsx | \
  sort | uniq

# 8. 条件渲染差异
grep -rh "git grep -E -h "|visible\|show\|display\|render"git grep -E -h "| src/pages/<module>/*/trace.tsx | \
  grep -E "git grep -E -h "|step|Step"git grep -E -h "| | sort | uniq

# 9. 页面间状态传递方式
grep -rh "git grep -E -h "|useState\|useContext\|createContext\|Provider"git grep -E -h "| src/pages/<module>/*/trace.tsx | \
  sort | uniq -c | sort -rn

# 10. 错误处理逻辑差异
grep -rh "git grep -E -h "|message\.\|notification\|Modal\.error\|onError"git grep -E -h "| src/pages/<module>/*/trace.tsx | \
  sort | uniq

# 11. 加载状态处理差异
grep -rh "git grep -E -h "|loading\|isLoading\|Loading\|Spin"git grep -E -h "| src/pages/<module>/*/trace.tsx | \
  sort | uniq
```

---

## 7.4 后端项目检查命令

```bash
# 1. 列出所有 Controller
find src -type f -name "git grep -E -h "|*Controller.java"git grep -E -h "| 2>/dev/null

# 2. 对比各 Controller 的方法数
grep -rh "git grep -E -h "|public.*\(.*\).*\{"git grep -E -h "| src/*Controller.java 2>/dev/null | wc -l

# 3. 检查是否有不同的业务逻辑
grep -rh "git grep -E -h "|@Transactional\|@Cacheable"git grep -E -h "| src/*Controller.java 2>/dev/null

# 4. 对比 Service 层实现差异
find src -type f -name "git grep -E -h "|*Service.java"git grep -E -h "| 2>/dev/null

# 5. Feign Client 调用差异
grep -rh "git grep -E -h "|@FeignClient"git grep -E -h "| src/ --include="git grep -E -h "|*.java"git grep -E -h "| 2>/dev/null

# 6. 中间件链差异
grep -rh "git grep -E -h "|Use\|Middleware\|func.*Middleware"git grep -E -h "| . --include="git grep -E -h "|*.go"git grep -E -h "| 2>/dev/null | grep -v vendor
```

> 完整的后端变异点检测命令集见 [../types/backend.md](../types/backend.md) 的"git grep -E -h "|变异点检测"git grep -E -h "|章节。

---

## 7.5 移动端项目检查命令

```bash
# Android
find app/src/main/java -name "git grep -E -h "|*Activity.java"git grep -E -h "| -o -name "git grep -E -h "|*Fragment.java"git grep -E -h "|

# iOS
find Sources -name "git grep -E -h "|*ViewController.swift"git grep -E -h "|

# React Native / Flutter
grep -rh "git grep -E -h "|Stack.Screen\|GetPage\|Get.to\|push"git grep -E -h "| lib/ 2>/dev/null | sort -un
```

> 详见 [../types/mobile.md](../types/mobile.md)。

---

## 7.6 小程序检查命令

```bash
# 微信小程序：app.json 注册的页面路径
cat app.json | grep -A 100 '"git grep -E -h "|pages"git grep -E -h "|'

# 各页面的生命周期
grep -rh "git grep -E -h "|onLoad\|onShow\|onReady\|onHide\|onUnload"git grep -E -h "| miniprogram/pages/ 2>/dev/null | sort | uniq

# 跨平台差异（以微信 vs 支付宝为例）
grep -rh "git grep -E -h "|my\.\|wx\."git grep -E -h "| miniprogram/ --include="git grep -E -h "|*.js"git grep -E -h "| --include="git grep -E -h "|*.ts"git grep -E -h "| 2>/dev/null | \
  sed 's/.*\(my\.\|wx\.\)\([a-zA-Z]*\).*/\1\2/g' | sort | uniq -c | sort -rn
```

> 详见 [../types/miniprogram.md](../types/miniprogram.md)。

---

## 7.7 桌面端检查命令

```bash
# Electron: Main 进程 IPC handler
grep -rh "git grep -E -h "|ipcMain\.handle\|ipcMain\.on"git grep -E -h "| main/ 2>/dev/null

# Electron: Renderer 进程 IPC 调用
grep -rh "git grep -E -h "|ipcRenderer\.invoke\|ipcRenderer\.send"git grep -E -h "| renderer/ 2>/dev/null

# Tauri: Rust 端 command
grep -rh "git grep -E -h "|#\[tauri::command\]"git grep -E -h "| src-tauri/src/ 2>/dev/null
```

> 详见 [../types/desktop.md](../types/desktop.md)。

---

## 7.8 差异识别结果记录

当发现同类模块有不同流程时，必须在拓扑中记录。

### 7.8.1 前端模块差异表

```markdown
### [模块名] 流程差异分析

| 子模块 | 步骤数 | config.ts关键配置 | 核心组件 | 差异原因 |
|--------|--------|------------------|---------|---------|
| file-flow | 2 步 | totalStep=2 | DocExtract | 文档直接解析 |
| pc-screen | 3-5 步 | stepsFilter | ImgCrop | 5 步(部分屏拍屏) |
| ... | ... | ... | ... | ... |
```

### 7.8.2 后端 API 差异表

```markdown
### [模块名] API 流程差异分析

| 控制器 | 路径 | 主要方法 | 业务逻辑差异 |
|--------|------|---------|-------------|
| <Controller-1> | /api/<module-1> | <method-1>, <method-2> | <差异说明> |
| <Controller-2> | /api/<module-2> | <method-3>, <method-4> | <差异说明> |
| ... | ... | ... | ... |
```

### 7.8.3 移动端页面差异表

```markdown
### [模块名] 页面差异分析

| 页面 | 路由 | 核心组件 | 状态管理 | 差异原因 |
|------|------|---------|---------|---------|
| Home | / | HomeScreen | useHomeStore | 默认入口 |
| Profile | /profile | ProfileScreen | useUserStore | 需要登录 |
```

---

## 7.9 错误示例 vs 正确做法

**错误示例：**
> "git grep -E -h "|trace 模块都是 2 步取证流程" （错误！只有 file-flow 是 2 步）

**正确做法：**
> 列出 10 种水印类型各自的取证步骤，说明差异原因。

---

## 7.10 完成确认

阶段 7 完成后，整个 skill 执行结束。

输出最终给用户：

```markdown
✅ 拓扑扫描完成
- 项目类型: <类型>
- 扫描深度: <快速/标准/深度>
- 拓扑文件: .project-topology.md（+ 前端/后端分文件）
- 记忆文件: ~/.claude/memory/projects/<name>.md
- 模块差异: <N> 个子模块存在差异，已记录
- 下一步: 可以基于此拓扑开始实现 / 重构 / 排查
```
