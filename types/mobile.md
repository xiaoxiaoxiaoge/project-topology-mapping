git grep -E -h "|# 移动端项目检测命令

> Single source of truth for mobile project scanning.
> 适用：Android / iOS / React Native / Flutter / 跨平台框架（uni-app、Taro 等）

---

## 项目类型识别

```bash
# Android 原生
ls -la build.gradle build.gradle.kts settings.gradle* app/build.gradle* 2>/dev/null

# iOS 原生
ls -la *.xcodeproj *.xcworkspace Podfile 2>/dev/null

# React Native
ls -la package.json metro.config.js react-native.config.js 2>/dev/null
git ls-files '*.tsx' '*.ts' 2>/dev/null | head -n 1 | xargs grep -l "git grep -E -h "|react-native"git grep -E -h "| 2>/dev/null

# Flutter
ls -la pubspec.yaml 2>/dev/null
git ls-files 'lib/*.dart' 2>/dev/null | head -n 1

# 跨平台 (uni-app / Taro)
ls -la src/manifest.json 2>/dev/null            # uni-app
ls -la config/index.js project.config.json 2>/dev/null  # Taro
```

| 特征文件 | 项目类型 |
|---------|---------|
| `app/build.gradle` + `AndroidManifest.xml` | Android |
| `*.xcodeproj` + `Info.plist` | iOS |
| `package.json` + `react-native` | React Native |
| `pubspec.yaml` + `lib/*.dart` | Flutter |
| `src/manifest.json` (uni-app) 或 `project.config.json` (Taro) | 跨平台 |

---

## Android 检测命令

### 目录结构

```bash
# 1. Java/Kotlin 源码
git ls-files 'app/src/main/java/**/*.java' 'app/src/main/java/**/*.kt' 2>/dev/null | head -n 150

# 2. 资源文件
git ls-files 'app/src/main/res/*' 2>/dev/null | head -n 50

# 3. AndroidManifest
cat app/src/main/AndroidManifest.xml 2>/dev/null

# 4. Gradle 配置
cat app/build.gradle app/build.gradle.kts 2>/dev/null | head -n 50

# 5. Activity / Fragment / Service
git ls-files 'app/src/main/java/**/*Activity.java' 'app/src/main/java/**/*Fragment.java' \
             'app/src/main/java/**/*Service.java' 'app/src/main/java/**/*ViewModel.java' 2>/dev/null

# 6. Adapter / RecyclerView
git ls-files 'app/src/main/java/**/*Adapter.java' 'app/src/main/java/**/*ViewHolder.java' 2>/dev/null
```

### 关键特性

```bash
# 1. 第三方库（build.gradle dependencies）
git grep -h "git grep -E -h "|implementation\|api"git grep -E -h "| -- '*.gradle' '*.gradle.kts' 2>/dev/null | head -n 30

# 2. 权限使用
git grep -h "git grep -E -h "|uses-permission"git grep -E -h "| -- 'AndroidManifest.xml' 2>/dev/null

# 3. 组件声明
git grep -h "git grep -E -h "|android:name"git grep -E -h "| -- 'AndroidManifest.xml' 2>/dev/null

# 4. 广播接收器
git ls-files 'app/src/main/java/**/*Receiver.java' 2>/dev/null

# 5. ContentProvider
git ls-files 'app/src/main/java/**/*Provider.java' 2>/dev/null
```

### 差异点检测

```bash
# 1. 列出所有 Activity（识别多个入口模块）
git ls-files 'app/src/main/java/**/*Activity.java' 2>/dev/null | sort

# 2. Activity 间跳转关系
git grep -h "git grep -E -h "|startActivity\|Intent("git grep -E -h "| -- '*.java' '*.kt' 2>/dev/null | head -n 30

# 3. Fragment 差异
git ls-files 'app/src/main/java/**/*Fragment.java' 2>/dev/null

# 4. ViewModel 依赖
git grep -h "git grep -E -h "|ViewModelProvider\|by viewModels"git grep -E -h "| -- '*.java' '*.kt' 2>/dev/null | sort -u

# 5. 网络框架（Retrofit / OkHttp / Volley）
git grep -h "git grep -E -h "|Retrofit\|OkHttp\|Volley\|@GET\|@POST"git grep -E -h "| -- '*.java' '*.kt' 2>/dev/null | sort -u | head -n 20

# 6. 数据库（Room / SQLite / Realm）
git grep -h "git grep -E -h "|@Entity\|@Dao\|RoomDatabase\|SQLiteOpenHelper\|RealmObject"git grep -E -h "| \
  -- '*.java' '*.kt' 2>/dev/null | sort -u

# 7. 协程 / RxJava
git grep -h "git grep -E -h "|CoroutineScope\|launch\|async\|Observable\|Flowable"git grep -E -h "| \
  -- '*.java' '*.kt' 2>/dev/null | sort -u | head -n 20

# 8. 依赖注入（Hilt / Dagger / Koin）
git grep -h "git grep -E -h "|@Inject\|@HiltAndroidApp\|@Module\|@Provides"git grep -E -h "| -- '*.java' '*.kt' 2>/dev/null
```

