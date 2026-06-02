# 后端项目检测命令

> Single source of truth for backend project scanning.
> 适用：Java/Spring / Go / Python (Django/Flask/FastAPI) / Node/TS (Express/Koa/NestJS/Fastify) / .NET / Rust。

---

## 项目类型识别

```bash
# Java / Spring
ls -la pom.xml build.gradle build.gradle.kts 2>/dev/null
ls -la src/main/java/ 2>/dev/null

# Go
ls -la go.mod go.sum 2>/dev/null
ls -la cmd/ pkg/ internal/ 2>/dev/null

# Python
ls -la requirements.txt pyproject.toml setup.py 2>/dev/null
ls -la manage.py app.py 2>/dev/null

# Node / TypeScript
ls -la package.json tsconfig.json 2>/dev/null
git grep -h "express\|@nestjs\|fastify\|koa\b" -- 'package.json' 2>/dev/null

# .NET
ls -la *.csproj *.sln Program.cs Startup.cs 2>/dev/null

# Rust
ls -la Cargo.toml 2>/dev/null
ls -la src/main.rs 2>/dev/null
```

| 特征文件 | 项目类型 |
|---------|---------|
| `pom.xml` / `build.gradle` | Java/Spring |
| `go.mod` | Go |
| `requirements.txt` + `manage.py` | Python/Django |
| `requirements.txt` + `app.py` | Python/Flask |
| `pyproject.toml` + `main.py` | Python/FastAPI |
| `package.json` + `express`/`@nestjs/core`/`fastify`/`koa` | Node/TS |
| `*.csproj` + `Program.cs` | .NET |
| `Cargo.toml` | Rust |

---

## Java / Spring

### 目录结构

```bash
# 1. Java 源码
git ls-files 'src/main/java/**/*.java' 2>/dev/null | head -n 100

# 2. 包结构
find src/main/java -maxdepth 2 -type d 2>/dev/null | sort

# 3. 资源
ls -la src/main/resources/ 2>/dev/null
cat pom.xml 2>/dev/null | head -n 50

# 4. Controller / Service / Repository
git ls-files 'src/**/*Controller.java' 2>/dev/null | head -n 30
git ls-files 'src/**/*Service.java' 2>/dev/null | head -n 30
git ls-files 'src/**/*Repository.java' 2>/dev/null | head -n 30
```

### 差异点检测

```bash
# 1. 所有 Controller + 路由映射
git ls-files 'src/**/*Controller.java' 2>/dev/null | sort
git grep -h "@RequestMapping\|@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping\|@PatchMapping" \
  -- '*.java' 2>/dev/null | sort -u

# 2. Service 方法签名
git ls-files 'src/**/*Service.java' 2>/dev/null | head -n 30
git grep -h "public.*(" -- 'src/**/*Service.java' 2>/dev/null | head -n 50

# 3. Entity / Model / DO
git ls-files 'src/**/*Entity.java' 'src/**/*Model.java' 'src/**/*DO.java' 2>/dev/null | head -n 20

# 4. 多数据源
git grep -h "@DataSource\|@Primary\|MultiDataSource" -- '*.java' 2>/dev/null

# 5. 事务
git grep -h "@Transactional\|@Transactional(readOnly" -- '*.java' 2>/dev/null | sort -u

# 6. 缓存
git grep -h "@Cacheable\|@CacheEvict\|@RedisCache" -- '*.java' 2>/dev/null | sort -u

# 7. 异常处理
git grep -h "@ExceptionHandler\|@ControllerAdvice\|@RestControllerAdvice" -- '*.java' 2>/dev/null | sort -u

# 8. 验证注解
git grep -h "@Valid\|@Validated" -- '*.java' 2>/dev/null | sort -u

# 9. Feign Client
git ls-files 'src/**/*Feign.java' 2>/dev/null
git grep -h "@FeignClient\|Feign\|fallback" -- '*.java' 2>/dev/null | head -n 30

# 10. 微服务调用（按服务名前缀）
git grep -h "GwBIZFeign\|GwDocsyFeign\|GwPicsyFeign\|GwVideosyFeign" -- '*.java' 2>/dev/null | head -n 20
```

