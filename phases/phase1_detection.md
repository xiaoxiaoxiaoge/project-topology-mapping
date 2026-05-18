# 阶段 1: 项目类型检测

**必须首先执行！不同项目类型使用不同的检测命令，不能用前端命令检测后端代码。**

---

## 1.1 检测项目类型

```bash
# 检查前端特征（package.json, src/, pages/, components/）
ls package.json 2>/dev/null && cat package.json | head -20

# 检查后端特征（pom.xml, go.mod, requirements.txt, src/main/, app.py）
ls pom.xml 2>/dev/null && echo "=== Maven/Java ===" && head -30 pom.xml
ls go.mod 2>/dev/null && echo "=== Go ===" && head -15 go.mod
ls requirements.txt 2>/dev/null && echo "=== Python ===" && head -20 requirements.txt
ls Cargo.toml 2>/dev/null && echo "=== Rust ===" && head -20 Cargo.toml

# 检查 HarmonyOS 特征（entry/src/main/, *.ets, lib*.so）
ls entry/src/main/ets 2>/dev/null && echo "=== HarmonyOS ==="
find . -maxdepth 3 -name "*.ets" 2>/dev/null | head -10
```

---

## 1.2 项目类型定义

根据检测结果，确定项目类型：

| 类型 | 特征 | 扫描策略 |
|------|------|---------|
| **前端项目** | 有 package.json，无后端特征 | 使用前端专用检测命令 |
| **后端项目** | 有 pom.xml/go.mod/requirements.txt 等 | 使用后端专用检测命令 |
| **全栈项目** | 同时有前端和后端特征 | 前端+后端检测命令都要执行 |
| **HarmonyOS** | 有 entry/src/main/ets, *.ets, lib*.so | 使用 HarmonyOS 专用检测命令 |
| **混合项目** | 前端框架+后端框架（如Next.js+Python） | 根据主要用途选择检测策略 |

---

## 1.3 全栈项目检测（避免遗漏后端代码）

**⚠️ 常见错误：只检测到前端就终止，忽略后端代码。**

```bash
# 如果 package.json 存在，继续检查后端特征
if [ -f package.json ]; then
  # 检查是否有后端目录
  ls -la server/ backend/ api/ src/main/ app/ 2>/dev/null
  # 检查后端配置文件
  ls pom.xml go.mod requirements.txt 2>/dev/null
fi
```

**全栈项目检测原则：必须同时检测前端和后端，不能因为检测到前端就停止。**

---

## 1.4 检测结果记录

将检测到的项目类型记录下来，用于决定后续使用哪套检测命令：

```markdown
**检测到的项目类型:** <前端/后端/全栈/HarmonyOS>
**检测依据:**
- 前端特征: <package.json存在，src/目录存在>
- 后端特征: <pom.xml存在，src/main/java存在>
```