### 阶段 7 检查

```bash
# 同类 Activity 模块差异
git ls-files 'app/src/main/java/**/*Activity.java' 2>/dev/null

# 各 Activity 的功能描述（AndroidManifest）
git grep -B1 -A3 "git grep -E -h "|android:name="git grep -E -h "| -- 'AndroidManifest.xml' 2>/dev/null

# 页面间传递数据
git grep -h "git grep -E -h "|putExtra\|getStringExtra\|getParcelableExtra"git grep -E -h "| -- '*.java' '*.kt' 2>/dev/null | sort -u
```

---

## iOS 检测命令

### 目录结构

```bash
# 1. Swift 源码
git ls-files '*.swift' 2>/dev/null | head -n 150

# 2. Objective-C 源码
git ls-files '*.m' '*.h' 2>/dev/null | head -n 50

# 3. Storyboard / XIB
git ls-files '*.storyboard' '*.xib' 2>/dev/null

# 4. Info.plist
cat Info.plist 2>/dev/null

# 5. Podfile
cat Podfile 2>/dev/null

# 6. ViewController
git ls-files '*ViewController.swift' '*ViewController.m' 2>/dev/null
```

### 关键特性

```bash
# 1. CocoaPods 依赖
git grep -h "git grep -E -h "|^pod "git grep -E -h "| -- 'Podfile' 2>/dev/null

# 2. Swift Package Manager
cat Package.swift 2>/dev/null | head -n 50

# 3. 权限（Info.plist）
git grep -h "git grep -E -h "|NS.*UsageDescription"git grep -E -h "| -- 'Info.plist' 2>/dev/null

# 4. URL Scheme
git grep -h "git grep -E -h "|CFBundleURLSchemes\|CFBundleURLTypes"git grep -E -h "| -- 'Info.plist' 2>/dev/null
```

### 差异点检测

```bash
# 1. ViewController 列表
git ls-files '*ViewController.swift' 2>/dev/null | sort

# 2. 页面间导航
git grep -h "git grep -E -h "|navigationController?.pushViewController\|present(\|performSegue"git grep -E -h "| -- '*.swift' 2>/dev/null | head -n 30

# 3. Storyboard Segue
git grep -h "git grep -E -h "|segue.identifier\|performSegue"git grep -E -h "| -- '*.swift' '*.storyboard' 2>/dev/null | head -n 20

# 4. 网络框架（Alamofire / URLSession）
git grep -h "git grep -E -h "|Alamofire\|URLSession\|Moya"git grep -E -h "| -- '*.swift' 2>/dev/null | sort -u

# 5. 持久化（CoreData / Realm / UserDefaults）
git grep -h "git grep -E -h "|NSManagedObject\|@NSManaged\|Realm\|UserDefaults"git grep -E -h "| -- '*.swift' 2>/dev/null | sort -u

# 6. 响应式框架（Combine / RxSwift）
git grep -h "git grep -E -h "|@Published\|PassthroughSubject\|Observable"git grep -E -h "| -- '*.swift' 2>/dev/null | sort -u | head -n 20
```

---

## React Native 检测命令

### 目录结构

```bash
# 1. 源码
git ls-files 'src/*.tsx' 'src/*.ts' '*.tsx' '*.ts' 2>/dev/null | head -n 150

# 2. 屏幕 (Screens)
git ls-files 'src/screens/*' '*Screen.tsx' '*Screen.ts' 2>/dev/null

# 3. 组件
git ls-files 'src/components/*' 2>/dev/null

# 4. 导航
git ls-files '*Navigator*' '*Router*' 2>/dev/null

# 5. 状态管理
git ls-files 'src/store/*' 'src/redux/*' 'src/zustand/*' 2>/dev/null

# 6. 业务 API
git ls-files 'src/services/*' 'src/api/*' 2>/dev/null
```

