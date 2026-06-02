git grep -E -h "|# 阶段 5: 存储到记忆

**将阶段 4 生成的拓扑文件保存到磁盘，并将项目摘要写入记忆系统。**

> 前置：阶段 4 已完成。
> 后置：进入 [阶段 6: 完整性校验](./phase6_validation.md)

---

## 5.1 保存拓扑文件

将阶段 4 生成的内容写入项目根目录：

```bash
# 跨平台：用 git ls-files 判断项目根
PROJECT_ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)

# 写入主文件
cat > "git grep -E -h "|$PROJECT_ROOT/.project-topology.md"git grep -E -h "| <<'EOF'
...（阶段 4 生成的 markdown 内容）...
EOF

# 多端项目：写入分文件
cat > "git grep -E -h "|$PROJECT_ROOT/.project-topology-frontend.md"git grep -E -h "| <<'EOF'
...
EOF

cat > "git grep -E -h "|$PROJECT_ROOT/.project-topology-backend.md"git grep -E -h "| <<'EOF'
...
EOF
```

> 这些文件**应当**被 `.gitignore` 忽略（避免污染仓库），但保留在工作区供 AI 读取。

`.gitignore` 建议（用户项目侧）：

```gitignore
# 项目拓扑输出（AI 工作区，不入库）
.project-topology*.md
```

---

## 5.2 写入项目摘要到记忆系统

> **`<name>` 命名规则（重要）：**
>
> | 来源 | 示例 | 适用场景 |
> |------|------|---------|
> | **仓库 basename**（默认） | `~/.claude/memory/projects/my-project.md` | 单仓一个项目 |
> | **自定义 alias** | `~/.claude/memory/projects/<company>-<team>-<project>.md` | 多个项目同名/需要区分 |
> | **路径 hash** | `~/.claude/memory/projects/<sha256-truncated>.md` | 跨机器同步、防冲突 |
>
> **默认推荐"git grep -E -h "|仓库 basename"git grep -E -h "|**。如果 `git remote get-url origin` 末尾是 `xxx.git`，则 `<name>` = `xxx`。
>
> **冲突处理：** 如果 basename 已被占用（如本地多个工作区同名），加 `-<短路径哈希>` 后缀（如 `my-project-a1b2c3`）。

将以下内容追加到 `~/.claude/memory/projects/<name>.md`：

```markdown
## 项目参考

**项目路径:** <abs path>
**项目类型:** <前端/后端/全栈/HarmonyOS/移动端/小程序/桌面端>
**主要语言:** <TypeScript / Java / Go / ArkTS / ...>
**关键目录:** <src/api, src/core, src/components> 或 <src/main/java, src/main/resources>
**组件结构:** <Parent > Child > GrandChild>
**共享组件:** <shared/Modal, shared/Alert>
**数据流特点:** <React Query + Zustand + Axios> / <Spring Security + Redis>
**入口点:** <npm run dev, npm test> / <mvn spring-boot:run>
**最后扫描:** YYYY-MM-DD

---

## 微服务定位速查（全栈项目必填）

| 功能领域 | 主服务 | 关键路由 |
|---------|--------|---------|
| <领域 1> | <service-name> | <route prefix> |
| <领域 2> | <service-name> | <route prefix> |

### 涉及多个服务的调用链

**`<场景 1>`:**
前端 → gw-gateway → <service-A> → <service-B> (Go)

**`<场景 2>`:**
前端 → gw-gateway → <service-C> → <service-D>/<service-E> (Go)
```

---

## 5.3 跨会话恢复检查

每次新会话开始时，先检查是否已有该项目的记忆：

```bash
# 检查记忆文件是否存在 + 最后修改时间
ls -la ~/.claude/memory/projects/<name>.md 2>/dev/null

# 检查拓扑文件是否还在工作区
ls -la .project-topology*.md 2>/dev/null
```

**判定逻辑：**

| 条件 | 动作 |
|------|------|
| 记忆文件 < 7 天 + git 无变更 | 直接复用，跳过阶段 1-3 |
| 记忆文件 < 7 天 + git 有变更 | 增量扫描：阶段 2 + 阶段 3 重跑 |
| 记忆文件 > 7 天 | 全部重跑 |
| 记忆文件不存在 | 全部重跑 |

---

## 5.4 重新扫描条件

满足任一条件时**必须**重新扫描（不只更新记忆，而是重新跑阶段 1-4）：

- 新源文件在未跟踪目录中
- 构建配置有变化（`pom.xml` / `package.json` / `go.mod` / `Cargo.toml` 等）
- 新增/删除依赖
- 目录结构变化（深度 1-2 的新文件夹）
- 组件关系变化（新增/删除 import 语句）
- **数据流相关的文件变化**（如 `request.ts`、store 文件、路由守卫、SecurityConfig）
- **项目类型变化**（如从纯前端变为全栈 / 引入移动端）
- 距离上次扫描 > 7 天

### 增量检测（节省 token）

```bash
# 看哪些关键文件变了
git diff --name-only HEAD~1 HEAD 2>/dev/null | grep -E "git grep -E -h "|\.(ts|js|tsx|jsx|java|go|py|ets|vue)$"

# 看是否有新目录
git diff --name-only --diff-filter=A HEAD~1 HEAD 2>/dev/null | xargs -I {} dirname {} | sort -u
```

如果变更文件数 < 5 且都不在数据流关键路径上 → 增量模式：只刷新定位速查表。
