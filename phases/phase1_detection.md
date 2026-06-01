# 阶段 1: 项目类型检测

**必须首先执行！不同项目类型使用不同的检测命令，不能用前端命令检测后端代码。**

> 前置：无。
> 后置：进入 [阶段 2: 结构扫描](./phase2_structure.md)

---

## 1.1 检测项目类型

```bash
# 检查前端特征（package.json, src/, pages/, components/）
ls package.json 2>/dev/null && head -n 20 package.json

# 检查后端特征（pom.xml, go.mod, requirements.txt, src/main/, app.py）
ls pom.xml 2>/dev/null && echo "=== Maven/Java ===" && head -n 30 pom.xml
ls go.mod 2>/dev/null && echo "=== Go ===" && head -n 15 go.mod
ls requirements.txt 2>/dev/null && echo "=== Python ===" && head -n 20 requirements.txt
ls Cargo.toml 2>/dev/null && echo "=== Rust ===" && head -n 20 Cargo.toml
ls package.json 2>/dev/null && git grep -h "express\|@nestjs\|fastify" -- package.json 2>/dev/null | head -n 3
ls *.csproj 2>/dev/null && echo "=== .NET ==="

# 检查移动端特征
ls app/src/main/AndroidManifest.xml 2>/dev/null && echo "=== Android ==="
ls *.xcodeproj 2>/dev/null && echo "=== iOS ==="
ls pubspec.yaml 2>/dev/null && echo "=== Flutter ==="
ls package.json 2>/dev/null && git grep -h "react-native" -- package.json 2>/dev/null | head -n 3

# 检查小程序特征
ls app.json 2>/dev/null && echo "=== WeChat Mini Program ==="
ls app.acss 2>/dev/null && echo "=== Alipay Mini Program ==="
ls project.swan.json 2>/dev/null && echo "=== Baidu Mini Program ==="

# 检查桌面端特征
ls package.json 2>/dev/null && git grep -h "\"electron\"" -- package.json 2>/dev/null | head -n 3
ls src-tauri/Cargo.toml 2>/dev/null && echo "=== Tauri ==="

# 检查 HarmonyOS 特征（entry/src/main/, *.ets, lib*.so）
ls entry/src/main/ets 2>/dev/null && echo "=== HarmonyOS ==="
git ls-files '*.ets' 2>/dev/null | head -n 10
```

> **Windows 提示：** 如果 `head` 命令不存在，说明你在 PowerShell 而非 Git Bash / WSL。请参考 [SKILL.md § 平台兼容性](../SKILL.md#平台兼容性)。

---

## 1.2 项目类型定义

根据检测结果，确定项目类型：

| 类型 | 特征 | 扫描策略 |
|------|------|---------|
| **前端项目** | 有 package.json，无后端特征 | 使用前端专用检测命令 |
| **后端项目** | 有 pom.xml/go.mod/requirements.txt 等 | 使用后端专用检测命令 |
| **全栈项目** | 同时有前端和后端特征 | 前端+后端检测命令都要执行 |
| **移动端** | Android/iOS/RN/Flutter 特征 | 使用移动端专用检测命令 |
| **小程序** | 微信/支付宝/抖音/百度特征 | 使用小程序专用检测命令 |
| **桌面端** | Electron/Tauri 特征 | 使用桌面端专用检测命令 |
| **HarmonyOS** | 有 entry/src/main/ets, *.ets, lib*.so | 使用 HarmonyOS 专用检测命令 |
| **混合项目** | 多种类型混合 | 根据主要用途选择 + 辅以次要类型 |

---

## 1.3 全栈项目检测（避免遗漏）

**⚠️ 常见错误：只检测到前端就终止，忽略后端代码。**

```bash
# 如果 package.json 存在，继续检查后端 / 移动端 / 其他特征
if [ -f package.json ]; then
  # 检查是否有后端目录
  ls -la server/ backend/ api/ src/main/ app/ 2>/dev/null

  # 检查后端配置文件
  ls pom.xml go.mod requirements.txt Cargo.toml 2>/dev/null

  # 检查是否同时是移动端
  ls app/src/main/AndroidManifest.xml 2>/dev/null
  ls ios/ 2>/dev/null
fi
```

**全栈项目检测原则：必须同时检测所有端，不能因为检测到一种类型就停止。**

---

## 1.4 检测结果记录

将检测到的项目类型记录下来，用于决定后续使用哪套检测命令：

```markdown
**检测到的项目类型:** <前端/后端/全栈/移动端/小程序/桌面端/HarmonyOS/混合>
**检测依据:**
- 前端特征: <package.json 存在，src/ 目录存在>
- 后端特征: <pom.xml 存在，src/main/java 存在>
- 移动端特征: <AndroidManifest.xml 存在>
- 小程序特征: <app.json + miniprogram/ 存在>
- 桌面端特征: <package.json + electron 依赖>
- HarmonyOS 特征: <entry/src/main/ets 存在>
```

---

## 1.5 用户确认事项

**检测到多端项目时，询问用户：**

### 1.5.1 定位速查表详细程度

> 定位速查表需要生成**详细版**还是**精简版**？
> - **详细版**：包含每个前端页面的所有 API 调用（映射表行数多）
> - **精简版**：只包含高频核心 API（约 20-30 条）
> - **说明**：服务端路由不受此选项影响，始终为完整版

### 1.5.2 多端项目文件分离

> 本项目涉及多个端，拓扑文件是否分离生成？
> - **分离**：生成 `.project-topology.md`（主索引）、`.project-topology-frontend.md`、`-backend.md`、`-mobile.md` 等
> - **合并**：生成单一 `.project-topology.md` 文件
> - **说明**：多端项目建议分离，便于不同团队/角色专注各自领域

### 1.5.3 扫描深度

> 本次扫描用哪个深度？（详见 [SKILL.md § 扫描深度选择](../SKILL.md#扫描深度选择)）
> - **快速**：L1 + L2，约 2k token
> - **标准**（默认）：L1-L3 + 核心数据流，约 8k token
> - **深度**：L1-L4 + 完整数据流 + 定位速查，约 20k+ token
