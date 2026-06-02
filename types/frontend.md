# 前端项目检测命令

## 目录结构检测

```bash
# 1. 目录深度分析（使用4层深度）
find src -maxdepth 4 -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" \) 2>/dev/null | head -n 150

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

# 9. router modules (Ant Design Pro 风格)
ls src/router/modules/*.tsx 2>/dev/null | wc -l
ls src/router/modules/*.tsx 2>/dev/null

# 9.1 router 入口文件（通用）
git ls-files 'src/router/index.*' 'src/router/routes.*' 'src/App.tsx' 'src/App.jsx' 2>/dev/null

# 10. package.json 分析
cat package.json 2>/dev/null | head -n 40
```

> **路由发现策略（按项目类型分支）：**
>
> | 项目风格 | 路由位置 | 检测命令 |
> |---------|---------|---------|
> | **Ant Design Pro** | `src/router/modules/*.tsx` | `ls src/router/modules/*.tsx` |
> | **纯 React Router** | `src/router/index.tsx` + `<Route>` 列表 | `git grep -h "<Route" -- 'src/router/index.*' 2>/dev/null` |
> | **Next.js** | `pages/*.tsx` 或 `app/*.tsx` (App Router) | `git ls-files 'pages/*.tsx' 'app/*.tsx' 2>/dev/null` |
> | **Nuxt** | `pages/*.vue` | `git ls-files 'pages/*.vue' 2>/dev/null` |
> | **Vue Router (默认)** | `src/router/index.ts` + 路由数组 | `git ls-files 'src/router/index.*' 2>/dev/null` |
> | **Remix** | `app/routes/*.tsx` | `git ls-files 'app/routes/*.tsx' 2>/dev/null` |
> | **SvelteKit** | `src/routes/*.svelte` | `git ls-files 'src/routes/*.svelte' 2>/dev/null` |
>
> **检测步骤：** 先看 package.json 的依赖判断框架（react-router/next/nuxt/sveltekit/remix），再选对应命令。

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
  sort | uniq -c | sort -rn | head -n 30

# 5. router modules 数量
ls src/router/modules/*.tsx 2>/dev/null | wc -l
```

---

## 差异点检测

> 用于阶段 2 差异点预检 + 阶段 7 同类模块差异识别。
> 同一父模块下的子项差异越大，越需要逐个采样而不是抽样。

```bash
# 1. 列出所有子模块（识别有多少个同级的相似模块）
find src/pages/<module> -maxdepth 1 -type d | sort

# 2. 对比各子模块的 config.ts 中的关键配置（检测步骤数差异）
git ls-files 'src/pages/<module>/*/config.ts' 2>/dev/null | \
  xargs -I {} sh -c 'echo "=== {} ==="; cat {}'

# 3. 检查动态步骤生成机制
git grep -h "totalStep\|stepsFilter\|steps\[" -- 'src/pages/<module>/*/config.ts' 2>/dev/null

# 4. 检查关键组件是否只在某些子模块中存在
git grep -h "ImgCrop\|ImgRotate\|ImgCorrect\|ImgExtract\|DocExtract" \
  -- 'src/pages/<module>/*/trace.tsx' 2>/dev/null | sort | uniq -c | sort -rn

# 5. 检查枚举或常量定义映射到不同子模块
git grep -h "WaterMarkEnum\|watermarkType" -- 'src/pages/<module>/*.{ts,tsx}' 2>/dev/null | head -n 20

# 6. API 调用差异（不同子模块调用的 services 是否不同）
git grep -h "from ['\"]@/services/" -- 'src/pages/<module>/*/list.tsx' 2>/dev/null | \
  sed "s/.*from ['\"]@\/services\///g; s/['\"].*//g" | \
  sort | uniq -c | sort -rn

# 7. 路由/跳转逻辑差异
git grep -h "router\.\|navigate\|useNavigate\|history\." \
  -- 'src/pages/<module>/*/trace.tsx' 2>/dev/null | sort -u

# 8. 表单处理差异
git grep -h "onFinish\|onSubmit\|handleSubmit" \
  -- 'src/pages/<module>/*/trace.tsx' 2>/dev/null | sort -u

# 9. 条件渲染差异
git grep -h "visible\|show\|display\|render" -- 'src/pages/<module>/*/trace.tsx' 2>/dev/null | \
  grep -E "step|Step" | sort -u

# 10. 页面间状态传递方式
git grep -h "useState\|useContext\|createContext\|Provider" \
  -- 'src/pages/<module>/*/trace.tsx' 2>/dev/null | sort | uniq -c | sort -rn

# 11. 错误处理逻辑差异
git grep -h "message\.\|notification\|Modal\.error\|onError" \
  -- 'src/pages/<module>/*/trace.tsx' 2>/dev/null | sort -u

# 12. 加载状态处理差异
git grep -h "loading\|isLoading\|Loading\|Spin" \
  -- 'src/pages/<module>/*/trace.tsx' 2>/dev/null | sort -u
```

### 差异表记录格式

```markdown
### [模块名] 流程差异分析

| 子模块 | 步骤数 | config.ts 关键配置 | 核心组件 | 差异原因 |
|--------|--------|------------------|---------|---------|
| file-flow | 2 步 | totalStep=2 | DocExtract | 文档直接解析 |
| pc-screen | 3-5 步 | stepsFilter | ImgCrop | 5 步（部分屏拍屏） |
| ... | ... | ... | ... | ... |
```