### 数据流

```bash
# HTTP 客户端（RestTemplate / WebClient / OkHttp）
git grep -h "RestTemplate\|WebClient\|OkHttp\|HttpClient" -- '*.java' 2>/dev/null | head -n 20

# 过滤器 / 拦截器
git ls-files 'src/**/*Filter.java' 2>/dev/null
git ls-files 'src/**/*Interceptor.java' 2>/dev/null

# 缓存（Redis / Caffeine）
git grep -h "@Cacheable\|@CacheEvict\|StringRedisTemplate\|RedisTemplate" -- '*.java' 2>/dev/null | head -n 20

# 会话
git grep -h "HttpSession\|@SessionAttribute\|session\|Cookie" -- '*.java' 2>/dev/null | head -n 20

# 安全配置
git ls-files 'src/**/*SecurityConfig.java' 'src/**/*ShiroConfig.java' 'src/**/*WebSecurityConfig.java' 2>/dev/null
git grep -h "FilterChain\|doFilter\|@PreAuthorize\|@Secured" -- '*.java' 2>/dev/null | head -n 20
```

---

## Go

### 目录结构

```bash
# 1. Go 源码
git ls-files '*.go' 2>/dev/null | grep -v vendor | head -n 100

# 2. 模块结构
ls -la cmd/ pkg/ internal/ 2>/dev/null

# 3. go.mod
cat go.mod 2>/dev/null | head -n 30
```

### 差异点检测

```bash
# 1. Handler / Controller
git ls-files '*handler*.go' '*controller*.go' 2>/dev/null | grep -v vendor

# 2. 路由定义（Gin / Echo / Chi / net/http）
git grep -h "HandleFunc\|Handle(\|r\.Get\|r\.Post\|r\.Put\|r\.Delete\|router\." \
  -- '*.go' 2>/dev/null | grep -v vendor | head -n 30

# 3. 中间件
git grep -h "Use\|Middleware\|func.*Middleware" -- '*.go' 2>/dev/null | grep -v vendor | head -n 20

# 4. 数据库模型
git ls-files '*.go' 2>/dev/null | xargs grep -l "type.*struct" 2>/dev/null | grep -v vendor | head -n 20

# 5. ORM（GORM / XORM / Ent）
git grep -h "gorm\.\|xorm\.\|ent\." -- '*.go' 2>/dev/null | head -n 20

# 6. 配置（Viper / envconfig）
git grep -h "viper\.\|envconfig\|os\.Getenv" -- '*.go' 2>/dev/null | sort -u | head -n 20
```

---

## Python

### Django

```bash
# 1. 源码
git ls-files '*.py' 2>/dev/null | grep -v venv | grep -v __pycache__ | head -n 100

# 2. 应用结构
ls -la apps/ core/ models/ views/ 2>/dev/null

# 3. settings.py
find . -maxdepth 3 -name "settings.py" 2>/dev/null

# 4. 依赖
cat requirements.txt 2>/dev/null | head -n 30
```

#### 差异点检测（Django）

```bash
# 1. View 列表
git ls-files 'views.py' 2>/dev/null | grep -v venv | grep -v __pycache__

# 2. URL 路由
git ls-files 'urls.py' 2>/dev/null
git grep -h "^from django.urls\|^from django.conf.urls" -- 'urls.py' 2>/dev/null
git grep -h "path\|url" -- 'urls.py' 2>/dev/null | head -n 30

# 3. Model
git ls-files 'models.py' 'models/*.py' 2>/dev/null
git grep -h "class.*Model\|class.*Admin" -- 'models.py' 'models/*.py' 2>/dev/null | head -n 20

# 4. 中间件
git grep -h "MIDDLEWARE" -- 'settings.py' 2>/dev/null

# 5. 缓存
git grep -h "CACHES" -- 'settings.py' 2>/dev/null
```

