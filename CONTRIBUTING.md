# 贡献指南

我们欢迎任何形式的贡献！🎉

## 行为准则

参与本项目即代表你同意遵守 [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md)。

## 我能贡献什么？

### 🐛 报告 Bug

发现 bug？请用 [Bug Report 模板](https://gitee.com/xiaoxiaoxiaoge/project-topology-mapping/issues/new?template=bug_report.yml) 提 issue。

请包含：
- 复现步骤
- 期望行为
- 实际行为
- 截图 / 错误日志（如有）
- 操作系统 / Claude Code 版本

### 💡 提出新功能

想要新功能？请用 [Feature Request 模板](https://gitee.com/xiaoxiaoxiaoge/project-topology-mapping/issues/new?template=feature_request.yml) 提 issue。

请说明：
- 解决的问题
- 期望的 API / 行为
- 替代方案（你考虑过的其他做法）
- 是否愿意自己实现

### 📖 完善文档

文档错误、翻译改进、示例补充都欢迎！

- 中英文 README 翻译
- 新增 `types/xxx.md` 覆盖更多项目类型（欢迎 KMP、SmartTV、Qt 等）
- 补充 `examples/` 中的真实案例

### 🔧 提交代码

参考下方的 [开发流程](#开发流程)。

---

## 开发流程

### 1. Fork & Clone

```bash
# 码云
git clone https://gitee.com/<your-username>/project-topology-mapping.git
cd project-topology-mapping

# 添加 upstream
git remote add upstream https://gitee.com/xiaoxiaoxiaoge/project-topology-mapping.git
```

### 2. 创建分支

```bash
git checkout -b feat/your-feature-name
# 或
git checkout -b fix/bug-description
```

分支命名规范：
- `feat/xxx` — 新功能
- `fix/xxx` — bug 修复
- `docs/xxx` — 文档变更
- `refactor/xxx` — 重构
- `chore/xxx` — 杂项（CI、依赖等）

### 3. 编写代码

#### 3.1 新增 `types/xxx.md`

如果你要支持新的项目类型：

1. 在 `types/` 下新建 `xxx.md`
2. 包含以下小节：
   - 项目类型识别
   - 目录结构检测
   - 关键特性检测
   - 差异点检测
   - 阶段 7 检查
   - 入口点
3. 在 `SKILL.md` § 项目类型速查表 加一行
4. 在 `templates/xxx_output.md` 创建对应输出模板
5. 提交 PR

#### 3.2 修改 phase 文件

**重要：保持 single source of truth 原则。**

- 检测命令放在 `types/*.md`
- 阶段文件用 `@see` 引用，不要重复命令
- 如果必须重复，注释说明原因

#### 3.3 命令跨平台要求

**所有 bash 命令必须跨 Windows / macOS / Linux。**

- ✅ 使用 `git ls-files` / `git grep` / `find`（Git Bash 提供）
- ❌ 避免 `head` / `tail`（Windows PowerShell 不存在）
- ❌ 避免 `awk -F` 复杂管道（用 `git` 等价命令）

详见 [SKILL.md § 平台兼容性](./SKILL.md#平台兼容性)。

### 4. 自我校验

提交前检查：

- [ ] `SKILL.md` 表格中是否引用了新增/修改的文件
- [ ] 是否破坏了 single source of truth（phases 重复 types 内容）
- [ ] 是否引入了跨平台不兼容命令
- [ ] CHANGELOG.md 是否更新
- [ ] 链接是否还有效（`./xxx.md` 路径）

### 5. 提交 PR

```bash
git add .
git commit -m "feat: 添加 types/kmp.md 支持 KMP 项目"
git push origin feat/your-feature-name
```

然后在码云 / GitHub 创建 Pull Request。

#### Commit Message 规范

参考 [Conventional Commits](https://www.conventionalcommits.org/zh-hans/)：

```
<type>(<scope>): <subject>

<body>

<footer>
```

**type**:
- `feat` — 新功能
- `fix` — bug 修复
- `docs` — 文档
- `style` — 格式（不影响代码）
- `refactor` — 重构
- `test` — 测试
- `chore` — 杂项

**示例：**
```
feat(types): 添加 KMP 项目类型支持

- types/kmp.md：检测 commonMain / androidMain / iosMain
- templates/kmp_output.md：对应输出模板
- SKILL.md 表格更新
- CHANGELOG.md 更新
```

### 6. Code Review

所有 PR 都会经过 review。请耐心等待，可能会有迭代修改。

---

## 项目规范

### 文件组织

```
project-topology-mapping/
├── SKILL.md             # 主索引（路由）
├── README.md            # 用户面向
├── CHANGELOG.md         # 变更日志
├── CONTRIBUTING.md      # 本文件
├── LICENSE              # MIT
├── phases/              # 7 阶段
├── types/               # 检测命令（SSOT）
├── templates/           # 输出模板
├── examples/            # 示例
└── .gitee/ + .github/   # Issue 模板
```

### 命名约定

- **目录**: 小写 + 连字符（`kmp-skill`）
- **文件**: 小写 + 下划线（`kmp_output.md`）
- **章节**: 数字编号（`2.1`, `2.2.3`）
- **变量占位符**: `<尖括号>`（如 `<project-name>`）

### Markdown 风格

- ATX 标题（`#` 而非 `===`）
- 表格对齐：用 `|` 即可，无需空格对齐
- 代码块：必须标注语言（```` ```bash ````）
- 链接：用相对路径（`./xxx.md`）

---

## 第一次贡献？

找 `good first issue` 标签的 issue 开始：

[Good First Issues](https://gitee.com/xiaoxiaoxiaoge/project-topology-mapping/issues?q=label%3A%22good+first+issue%22)

---

## 社区

- 💬 讨论：[Discussion 区](https://gitee.com/xiaoxiaoxiaoge/project-topology-mapping/issues)
- 📧 邮件：<your-email@example.com>
- 🐛 Bug 报告：[GitHub Issues](https://gitee.com/xiaoxiaoxiaoge/project-topology-mapping/issues)

---

## 致谢

感谢所有 [贡献者](https://gitee.com/xiaoxiaoxiaoge/project-topology-mapping/contributors)！🎉
