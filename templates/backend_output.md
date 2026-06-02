# 后端项目输出模板

> **说明**：本文件是 **Markdown 写作骨架/参考模板**，含占位符（`<name>`、`<project-name>` 等），供 AI 写作时套用。
> **不是** Jinja2/Handlebars 等程序化模板引擎格式，不能被工具直接渲染。
> 使用时将占位符替换为项目真实值，**不要直接复制示例**。

```markdown
# 项目拓扑

**生成时间:** YYYY-MM-DD HH:mm
**项目名称:** <name>
**项目类型:** 后端
**主要语言:** Java/Golang/Python
**框架:** Spring Boot / Gin / Django

## 目录结构

```
<tree output>
```

## 模块地图

| 模块 | 包路径 | Controller数 | Service数 | 主要API |
|------|--------|-------------|---------|---------|
| 用户模块 | com.example.user | 1 | 2 | /api/user/* |
| 订单模块 | com.example.order | 2 | 3 | /api/order/* |

## 组件关系图

### 包结构树
```
src/main/java/com/example/
├── controller/          # 控制层
│   ├── UserController.java
│   └── OrderController.java
├── service/            # 服务层
│   ├── impl/          # 实现
│   └── IUserService.java
├── repository/        # 数据访问层
│   ├── UserRepository.java
│   └── OrderRepository.java
├── entity/            # 实体层
│   ├── User.java
│   └── Order.java
└── config/            # 配置层
    ├── SecurityConfig.java
    └── RedisConfig.java
```

### API 路由表
| Controller | 路径 | 主要方法 |
|-----------|------|---------|
| UserController | /api/user | login, logout, profile |
| OrderController | /api/order | create, cancel, query |

### 依赖关系图
```
Controller → Service → Repository → Database
                ↓
            Cache (Redis)
```

## 数据流

### 1. 请求数据流
```
请求 → Filter → Interceptor → Controller → Service → Repository → DB
                                    ↓
                                Redis Cache
```

### 2. 安全过滤链
```
请求 → Security Filter → Authentication → Authorization → Handler
```

### 3. 缓存策略
```
读: DB → Cache → Response
写: Request → DB → Cache (invalidate)
```

## 入口点

| 命令 | 端口 | 说明 |
|------|------|------|
| mvn spring-boot:run | 8080 | 开发服务器 |
| java -jar app.jar | 8080 | 生产环境 |

## 依赖

- 框架: Spring Boot 2.7
- 数据库: MySQL 8.0
- 缓存: Redis 6.0
- 运行时: Java 17

## 服务名映射表（必须包含）

> ⚠️ **重要：命名相似 ≠ 同一服务，必须交叉验证**

| 客户端服务名 | 客户端常量 | 服务端注册名 | 是否同一服务 |
|------------|----------|------------|------------|
| <CLIENT-NAME-1> | <CONSTANT-NAME-1> | <SERVER-NAME-1> | <是/否> |
| <CLIENT-NAME-2> | <CONSTANT-NAME-2> | <SERVER-NAME-2> | <是/否> |

**说明：**
- 客户端 name 与服务端注册名完全一致 → 同一服务
- 客户端 name 与服务端注册名不同 → **不同服务**
- 命名相似**不是**合并依据，必须验证实际注册名
```