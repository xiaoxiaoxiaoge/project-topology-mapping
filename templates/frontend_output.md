# 前端项目输出模板

```markdown
# 项目拓扑

**生成时间:** YYYY-MM-DD HH:mm
**项目名称:** <name>
**项目类型:** 前端
**主要语言:** <primary language(s)>
**框架:** <if detected>

## 目录结构

```
<tree output>
```

## 模块地图

| 模块 | 路由前缀 | 子模块数 | 主要页面 |
|------|----------|---------|----------|
| 首页 | /home | 1 | index.tsx |
| 业务模块 | /xxx | N | page1, page2 |

## 组件关系图

### 组件层级树
```
src/
├── components/
│   ├── Icons.tsx           # 图标 (223引用)
│   └── ...
```

### 页面模块统一结构模式
```
trace*/                    # 统一结构（适用于 X 个子模块）
├── list.tsx
├── detail.tsx
├── create.tsx
├── modify.tsx
└── components/
```

### 共享组件引用表
| 组件 | 路径 | 被引用次数 |
|------|------|-----------|
| Icons | components/Icons.tsx | 223 |
| CustomFooterToolbar | components/CustomFooterToolbar.tsx | 70 |

### 组件依赖关系示例
```
页面 → hooks → services → API
```

## 数据流

### 1. HTTP请求数据流
```
请求 → 拦截器 → API → 响应拦截器 → 业务代码
```

### 2. Store状态数据流
```
useUserStore: token/userinfo/userMenus → sessionStorage
useThemeStore: layout/colors → 内存
useConfigStore: systemConfig → 内存
```

### 3. 路由守卫数据流
```
访问 → Token校验 → 菜单加载 → 权限码校验 → 放行/重定向
```

### 4. 核心页面数据流
（选取项目中最具代表性的页面，绘制完整数据流图）

### 5. 模块间关系
（展示核心模块间的依赖和调用关系）

## 入口点

| 命令 | 端口 | 说明 |
|------|------|------|
| pnpm dev | 5173 | 开发服务器 |

## 路由模块

| 文件 | 对应模块 |
|------|---------|
| home.tsx | 首页 |

## 依赖

- 框架: React 18
- 运行时: Node 20
```