### Flask

```bash
# 1. 入口
ls app.py wsgi.py 2>/dev/null

# 2. Blueprint
git ls-files '*blueprint*.py' '*bp.py' 2>/dev/null
git grep -h "Blueprint(" -- '*.py' 2>/dev/null | head -n 20

# 3. 装饰器路由
git grep -h "@app.route\|@bp.route" -- '*.py' 2>/dev/null | sort -u | head -n 30
```

### FastAPI

```bash
# 1. 入口
ls main.py app.py 2>/dev/null

# 2. 路由
git grep -h "@app\.\(get\|post\|put\|delete\|patch\)\|@router\." -- '*.py' 2>/dev/null | sort -u | head -n 30

# 3. 依赖注入
git grep -h "Depends(" -- '*.py' 2>/dev/null | sort -u | head -n 20

# 4. Pydantic 模型
git ls-files 'schemas/*.py' '*schema*.py' 2>/dev/null
git grep -h "BaseModel" -- '*.py' 2>/dev/null | head -n 20
```

---

## Node.js / TypeScript

### 目录结构

```bash
# 1. 源码
git ls-files 'src/**/*.ts' 'src/**/*.js' 2>/dev/null | head -n 100

# 2. 路由
git ls-files 'src/routes/*' 'src/router/*' 'src/controllers/*' 2>/dev/null

# 3. 中间件
git ls-files 'src/middlewares/*' 'src/middleware/*' 2>/dev/null

# 4. 服务 / Repository
git ls-files 'src/services/*' 'src/repositories/*' 2>/dev/null

# 5. 配置
git ls-files 'src/config/*' 2>/dev/null
cat package.json 2>/dev/null | head -n 50
```

### 框架识别

```bash
git grep -h "express\|@nestjs/core\|fastify\|koa\b\|hapi\b" -- 'package.json' 2>/dev/null | head -n 3
```

| 依赖 | 框架 |
|------|------|
| `express` | Express |
| `@nestjs/core` | NestJS |
| `fastify` | Fastify |
| `koa` | Koa |
| `@hapi/hapi` | Hapi |

### 差异点检测

```bash
# 1. 路由定义
git grep -h "router\.\(get\|post\|put\|delete\|patch\)\|app\.\(get\|post\|put\|delete\|patch\)\|@Get\|@Post\|@Put\|@Delete" \
  -- '*.ts' '*.js' 2>/dev/null | sort -u | head -n 50

# 2. 控制器
git ls-files 'src/**/*controller*.ts' 'src/**/*controller*.js' 2>/dev/null

# 3. 中间件
git ls-files 'src/**/*middleware*.ts' 'src/**/*middleware*.js' 2>/dev/null
git grep -h "app\.use\|router\.use\|@Injectable" -- '*.ts' '*.js' 2>/dev/null | sort -u | head -n 20

# 4. ORM / ODM（TypeORM / Sequelize / Prisma / Mongoose）
git grep -h "@Entity\|@Column\|@PrimaryGeneratedColumn\|Schema\|Model<" \
  -- '*.ts' 2>/dev/null | sort -u | head -n 20

# 5. 装饰器（NestJS）
git grep -h "@Controller\|@Injectable\|@Module\|@Get\|@Post\|@Body\|@Param\|@Query" \
  -- '*.ts' 2>/dev/null | sort -u | head -n 30

# 6. WebSocket / GraphQL
git grep -h "@WebSocketGateway\|@Resolver\|@Query\|@Mutation\|@Subscription" \
  -- '*.ts' 2>/dev/null | sort -u | head -n 20

# 7. 异常过滤器
git grep -h "@Catch\|@UseFilters\|HttpException" -- '*.ts' 2>/dev/null | sort -u | head -n 20
```

### 数据流

