# 后端项目检测命令

## Java/Spring 项目

### 目录结构检测
```bash
# 1. 源码目录结构
find src/main/java -maxdepth 5 -type f -name "*.java" 2>/dev/null | head -100

# 2. 包结构
ls -la src/main/java/ 2>/dev/null
find src/main/java -maxdepth 2 -type d | sort

# 3. 配置文件
ls -la src/main/resources/ 2>/dev/null
cat pom.xml 2>/dev/null | head -50

# 4. Controller 层
find src -type f -name "*Controller.java" 2>/dev/null | head -30

# 5. Service 层
find src -type f -name "*Service.java" 2>/dev/null | head -30

# 6. Repository 层
find src -type f -name "*Repository.java" 2>/dev/null | head -30
```

### 变异点检测命令
```bash
# 1. 列出所有 Controller（识别有多少个同级的 API 模块）
find src -type f -name "*Controller.java" 2>/dev/null | sort

# 2. 检查各 Controller 的路由映射（@RequestMapping, @GetMapping 等）
grep -rh "@RequestMapping\|@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping" \
  src/ 2>/dev/null | sort | uniq

# 3. 检查 Service 层的业务逻辑差异
find src -type f -name "*Service.java" 2>/dev/null | head -30
grep -rh "public.*\(.*\).*\{" src/*Service.java 2>/dev/null | head -50

# 4. 检查 Entity/Model 层的字段差异
find src -type f -name "*.java" \( -name "*Entity.java" -o -name "*Model.java" -o -name "*DO.java" \) 2>/dev/null | head -20

# 5. 检查是否有不同的数据源配置
grep -rh "@DataSource\|@Primary\|MultiDataSource" src/ 2>/dev/null

# 6. 检查事务配置差异
grep -rh "@Transactional\|@Transactional(readOnly" src/ 2>/dev/null | sort | uniq

# 7. 检查缓存配置差异
grep -rh "@Cacheable\|@CacheEvict\|@RedisCache" src/ 2>/dev/null | sort | uniq

# 8. 检查日志级别配置
grep -rh "log\.\|logger\." src/ 2>/dev/null | grep -E "debug|info|warn|error" | sort | uniq

# 9. 检查异常处理差异
grep -rh "@ExceptionHandler\|@ControllerAdvice\|@RestControllerAdvice" src/ 2>/dev/null | sort | uniq

# 10. 检查验证注解差异（@Valid, @Validated）
grep -rh "@Valid\|@Validated" src/ 2>/dev/null | sort | uniq
```

### 数据流检测
```bash
# HTTP 客户端配置（RestTemplate, WebClient, OkHttp 等）
grep -rh "RestTemplate\|WebClient\|OkHttp\|HttpClient" src/ 2>/dev/null | head -20

# 过滤器/拦截器
find src -type f -name "*Filter.java" 2>/dev/null | head -10
find src -type f -name "*Interceptor.java" 2>/dev/null | head -10

# 缓存配置（Redis, Caffeine, Guava 等）
grep -rh "@Cacheable\|@CacheEvict\|StringRedisTemplate\|RedisTemplate" src/ 2>/dev/null | head -20

# 会话管理
grep -rh "HttpSession\|@SessionAttribute\|session\|Cookie" src/ 2>/dev/null | head -20

# 安全配置（Spring Security, Shiro, JWT 等）
find src -type f \( -name "*SecurityConfig.java" -o -name "*ShiroConfig.java" -o -name "*WebSecurityConfig.java" \) 2>/dev/null | head -5

# 拦截器/过滤器链
grep -rh "FilterChain\|doFilter\|@PreAuthorize\|@Secured" src/ 2>/dev/null | head -20
```

## Go 项目

### 目录结构检测
```bash
# 1. 源码目录结构
find . -maxdepth 4 -type f -name "*.go" 2>/dev/null | grep -v vendor | head -100

# 2. 模块结构
ls -la cmd/ pkg/ internal/ 2>/dev/null

# 3. go.mod 分析
cat go.mod 2>/dev/null | head -30
```

### 变异点检测命令
```bash
# 1. 列出所有 Handler/Controller
find . -type f -name "*handler*.go" -o -name "*controller*.go" 2>/dev/null | grep -v vendor

# 2. 检查路由定义
grep -rh "HandleFunc\|Handle\|r\.Get\|r\.Post\|r\.Put\|r\.Delete" . --include="*.go" 2>/dev/null | grep -v vendor | head -30

# 3. 检查中间件链
grep -rh "Use\|Middleware\|func.*Middleware" . --include="*.go" 2>/dev/null | grep -v vendor | head -20

# 4. 检查数据库模型差异
find . -type f -name "*.go" 2>/dev/null | xargs grep -l "type.*struct" | grep -v vendor | head -20
```

## Python/Django 项目

### 目录结构检测
```bash
# 1. 源码目录结构
find . -maxdepth 4 -type f -name "*.py" 2>/dev/null | grep -v venv | grep -v __pycache__ | head -100

# 2. 应用结构
ls -la apps/ core/ models/ views/ 2>/dev/null

# 3. settings.py
find . -maxdepth 3 -name "settings.py" 2>/dev/null | head -5

# 4. requirements.txt
cat requirements.txt 2>/dev/null | head -30
```

### 变异点检测命令
```bash
# 1. 列出所有 View
find . -type f -name "views.py" 2>/dev/null | grep -v venv | grep -v __pycache__

# 2. 检查 URL 路由
grep -rh "path\|url" . --include="urls.py" 2>/dev/null | grep -v venv | head -30

# 3. 检查 Model 字段差异
grep -rh "class.*Model\|class.*Admin" . --include="models.py" 2>/dev/null | grep -v venv | head -20

# 4. 检查中间件
grep -rh "MIDDLEWARE\|middleware" . --include="settings.py" 2>/dev/null | grep -v venv

# 5. 检查缓存配置
grep -rh "CACHES\|cache" . --include="settings.py" 2>/dev/null | grep -v venv
```

## 通用后端采样命令

```bash
# 分析一个典型 Controller
cat src/main/java/com/example/controller/xxxController.java 2>/dev/null

# 分析一个典型 Service
cat src/main/java/com/example/service/xxxService.java 2>/dev/null

# 如果是极度变异，必须逐个分析所有 Controller
cat src/main/java/com/example/controller/FileFlowController.java 2>/dev/null
cat src/main/java/com/example/controller/PcScreenController.java 2>/dev/null
```