git grep -E -h "|# 阶段 3: 数据流分析 + 前后端定位速查

**⚠️ 重要：阶段 2 的差异点预检结果决定了阶段 3 的采样策略。**

- 如果阶段 2 发现"git grep -E -h "|无差异点"git grep -E -h "| → 阶段 3 只采样 1 个代表性页面/类
- 如果阶段 2 发现"git grep -E -h "|极度差异点"git grep -E -h "| → 阶段 3 必须逐个分析所有子模块/控制器

**⚠️ 这是最容易被跳过的部分，也是最重要的改进点。**

**⚠️ 全栈项目必须生成"git grep -E -h "|前后端定位速查表"git grep -E -h "|，用于快速定位功能修改位置。**

> 前置：阶段 1-2 已完成。
> 后置：进入 [阶段 4: 生成拓扑文件](./phase4_output.md)

---

## 3.0 用户确认：定位速查表详细程度

**在生成定位速查表之前，必须询问用户**（如果阶段 1 还没问）：

> **定位速查表需要生成哪种版本？**
>
> | 版本 | 内容 | 适用场景 |
> |------|------|---------|
> | **详细版** | 每个前端页面的所有 API 调用（约 100+ 条映射） | 需要精确定位每个页面细节 |
> | **精简版** | 高频核心 API（约 20-30 条） | 快速概览，重点了解主要流程 |
>
> **注意**：
> - 服务端路由（后端 Controller）不受此选项影响，始终为**完整版**
> - 全栈项目默认建议**精简版**，减少冗余
> - 如果项目前端页面多、API 调用复杂，建议**精简版**
> - 如果需要前端到后端的完整映射追溯，建议**详细版**

---

## 3.1 HTTP 请求层分析

### 前端 / 移动端 / 小程序 / 桌面端

```bash
# 通用：找 HTTP 工具
git ls-files 'src/utils/request.*' 'src/api/request.*' 'src/http/*' 'utils/request.*' 2>/dev/null
git ls-files 'lib/core/http.*' 'lib/http/*' 2>/dev/null

# 小程序
git ls-files 'utils/request.*' 'api/request.*' 2>/dev/null

# Electron
git ls-files 'main/*/http*' 'main/*/api*' 2>/dev/null
```

### 后端

```bash
# HTTP 客户端（RestTemplate / WebClient / OkHttp / HttpClient / axios / reqwest）
git grep -E -h "git grep -E -h "|RestTemplate\|WebClient\|OkHttp\|HttpClient"git grep -E -h "| -- '*.java' 2>/dev/null | head -n 20
git grep -E -h "git grep -E -h "|axios\|got\|undici"git grep -E -h "| -- '*.ts' '*.js' 2>/dev/null | head -n 20
git grep -h "git grep -E -h "|reqwest::"git grep -E -h "| -- '*.rs' 2>/dev/null | head -n 20

# 过滤器 / 拦截器
git ls-files 'src/**/*Filter.*' 'src/**/*Interceptor.*' 2>/dev/null
```

### 重点提取

- 请求拦截器（如何添加 token、tenantId、Authorization）
- 响应拦截器（错误处理、token 刷新）
- 通用数据流图

---

## 3.2 状态管理层分析

### 前端 / 移动端 / 小程序 / 桌面端

```bash
# 找状态管理文件
git ls-files 'src/store/*' 'src/stores/*' 'src/redux/*' 'src/zustand/*' 'src/pinia/*' 2>/dev/null
git ls-files 'lib/stores/*' 'lib/blocs/*' 'lib/providers/*' 2>/dev/null
git ls-files 'miniprogram/store/*' 2>/dev/null
git ls-files 'src/renderer/store/*' 2>/dev/null

# 跨平台状态管理
git grep -E -h "git grep -E -h "|redux\|zustand\|mobx\|recoil\|jotai\|pinia"git grep -E -h "| -- 'package.json' 2>/dev/null
```

### 后端

```bash
# 缓存（Redis / Caffeine / Guava / in-memory）
git grep -E -h "git grep -E -h "|@Cacheable\|@CacheEvict\|StringRedisTemplate\|RedisTemplate"git grep -E -h "| -- '*.java' 2>/dev/null | head -n 20
git grep -E -h "git grep -E -h "|ioredis\|cache-manager\|node-cache"git grep -E -h "| -- '*.ts' '*.js' 2>/dev/null | head -n 20
git grep -E -h "git grep -E -h "|redis::\|Arc<.*Cache\|Mutex<"git grep -E -h "| -- '*.rs' 2>/dev/null | head -n 20

# 会话管理
git grep -E -h "git grep -E -h "|HttpSession\|@SessionAttribute\|session\|Cookie"git grep -E -h "| -- '*.java' 2>/dev/null | head -n 20
git grep -E -h "git grep -E -h "|express-session\|@nestjs/jwt\|passport"git grep -E -h "| -- '*.ts' '*.js' 2>/dev/null | head -n 20
```