```bash
# 1. HTTP 客户端
git grep -h "axios\|got\|node-fetch\|undici" -- '*.ts' '*.js' 2>/dev/null | sort -u | head -n 20

# 2. 拦截器（响应/请求）
git ls-files 'src/**/*interceptor*.ts' 2>/dev/null
git grep -h "intercept(\|@UseInterceptors" -- '*.ts' 2>/dev/null | sort -u | head -n 20

# 3. 缓存（ioredis / cache-manager）
git grep -h "ioredis\|cache-manager\|@nestjs/cache" -- '*.ts' 2>/dev/null | sort -u | head -n 20

# 4. 会话 / 鉴权
git grep -h "express-session\|@nestjs/jwt\|passport\|@UseGuards" -- '*.ts' 2>/dev/null | sort -u | head -n 20
```

---

## .NET (ASP.NET Core)

### 目录结构

```bash
# 1. C# 源码
git ls-files '*.cs' 2>/dev/null | head -n 100

# 2. 项目配置
cat *.csproj 2>/dev/null | head -n 30

# 3. Controllers / Services
git ls-files 'Controllers/*' 'Services/*' 2>/dev/null

# 4. 中间件
git ls-files 'Middleware/*' 2>/dev/null
```

### 差异点检测

```bash
# 1. Controller + 路由
git ls-files 'Controllers/*Controller.cs' 2>/dev/null
git grep -h "\[HttpGet\]\|\[HttpPost\]\|\[HttpPut\]\|\[HttpDelete\]\|\[Route" \
  -- '*.cs' 2>/dev/null | sort -u | head -n 30

# 2. 特性路由
git grep -h "\[ApiController\]\|\[Route(" -- '*.cs' 2>/dev/null | sort -u

# 3. 服务注入
git grep -h "AddScoped\|AddSingleton\|AddTransient" -- '*.cs' 2>/dev/null | sort -u

# 4. EF Core / Dapper
git grep -h "DbContext\|DbSet\|EntityFramework\|IDbConnection" -- '*.cs' 2>/dev/null | sort -u | head -n 20

# 5. 中间件
git grep -h "UseMiddleware\|app\.Use" -- '*.cs' 2>/dev/null | sort -u | head -n 20

# 6. 鉴权
git grep -h "AddAuthentication\|AddAuthorization\|\[Authorize\]" -- '*.cs' 2>/dev/null | sort -u

# 7. SignalR
git grep -h "Hub\b\|IHubContext" -- '*.cs' 2>/dev/null | sort -u | head -n 20

# 8. gRPC
git ls-files '*.proto' 2>/dev/null
git grep -h "GrpcService\|AddGrpc" -- '*.cs' 2>/dev/null
```

### 数据流

```bash
# 1. HTTP 客户端（HttpClient / IHttpClientFactory）
git grep -h "HttpClient\|IHttpClientFactory" -- '*.cs' 2>/dev/null | sort -u | head -n 20

# 2. 过滤器 / 拦截器
git ls-files 'Filters/*' 2>/dev/null
git grep -h "\[TypeFilter\|\[ServiceFilter\]\|IAsyncActionFilter" -- '*.cs' 2>/dev/null | sort -u

# 3. 缓存
git grep -h "IMemoryCache\|IDistributedCache\|AddStackExchangeRedisCache" -- '*.cs' 2>/dev/null | sort -u
```

---

## Rust

### 目录结构

```bash
# 1. 源码
git ls-files 'src/*.rs' 2>/dev/null | head -n 50

# 2. Cargo.toml
cat Cargo.toml 2>/dev/null

# 3. Handler / Route 模块
git ls-files 'src/handlers/*' 'src/routes/*' 2>/dev/null
```

### 框架识别

```bash
git grep -h "actix-web\|axum\|rocket\|warp" -- 'Cargo.toml' 2>/dev/null | head -n 3
```

| 依赖 | 框架 |
|------|------|
| `actix-web` | Actix |
| `axum` | Axum |
| `rocket` | Rocket |
| `warp` | Warp |

