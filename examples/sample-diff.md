# 扫描前后对比示例

> 本文件展示 **「原始项目 → 扫描输出」** 的最小可运行示例。
> 复制以下目录结构到本地，参考 `.project-topology.md` 的格式，照着执行一遍就能验证你用对了。

---

## 1. 准备示例项目

```bash
mkdir demo-project && cd demo-project
git init

mkdir -p src/pages/user src/pages/order src/components src/services src/hooks
mkdir -p src/main/java/com/example/controller src/main/java/com/example/service

cat > package.json <<'EOF'
{
  "name": "demo-project",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite"
  },
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "antd": "^5.0.0"
  }
}
EOF

cat > src/pages/user/list.tsx <<'EOF'
import React from 'react';
import { Table, Button } from 'antd';
import { fetchUserList } from '@/services/user';
export default function UserList() { /* ... */ }
EOF

cat > src/pages/order/list.tsx <<'EOF'
import React from 'react';
import { Table } from 'antd';
import { fetchOrderList } from '@/services/order';
export default function OrderList() { /* ... */ }
EOF

cat > src/services/user.ts <<'EOF'
import request from '@/utils/request';
export const fetchUserList = (params: any) => request.get('/api/user/list', { params });
export const createUser = (data: any) => request.post('/api/user/create', data);
EOF

cat > src/services/order.ts <<'EOF'
import request from '@/utils/request';
export const fetchOrderList = (params: any) => request.get('/api/order/list', { params });
export const cancelOrder = (id: string) => request.post(`/api/order/${id}/cancel`);
EOF

git add .
git commit -m "chore: initial demo project"
```

---

## 2. 触发扫描

在 Claude Code 中：

```
/project-topology-mapping
```

---

## 3. 期望输出

执行后会在项目根目录生成 `.project-topology.md`，包含至少以下章节（与本文件大小成比例）：

```markdown
# 项目拓扑 - demo-project

**生成时间:** 2026-06-XX
**项目类型:** 前端 (React 18 + TypeScript)
**主要语言:** TypeScript
**框架:** Ant Design Pro 风格

## 1. 模块地图

| 模块 | 路由前缀 | 子模块数 | 主要页面 |
|------|---------|---------|---------|
| user | /user | 1 | list.tsx |
| order | /order | 1 | list.tsx |

## 4. 数据流

### 1. HTTP 请求数据流
```
list.tsx → services/*.ts → utils/request.ts → 后端 API
```

## 5. 定位速查表（精简版）

| 前端页面 | 前端文件 | 调用 API | 后端文件 |
|---------|---------|---------|---------|
| 用户列表 | src/pages/user/list.tsx | /api/user/list | UserController.java |
| 订单列表 | src/pages/order/list.tsx | /api/order/list | OrderController.java |
```

---

## 4. 对比验证

| 项目状态 | 文件数 | 大小 |
|---------|-------|------|
| 原始项目 | ~6 个 | < 5 KB |
| 扫描输出（.project-topology.md） | 1 个 | ~2-5 KB |
| 记忆文件（~/.claude/memory/projects/demo-project.md） | 1 个 | ~1-2 KB |

---

## 5. 常见问题

### Q: 扫描输出比项目还小，是不是有遗漏？

A: 正常。`.project-topology.md` 是**结构化摘要**，不是源码复制。完整源码在 `src/` 里。

### Q: 为什么我的扫描输出比示例大很多？

A: 大项目（> 50 模块）会触发"快速 / 标准 / 深度"档位选择。深度档会生成更多章节。

### Q: 第二次执行会重复吗？

A: 不会。skill 会读上次记忆，git 无变更时直接复用。