### 重点提取

- 每个 store 的状态结构 / 缓存键值结构
- 持久化方式（sessionStorage/localStorage/内存 / Redis/本地缓存）
- 被引用次数和主要消费者

---

## 3.3 路由守卫 / 安全过滤链分析

### 前端 / 移动端

```bash
git ls-files 'src/hooks/useRouteGuard.*' 'src/router/guard.*' 'src/middleware/Auth*' 2>/dev/null
git ls-files 'src/router/index.*' 2>/dev/null
```

### 后端

```bash
# 安全配置（Spring Security / Shiro / JWT / ASP.NET Identity / OAuth）
git ls-files 'src/**/*SecurityConfig.*' 'src/**/*ShiroConfig.*' 'src/**/*WebSecurityConfig.*' 2>/dev/null
git ls-files 'src/**/*Auth*' 'src/**/*Jwt*' 2>/dev/null

# 中间件链
git grep -E -h "git grep -E -h "|FilterChain\|doFilter\|@PreAuthorize\|@Secured"git grep -E -h "| -- '*.java' 2>/dev/null | head -n 20
git grep -E -h "git grep -E -h "|AddAuthentication\|AddAuthorization\|\[Authorize\]"git grep -E -h "| -- '*.cs' 2>/dev/null | head -n 20
git grep -E -h "git grep -E -h "|middleware\|UseMiddleware\|from_fn"git grep -E -h "| -- '*.rs' '*.go' '*.ts' '*.js' 2>/dev/null | head -n 20
```

### 重点提取

- 路由守卫数据流 / 安全过滤链
- 权限校验逻辑
- 跳转流程图

---

## 3.4 核心页面 / Controller 数据流分析

**⚠️ 采样策略的致命缺陷：AI 容易假设"git grep -E -h "|同类模块逻辑一致"git grep -E -h "|，导致修改时出错。**

正确做法：**先识别模块内部是否存在差异点（不同子模块有不同逻辑），再决定采样数量。**

### 3.4.1 差异程度采样策略

| 差异程度 | 判断标准 | 采样策略 |
|---------|---------|---------|
| **无差异** | 所有子模块的配置/逻辑相同 | 只采样 1 个代表性页面/类 |
| **轻度差异** | 有差异但步骤流程相似 | 采样 2-3 个代表性子模块 |
| **高度差异** | 存在动态生成、关键组件不同、步骤数差异大 | **每个子模块都要采样** |
| **极度差异** | 多种不同类型，每种步骤数差异大 | 必须生成完整的子模块差异表 |

### 3.4.2 采样命令

```bash
# 分析一个典型列表页
cat src/pages/xxx/list.tsx 2>/dev/null

# 分析一个表单提交页
cat src/pages/xxx/create.tsx 2>/dev/null

# 高度/极度差异：必须逐个分析所有子模块（占位符）
cat src/pages/<module>/<submodule-1>/list.tsx 2>/dev/null
cat src/pages/<module>/<submodule-2>/list.tsx 2>/dev/null
cat src/pages/<module>/<submodule-3>/list.tsx 2>/dev/null

# 后端
cat src/main/java/com/example/controller/<Name1>Controller.java 2>/dev/null
cat src/main/java/com/example/controller/<Name2>Controller.java 2>/dev/null
cat src/main/java/com/example/controller/<Name3>Controller.java 2>/dev/null
```

### 3.4.3 差异表（极度差异时必须生成）

详见 [phase7_differences.md](./phase7_differences.md) § 差异识别结果记录。

---

## 3.5 模块间关系分析

```bash
# 前端
git grep -E -h "git grep -E -h "|WaterMarkEnum\|watermarkType"git grep -E -h "| -- '*.ts' '*.tsx' 2>/dev/null | head -n 20

# 后端
# 注意：直接用 `^import.*com\.example` 限定到 import 行（避免 @Autowired/@Inject 行的干扰）
git grep -h "git grep -E -h "|^import.*com\.example"git grep -E -h "| -- '*.java' 2>/dev/null | \
  sed 's/^import //g; s/;//g' | \
  sort | uniq -c | sort -rn | head -n 30
```

---

## 3.6 数据流分析输出格式