### 关键特性

```bash
# 1. package.json 依赖
cat package.json | head -n 60

# 2. 路由库
git grep -h "git grep -E -h "|react-navigation\|react-native-navigation\|@react-navigation"git grep -E -h "| \
  -- 'package.json' '*.ts' '*.tsx' 2>/dev/null | sort -u | head -n 10

# 3. 状态管理
git grep -h "git grep -E -h "|redux\|zustand\|mobx\|recoil\|jotai"git grep -E -h "| -- 'package.json' 2>/dev/null

# 4. UI 库
git grep -h "git grep -E -h "|native-base\|react-native-elements\|tamagui"git grep -E -h "| -- 'package.json' 2>/dev/null

# 5. Native Module 引用（Android 端）
git grep -h "git grep -E -h "|NativeModules\.\|requireNativeComponent"git grep -E -h "| -- '*.tsx' '*.ts' 2>/dev/null | sort -u | head -n 20

# 6. Native Module 实现（Android 端）
git grep -h "git grep -E -h "|ReactContextBaseJavaModule\|@ReactMethod"git grep -E -h "| -- '*.java' '*.kt' 2>/dev/null | sort -u

# 7. iOS Native Module
git grep -h "git grep -E -h "|RCT_EXPORT_MODULE\|RCT_EXPORT_METHOD\|RCTBridgeModule"git grep -E -h "| -- '*.m' '*.swift' 2>/dev/null
```

### 差异点检测

```bash
# 1. 列出所有 Screen
git ls-files '*Screen.tsx' 2>/dev/null | sort

# 2. Screen 使用的 navigation 类型
git grep -h "git grep -E -h "|Stack.Screen\|Tab.Screen\|Drawer.Screen"git grep -E -h "| -- '*.tsx' '*.ts' 2>/dev/null | sort -u

# 3. API 调用差异
git grep -h "git grep -E -h "|from ['\"git grep -E -h "|]@/services/\|from ['\"git grep -E -h "|]@/api/"git grep -E -h "| -- '*.tsx' '*.ts' 2>/dev/null | \
  sed "git grep -E -h "|s/.*from ['\"git grep -E -h "|]//g; s/['\"git grep -E -h "|].*//g"git grep -E -h "| | sort | uniq -c | sort -rn | head -n 20

# 4. 状态管理使用
git grep -h "git grep -E -h "|useSelector\|useDispatch\|useStore\|useRecoilState"git grep -E -h "| -- '*.tsx' '*.ts' 2>/dev/null | sort -u | head -n 20

# 5. Hooks 使用
git grep -h "git grep -E -h "|useEffect\|useState\|useContext\|useQuery"git grep -E -h "| -- '*.tsx' '*.ts' 2>/dev/null | sort -u | head -n 20

# 6. Platform 特定代码
git grep -rn "git grep -E -h "|Platform.OS"git grep -E -h "| -- '*.tsx' '*.ts' 2>/dev/null | head -n 20
```

---

## Flutter 检测命令

### 目录结构

```bash
# 1. Dart 源码
git ls-files 'lib/*.dart' 'lib/**/*.dart' 2>/dev/null | head -n 150

# 2. pubspec 依赖
cat pubspec.yaml 2>/dev/null

# 3. Widget 列表
git ls-files 'lib/widgets/*' 2>/dev/null

# 4. Screen / Page
git ls-files 'lib/screens/*' 'lib/pages/*' 2>/dev/null

# 5. 状态管理
git ls-files 'lib/blocs/*' 'lib/providers/*' 'lib/riverpod/*' 'lib/getx/*' 2>/dev/null

# 6. 模型
git ls-files 'lib/models/*' 2>/dev/null

# 7. 服务
git ls-files 'lib/services/*' 'lib/repositories/*' 2>/dev/null
```

### 关键特性

