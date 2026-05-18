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