### 差异点检测

```bash
# 1. Handler 函数
git grep -h "#\[get\|#\[post\|#\[put\|#\[delete\|#\[handler\|#\[route\|#\[actix_web::" \
  -- '*.rs' 2>/dev/null | sort -u | head -n 30

# 2. 路由注册
git grep -h "\.route(\|\.service(\|web::resource\|web::scope" -- '*.rs' 2>/dev/null | sort -u | head -n 20

# 3. 中间件
git grep -h "middleware\|wrap_fn\|from_fn" -- '*.rs' 2>/dev/null | sort -u | head -n 20

# 4. Trait 实现
git grep -h "impl.*for\|trait " -- '*.rs' 2>/dev/null | sort -u | head -n 20

# 5. 数据库（sqlx / diesel / sea-orm）
git grep -h "sqlx::\|diesel::\|sea_orm::" -- '*.rs' 2>/dev/null | sort -u | head -n 20

# 6. 错误处理
git grep -h "thiserror\|anyhow\|#\[derive(Debug" -- '*.rs' 2>/dev/null | sort -u | head -n 20

# 7. 异步运行时
git grep -h "tokio::main\|#\[tokio::" -- '*.rs' 2>/dev/null | sort -u
```

### 数据流

```bash
# 1. 状态共享
git grep -h "Arc<\|Mutex<\|RwLock<\|web::Data" -- '*.rs' 2>/dev/null | sort -u | head -n 20

# 2. HTTP 客户端（reqwest）
git grep -h "reqwest::" -- '*.rs' 2>/dev/null | sort -u | head -n 20

# 3. 配置
git grep -h "config::\|env::var\|Figment" -- '*.rs' 2>/dev/null | sort -u | head -n 20
```

---

## 通用后端采样命令

> ⚠️ **以下是模板路径**（`com/example`、`UserController`、`<Name1>` 等都是占位符）。**实际使用时必须替换为你的项目真实路径。** 直接复制执行会找不到文件。

```bash
# 分析一个典型 Controller（请替换 com/example 为实际包路径）
cat src/main/java/com/example/controller/<Name1>Controller.java 2>/dev/null
cat src/controllers/<Name1>Controller.ts 2>/dev/null

# 分析一个典型 Service（请替换 com/example 为实际包路径）
cat src/main/java/com/example/service/<Name1>Service.java 2>/dev/null
cat src/services/<Name1>Service.ts 2>/dev/null

# 高度变异：必须逐个采样（占位符）
cat src/main/java/com/example/controller/<Name1>Controller.java
cat src/main/java/com/example/controller/<Name2>Controller.java
```

---

## 服务名交叉验证

**⚠️ 关键警告：命名相似 ≠ 同一服务，必须交叉验证**

当项目涉及一种语言的微服务调用另一种语言的服务时（如 Java 调用 Go / Go 调用 Python），容易犯的错误是：根据命名相似性推断服务关系，导致拓扑错误。

**反例（占位符版）：**
```
服务 A（Go）注册到 Eureka → <SERVICE-A-REG-NAME>
服务 B（Java）侧常量 → <SERVICE-B-CLIENT-NAME> = "<SERVICE-B-ACTUAL-NAME>"

错误推断：A 被 B 代理
实际上：A 和 B 是两个完全不同的服务！
```

> **占位符说明：** 实际项目里把 `<SERVICE-A-REG-NAME>` 等替换成项目真实的服务名 / 常量名。本节是通用方法论，**不绑定任何特定业务**。

### 验证步骤

**步骤 1：找到所有服务名常量**