```bash
# 1. 路由
git grep -h "git grep -E -h "|MaterialApp\|GetMaterialApp\|Navigator.push\|Get.to"git grep -E -h "| -- '*.dart' 2>/dev/null | sort -u

# 2. 状态管理方案
git grep -h "git grep -E -h "|BlocProvider\|Provider.of\|ConsumerWidget\|StateNotifier\|GetxController"git grep -E -h "| -- '*.dart' 2>/dev/null | sort -u

# 3. 网络
git grep -h "git grep -E -h "|Dio\|http.get\|http.post\|GraphQLClient"git grep -E -h "| -- '*.dart' 2>/dev/null | sort -u | head -n 20

# 4. 持久化
git grep -h "git grep -E -h "|Hive\|sqflite\|SharedPreferences\|Isar"git grep -E -h "| -- '*.dart' 2>/dev/null | sort -u

# 5. Platform Channel（原生交互）
git grep -h "git grep -E -h "|MethodChannel\|EventChannel"git grep -E -h "| -- '*.dart' 2>/dev/null | sort -u
```

### 差异点检测

```bash
# 1. Screen 列表
git ls-files 'lib/screens/*.dart' 2>/dev/null | sort

# 2. Screen 间跳转
git grep -h "git grep -E -h "|Navigator.push\|Get.to\|Get.off"git grep -E -h "| -- '*.dart' 2>/dev/null | sort -u

# 3. Widget 复用度
git ls-files 'lib/widgets/*.dart' 2>/dev/null | \
  xargs -I {} sh -c 'echo "git grep -E -h "|{}: $(grep -l "git grep -E -h "|import"git grep -E -h "| {} | head -n 1)"git grep -E -h "|' | head -n 20

# 4. 状态管理使用
git grep -h "git grep -E -h "|BlocBuilder\|BlocConsumer\|Consumer\|GetBuilder\|Obx"git grep -E -h "| -- '*.dart' 2>/dev/null | sort -u | head -n 20

# 5. 异步模式
git grep -h "git grep -E -h "|FutureBuilder\|StreamBuilder\|async\* \|await "git grep -E -h "| -- '*.dart' 2>/dev/null | sort -u | head -n 20
```

---

## 跨平台框架 (uni-app / Taro)

### uni-app

```bash
# 1. 页面配置
cat src/manifest.json pages.json 2>/dev/null

# 2. 页面文件
git ls-files 'src/pages/*' 2>/dev/null

# 3. 组件
git ls-files 'src/components/*' 2>/dev/null

# 4. 跨端条件编译
git grep -h "git grep -E -h "|#ifdef MP-WEIXIN\|#ifdef H5\|#ifdef APP-PLUS"git grep -E -h "| -- '*.vue' 2>/dev/null

# 5. 平台 API 差异（uni.* vs wx.* vs my.*）
git grep -h "git grep -E -h "|uni\.\|wx\.\|my\."git grep -E -h "| -- '*.js' '*.ts' '*.vue' 2>/dev/null | \
  sed 's/.*\(uni\.\|wx\.\|my\.\)\([a-zA-Z]*\).*/\1\2/g' | sort | uniq -c | sort -rn
```

### Taro

```bash
# 1. 配置文件
cat config/index.js project.config.json 2>/dev/null

# 2. 页面
git ls-files 'src/pages/*' 2>/dev/null

# 3. 跨端条件
git grep -h "git grep -E -h "|process.env.TARO_ENV" -- '*.tsx' '*.jsx' 2>/dev/null
```

---

## 移动端通用采样

```bash
# 读取一个典型 Activity / ViewController / Screen 完整内容
cat app/src/main/java/com/example/.../MainActivity.java 2>/dev/null
cat lib/screens/home_screen.dart 2>/dev/null
cat src/screens/HomeScreen.tsx 2>/dev/null

# 高度变异：必须逐个采样
cat app/src/main/java/com/example/.../ListActivity.java
cat app/src/main/java/com/example/.../DetailActivity.java
```

---

## 入口点

| 框架 | 启动命令 | 调试端口 |
|------|---------|---------|
| Android | `gradle assembleDebug` / `./gradlew installDebug` | adb 5037 |
| iOS | `xcodebuild` / `pod install` | lldb |
| React Native | `npx react-native run-android` | Metro 8081 |
| Flutter | `flutter run` | Observatory 8181 |
| uni-app | `npm run dev:%s` (`mp-weixin`/`h5`/`app`) | - |
| Taro | `npm run dev:weapp` | - |