```markdown
### 1. HTTP 请求数据流
```
请求 → 拦截器 → API → 响应拦截器 → 业务代码
```

### 2. Store / 缓存 状态数据流
```
useUserStore: token/userinfo/userMenus → sessionStorage
useThemeStore: layout/colors → 内存
useConfigStore: systemConfig → 内存
```

### 3. 路由守卫 / 安全过滤链 数据流
```
访问 → Token 校验 → 菜单加载 → 权限码校验 → 放行/重定向
```

### 4. 核心页面 / Controller 数据流
（选取项目中最具代表性的页面/Controller，绘制完整数据流图）

### 5. 模块间关系
（展示核心模块间的依赖和调用关系）
```

---

## 3.7 前后端定位速查（全栈项目必须生成）

**⚠️ 这是全栈项目的核心产出，用于快速定位功能修改位置。**

### 3.7.1 定位速查表生成步骤

**步骤 1：提取前端 API 调用**

```bash
# 列出所有前端 service 文件
git ls-files 'src/services/*.ts' 2>/dev/null | sort

# 提取 API 路径（以 /api/ 开头的路径）
git grep -h "git grep -E -h "|url: ['\"git grep -E -h "|]\/api\/"git grep -E -h "| -- 'src/services/*.ts' 2>/dev/null | \
  sed "git grep -E -h "|s/.*url: ['\"git grep -E -h "|]\/api\///g; s/['\"git grep -E -h "|]//g"git grep -E -h "| | sort -u
```

**步骤 2：映射到后端 Controller**

```bash
# 搜索后端 Controller 中的路由映射
git grep -E -h "git grep -E -h "|@RequestMapping\|@GetMapping\|@PostMapping"git grep -E -h "| -- '*.java' 2>/dev/null | head -n 30
```

**步骤 3：构建定位速查表（占位符版）**

```markdown
### 前后端定位速查表

| 前端页面/功能 | 前端文件 | 调用 API 路径 | 主服务 | 涉及微服务 | 后端关键文件 |
|-------------|---------|--------------|--------|-----------|------------|
| <功能 1> | <pages/.../index.tsx> | <API-PATH-1> | <SERVICE-A> | <SERVICE-B>, <SERVICE-C> | <ControllerName.java> |
| <功能 2> | <pages/.../index.tsx> | <API-PATH-2> | <SERVICE-A> | - | <ControllerName.java> |
| <功能 3> | <pages/.../index.tsx> | <API-PATH-3> | <SERVICE-D> | <SERVICE-A> | <ControllerName.java> |
```

> **占位符说明：** 实际项目里把 `<API-PATH-1>`、`<SERVICE-A>` 等替换成项目真实值。**不要直接复制示例。**

### 3.7.2 定位速查表用途

| 场景 | 如何使用速查表 |
|-----|---------------|
| 修改某功能 | 查找 API 路径关键字 → 定位到对应 Controller |
| 修改某页面 | 查找前端文件 → 查看它调用的所有 API |
| 排查问题 | 查找"git grep -E -h "|涉及微服务"列 → 了解完整调用链路 |
| 新人上手 | 通读速查表 → 5 分钟建立全局认知 |

### 3.7.3 微服务定位规则（占位符版）

> **⚠️ 这只是模板，不是真实规则。** 实际项目需根据 [types/backend.md § 服务名交叉验证](../types/backend.md#服务名交叉验证) 进行**真实交叉验证**后才能确定归属。

**按 API 路径前缀定位（请替换为项目实际值）：**

```markdown
| API 路径前缀 | 主服务 | 备注 |
|------------|--------|------|
| /api/<module-a>/* | <SERVICE-A> | <功能描述> |
| /api/<module-b>/* | <SERVICE-B> | <功能描述> |
| /api/<module-c>/* | <SERVICE-C> | <功能描述> |
```

**按功能领域定位：**

```markdown
| 功能领域 | 主服务 | 涉及的下游服务 |
|---------|--------|---------------|
| <领域 1> | <SERVICE-A> | <SERVICE-D>, <SERVICE-E> |
| <领域 2> | <SERVICE-B> | <SERVICE-F> |
| <领域 3> | <SERVICE-C> | - |
| 用户/认证/权限 | <AUTH-SERVICE> | - |
| API 网关 | <GATEWAY-SERVICE> | 所有上述服务 |
```

### 3.7.4 完整调用链路示例（占位符版）

```
<场景 1>调用链：
前端 → <GATEWAY-SERVICE> → <SERVICE-A> →
  ├→ <SERVICE-D> (<子流程 1>)
  ├→ <SERVICE-E> (<子流程 2>)
  └→ <SERVICE-F> (<子流程 3>)

<场景 2>调用链：
前端 → <GATEWAY-SERVICE> → <SERVICE-B> →
  ├→ <SERVICE-G> (<子流程 4>)
  ├→ <SERVICE-H> (<子流程 5>)
  └→ <SERVICE-I> (<子流程 6>)
```