```bash
# Java（公共常量类）
git grep -h "SERVICE_NAME.*=\|ServiceName.*=" -- '*.java' 2>/dev/null

# Go（注册配置）
git grep -h "server_name\|ServiceName" -- '*.go' 2>/dev/null | grep -v vendor

# Node/TS
git grep -h "SERVICE_NAME\|server_name" -- '*.ts' '*.js' 2>/dev/null

# Python
git grep -h "SERVICE_NAME\|service_name" -- '*.py' 2>/dev/null | grep -v venv

# .NET
git grep -h "ServiceName\|SERVICE_NAME" -- '*.cs' 2>/dev/null
```

**步骤 2：找到客户端的 name 值（Feign / gRPC / HTTP Client）**

```bash
# Java Feign
git grep -h "@FeignClient.*name\|@FeignClient(name" -- '*.java' 2>/dev/null

# Go gRPC service
git ls-files '*.proto' 2>/dev/null
git grep -h "^service " -- '*.proto' 2>/dev/null

# .NET HttpClient / GrpcService
git grep -h "AddHttpClient\|\[Service\]" -- '*.cs' 2>/dev/null
```

**步骤 3：找到目标服务的注册名**

```bash
# 通用：所有配置文件
git grep -h "server_name\|application.name\|spring.application.name" \
  -- '*.yaml' '*.yml' '*.toml' '*.properties' 2>/dev/null

# Go 服务注册（Eureka / Consul / Nacos）
git grep -h "eureka\|Register\|ServerName" -- '*.go' 2>/dev/null | grep -v vendor

# Node/TS 服务注册
git grep -h "service.name\|SERVICE_NAME" -- '*.ts' '*.js' 2>/dev/null
```

**步骤 4：交叉验证表**

把步骤 1-3 的结果填入下表，**逐行判断**：

| 客户端服务名（Feign/gRPC/HTTP） | 客户端常量定义 | 服务端实际注册名 | 是否同一服务 |
|------------------------------|--------------|----------------|------------|
| `<CLIENT-NAME-1>` | `<CONSTANT-NAME-1>` | `<SERVER-NAME-1>` | 需判断 |
| `<CLIENT-NAME-2>` | `<CONSTANT-NAME-2>` | `<SERVER-NAME-2>` | 需判断 |
| ... | ... | ... | ... |

**判断规则：**
- 客户端 name 与服务端注册名**完全一致** → 同一服务
- 客户端 name 与服务端注册名**不一致**（即使前缀相似）→ 不同服务，在拓扑中**分别描述**
- **命名相似不是合并依据**，必须验证实际注册名

### 检测结果记录格式（占位符版）

```markdown
### 服务名映射表

| 客户端服务名 | 客户端常量 | 服务端注册名 | 是否同一服务 |
|------------|----------|------------|------------|
| <CLIENT-NAME-1> | <CONSTANT-NAME-1> | <SERVER-NAME-1> | 是 / 否 |
| <CLIENT-NAME-2> | <CONSTANT-NAME-2> | <SERVER-NAME-2> | 是 / 否 |
```

### 后端项目服务名检测（通用版）

```bash
# 1. 查找服务名常量定义文件（任何语言的项目通用模式：*ServiceName*Constant*）
git ls-files 2>/dev/null | grep -iE "ServiceName.*Constant|ServiceConstants" | head -n 5

# 2. 读取所有候选常量文件，列出服务名
for f in $(git ls-files 2>/dev/null | grep -iE "ServiceName.*Constant|ServiceConstants"); do
  echo "=== $f ==="
  cat "$f" | grep -E "SERVICE_NAME|ServiceName" | head -n 20
done

# 3. 列出所有 Feign Client（Java 专用）
git ls-files 'src/**/*Feign.java' 'src/**/*FeignClient.java' 2>/dev/null

# 4. 查找目标服务的注册配置
git ls-files 2>/dev/null | grep -E "(application|bootstrap|config)\.(yaml|yml|properties|json|toml)" | \
  xargs grep -l "server_name\|application.name\|spring.application" 2>/dev/null
```

> **重要：** 本节的命令和示例**全部用占位符**，不绑定任何特定业务。请在使用时替换为你的项目真实的服务名/常量名。
