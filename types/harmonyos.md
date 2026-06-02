git grep -E -h "|# HarmonyOS 项目检测命令

## 目录结构检测

```bash
# 1. ArkTS 源码扫描（.ets 文件）
find entry/src/main/ets -maxdepth 4 -type f \( -name "git grep -E -h "|*.ets"git grep -E -h "| \) 2>/dev/null | sort

# 2. Native C++ 层扫描
find entry/src/main/cpp -maxdepth 2 -type f \( -name "git grep -E -h "|*.cpp"git grep -E -h "| -o -name "git grep -E -h "|*.h"git grep -E -h "| \) 2>/dev/null | sort

# 3. Ability 入口检测
find entry/src -type d -name "git grep -E -h "|entryability"git grep -E -h "| -o -type d -name "git grep -E -h "|entrybackupability"git grep -E -h "| 2>/dev/null

# 4. services 目录（ArkTS 服务层）
find entry/src/main/ets -type d -name "git grep -E -h "|services"git grep -E -h "| 2>/dev/null

# 5. models 目录（ArkTS 数据模型）
find entry/src/main/ets -type d -name "git grep -E -h "|models"git grep -E -h "| 2>/dev/null

# 6. pages 目录（ArkTS 页面）
find entry/src/main/ets -type d -name "git grep -E -h "|pages"git grep -E -h "| 2>/dev/null

# 7. Native 库检测（lib*.so）
ls -la entry/libs/ 2>/dev/null

# 8. 资源文件检测
find entry/src/main/resources -maxdepth 2 -type d | sort 2>/dev/null
```

## ArkTS 特性检测

```bash
# 9. 解析 ArkTS import 关系
grep -rh "git grep -E -h "|^import "git grep -E -h "| --include="git grep -E -h "|*.ets"git grep -E -h "| entry/src/main/ets/ 2>/dev/null | \
  grep "git grep -E -h "|from ['\"git grep -E -h "|]"git grep -E -h "| | \
  sed "git grep -E -h "|s/.*from ['\"git grep -E -h "|]\.\.?\//\//g; s/['\"git grep -E -h "|]//g"git grep -E -h "| | \
  sort | uniq -c | sort -rn | head -n 50

# 10. Native 模块依赖检测（lib*.so）
grep -rh "git grep -E -h "|lib.*\.so"git grep -E -h "| --include="git grep -E -h "|*.ets"git grep -E -h "| entry/src/main/ets/ 2>/dev/null

# 11. 系统 API 依赖统计（@kit.XXX 引用次数）
grep -rh "git grep -E -h "|@kit\."git grep -E -h "| --include="git grep -E -h "|*.ets"git grep -E -h "| entry/src/main/ets/ 2>/dev/null | \
  sed 's/.*@kit\./@kit./g' | cut -d' ' -f1 | cut -d';' -f1 | \
  sort | uniq -c | sort -rn

# 12. 服务依赖关系检测（DatabaseService 根基）
grep -rh "git grep -E -h "|DatabaseService"git grep -E -h "| --include="git grep -E -h "|*.ets"git grep -E -h "| entry/src/main/ets/services/ 2>/dev/null

# 13. 数据流检测（图像处理流程）
grep -rh "git grep -E -h "|PhotoPickerDialog\|RawFileToSandbox\|testNapi"git grep -E -h "| --include="git grep -E -h "|*.ets"git grep -E -h "| entry/src/main/ets/pages/ 2>/dev/null
```

## 差异点检测命令

```bash
# 1. 列出所有页面（识别有多少个同级的页面模块）
find entry/src/main/ets/pages -maxdepth 1 -type d | sort

# 2. 检查各页面的路由配置
grep -rh "git grep -E -h "|router\.\|push\|replace\|back"git grep -E -h "| entry/src/main/ets/pages/ 2>/dev/null | sort | uniq

# 3. 检查 Ability 生命周期差异
grep -rh "git grep -E -h "|onCreate\|onDestroy\|onWindowDisplay\|onFocus\|onBlur"git grep -E -h "| \
  entry/src/main/ets/entryability/*.ets 2>/dev/null | sort | uniq

# 4. 检查服务层依赖差异
grep -rh "git grep -E -h "|import.*services"git grep -E -h "| entry/src/main/ets/pages/*.ets 2>/dev/null | \
  sed 's/.*services\///g; s/\.ets.*//g' | sort | uniq

# 5. 检查 Native 调用差异（lib*.so）
grep -rh "git grep -E -h "|import.*\.so\|testNapi\|napi"git grep -E -h "| entry/src/main/ets/ 2>/dev/null | \
  sort | uniq

# 6. 检查 UI 组件使用差异
grep -rh "git grep -E -h "|@Component\|@State\|@Link\|@Prop"git grep -E -h "| entry/src/main/ets/pages/*.ets 2>/dev/null | \
  sed 's/.*@//g' | cut -d' ' -f1 | sort | uniq -c | sort -rn

# 7. 检查数据持久化差异
grep -rh "git grep -E -h "|relationalStore\|rdb\|Preferences"git grep -E -h "| entry/src/main/ets/ 2>/dev/null | \
  sort | uniq

# 8. 检查网络请求差异
grep -rh "git grep -E -h "|http\.\|request\|axios\|fetch"git grep -E -h "| entry/src/main/ets/ 2>/dev/null | \
  sort | uniq
```

## HarmonyOS 检查命令（阶段7）

```bash
# 1. 列出所有页面
find entry/src/main/ets/pages -maxdepth 1 -type f -name "git grep -E -h "|*.ets"git grep -E -h "| 2>/dev/null

# 2. 对比页面路由配置
grep -rh "git grep -E -h "|router\|push\|replace"git grep -E -h "| entry/src/main/ets/pages/*.ets 2>/dev/null

# 3. 检查 Native 调用差异
grep -rh "git grep -E -h "|testNapi\|lib.*\.so" entry/src/main/ets/pages/*.ets 2>/dev/null
```