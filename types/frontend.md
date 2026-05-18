# 前端项目检测命令

## 目录结构检测

```bash
# 1. 目录深度分析（使用4层深度）
find src -maxdepth 4 -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" \) 2>/dev/null | head -150

# 2. 一级目录
find src -maxdepth 1 -type d | sort

# 3. pages 直接子模块
find src/pages -maxdepth 1 -type d | sort

# 4. pages 二级子模块（确保穿透 pages/trace/*/list.tsx）
find src/pages -maxdepth 2 -type d | sort

# 5. 组件目录
ls -la src/components/ src/layouts/ src/router/ 2>/dev/null

# 6. services 目录结构
find src/services -maxdepth 1 -type d | sort
find src/services -type f -name "*.ts" | sort

# 7. hooks 目录
find src/hooks -maxdepth 1 -type f -name "*.ts" | sort

# 8. store 目录
find src/store -maxdepth 1 -type f -name "*.ts" | sort

# 9. router modules
ls src/router/modules/*.tsx 2>/dev/null | wc -l
ls src/router/modules/*.tsx 2>/dev/null

# 10. package.json 分析
cat package.json 2>/dev/null | head -40
```

## 页面模块检测

```bash
# 1. pages 一级子模块
find src/pages -maxdepth 1 -type d | sort

# 2. pages 二级子模块
find src/pages -maxdepth 2 -type d | sort

# 3. 所有页面 TSX 文件（按目录分组）
find src/pages -type f -name "*.tsx" | sort

# 4. 识别页面模块统一结构模式
find src/pages -name "list.tsx" | sed 's|/list\.tsx||' | xargs -I {} dirname {} | xargs -I {} basename {}
```

## 组件引用关系检测

```bash
# 1. 组件目录完整结构
find src/components -type f \( -name "*.tsx" -o -name "*.ts" \) | sort

# 2. 共享组件被引用情况
grep -rh "from ['\"]@/components/" --include="*.tsx" --include="*.jsx" src/ 2>/dev/null | \
  sed "s/.*from ['\"]@\/components\///g; s/['\"].*//g" | \
  sort | uniq -c | sort -rn

# 3. hooks 被引用情况
grep -rh "from ['\"]@/hooks/" --include="*.tsx" --include="*.ts" src/ 2>/dev/null | \
  sed "s/.*from ['\"]@\/hooks\///g; s/['\"].*//g" | \
  sort | uniq -c | sort -rn

# 4. services 被引用情况
grep -rh "from ['\"]@/services/" --include="*.tsx" --include="*.ts" src/ 2>/dev/null | \
  sed "s/.*from ['\"]@\/services\///g; s/['\"].*//g" | \
  sort | uniq -c | sort -rn | head -30

# 5. router modules 数量
ls src/router/modules/*.tsx 2>/dev/null | wc -l
```

## 前端项目变异点检测命令

```bash
# 1. 列出所有子模块（识别有多少个同级的相似模块）
find src/pages/[module] -maxdepth 1 -type d | sort

# 2. 对比各子模块的 config.ts 中的关键配置（检测步骤数差异）
find src/pages/trace -name "config.ts" -exec echo "=== {} ===" \; -exec cat {} \;

# 3. 检查动态步骤生成机制（stepsFilter、totalStep 等）
grep -rh "totalStep\|stepsFilter\|steps\[" src/pages/[module]/*/config.ts

# 4. 检查关键组件是否只在某些子模块中存在（如 ImgCrop 只在 pc-screen）
grep -rh "ImgCrop\|ImgRotate\|ImgCorrect\|ImgExtract\|DocExtract" \
  src/pages/trace/*/trace.tsx | sort | uniq -c | sort -rn

# 5. 检查枚举或常量定义（WaterMarkEnum 等）映射到不同取证类型
grep -rh "WaterMarkEnum\|watermarkType" src/pages/trace/ --include="*.ts" --include="*.tsx" | head -20

# 6. API 调用差异（不同子模块调用的 services 是否不同）
grep -rh "from ['\"]@/services/" src/pages/trace/*/list.tsx | \
  sed "s/.*from ['\"]@\/services\///g; s/['\"].*//g" | \
  sort | uniq -c | sort -rn

# 7. 路由/跳转逻辑差异
grep -rh "router\.\|navigate\|useNavigate\|history\." src/pages/trace/*/trace.tsx | \
  sort | uniq

# 8. 表单处理差异
grep -rh "onFinish\|onSubmit\|handleSubmit" src/pages/trace/*/trace.tsx | \
  sort | uniq

# 9. 条件渲染差异
grep -rh "visible\|show\|display\|render" src/pages/trace/*/trace.tsx | \
  grep -E "step|Step" | sort | uniq

# 10. 页面间状态传递方式
grep -rh "useState\|useContext\|createContext\|Provider" src/pages/trace/*/trace.tsx | \
  sort | uniq -c | sort -rn

# 11. 错误处理逻辑差异
grep -rh "message\.\|notification\|Modal\.error\|onError" src/pages/trace/*/trace.tsx | \
  sort | uniq

# 12. 加载状态处理差异
grep -rh "loading\|isLoading\|Loading\|Spin" src/pages/trace/*/trace.tsx | \
  sort | uniq
```

## 前端项目数据流检测

```bash
# HTTP 请求层
cat src/utils/request.ts 2>/dev/null || cat src/utils/api.ts 2>/dev/null

# 状态管理层
cat src/store/*.ts

# 路由守卫
cat src/hooks/useRouteGuard.ts 2>/dev/null
cat src/router/index.tsx 2>/dev/null

# 核心页面采样
cat src/pages/xxx/list.tsx 2>/dev/null
cat src/pages/xxx/create.tsx 2>/dev/null
```