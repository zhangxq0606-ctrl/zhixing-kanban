# Supabase 配置指南

## 第一步：创建 Supabase 项目

1. 访问 [supabase.com](https://supabase.com)
2. 点击 "Start your project" 或 "Sign Up"
3. 使用 GitHub 账号登录（推荐）或邮箱注册
4. 点击 "New project"

### 填写项目信息：
```
Name: 知行计划看板
Database Password: [设置一个强密码，请妥善保管]
Region: Asia South (Singapore) - 推荐，国内访问较快
Pricing Plan: Free (免费版足够使用)
```

5. 点击 "Create new project"
6. 等待 2-3 分钟，项目创建完成

---

## 第二步：获取 API 密钥

1. 进入项目后，点击左侧菜单 **Settings** (齿轮图标)
2. 点击 **API**
3. 记录以下两个值：
   - **Project URL**: `https://xxxxx.supabase.co`
   - **anon public**: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

---

## 第三步：创建数据库表

### 方法 A：通过 SQL Editor（推荐）

1. 点击左侧菜单 **SQL Editor**
2. 点击 **New query**
3. 复制以下 SQL 脚本并粘贴
4. 点击 **Run** 执行

```sql
-- 创建项目表
CREATE TABLE projects (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    deadline DATE,
    award TEXT
);

-- 创建任务表
CREATE TABLE tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id INTEGER NOT NULL REFERENCES projects(id),
    parent_task_id UUID REFERENCES tasks(id),
    name TEXT NOT NULL,
    description TEXT,
    assignee TEXT,
    deadline DATE,
    status TEXT DEFAULT 'todo' CHECK (status IN ('todo', 'inprogress', 'completed')),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 创建索引（优化查询性能）
CREATE INDEX idx_tasks_project ON tasks(project_id);
CREATE INDEX idx_tasks_parent ON tasks(parent_task_id);
CREATE INDEX idx_tasks_status ON tasks(status);

-- 启用实时订阅（Realtime）
ALTER PUBLICATION supabase_realtime ADD TABLE tasks;

-- 关闭行级安全（无需登录即可读写）
ALTER TABLE projects DISABLE ROW LEVEL SECURITY;
ALTER TABLE tasks DISABLE ROW LEVEL SECURITY;
```

### 方法 B：通过 Table Editor

1. 点击左侧菜单 **Table Editor**
2. 点击 **New table**
3. 手动创建 `projects` 和 `tasks` 表（较繁琐，不推荐）

---

## 第四步：配置 HTML 文件

打开 `知行计划看板.html` 文件，找到第 476-477 行：

```javascript
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

替换为你在第二步记录的密钥：

```javascript
const SUPABASE_URL = 'https://xxxxx.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

---

## 第五步：测试运行

### 本地测试：
1. 直接双击打开 `知行计划看板.html`
2. 首次加载会自动初始化 7 个项目和子任务数据
3. 尝试修改任务状态、负责人等
4. 刷新页面，数据应该保持不变

### 验证实时同步：
1. 在浏览器 A 打开 HTML 文件
2. 在浏览器 B（或无痕模式）打开同一个 HTML 文件
3. 在浏览器 A 修改某个任务状态
4. 浏览器 B 应该**自动看到更新**（无需刷新）

---

## 第六步：部署到 EdgeOne Pages

### 1. 创建 GitHub 仓库

```bash
# 在项目目录执行
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/你的用户名/知行计划看板.git
git push -u origin main
```

### 2. 配置 EdgeOne Pages

1. 访问 [EdgeOne Pages](https://edgeone.ai/pages)
2. 使用 GitHub 账号登录
3. 点击 **Import project**
4. 选择你的 `知行计划看板` 仓库
5. 配置构建设置：
   ```
   Framework preset: None
   Base directory: (留空)
   Output directory: (留空)
   Build command: (留空)
   ```
6. 点击 **Deploy**

### 3. 获取访问链接

部署完成后，EdgeOne 会提供一个链接，例如：
```
https://zhixing-board.edgeone.app
```

分享给团队成员即可！

---

## 常见问题

### Q1: 数据会丢失吗？
不会。所有数据存储在 Supabase 云端，HTML 文件只是界面。即使删除 HTML 文件，重新上传后数据依然存在。

### Q2: 免费版够用吗？
完全够用。Supabase 免费版包含：
- 500MB 数据库存储（你们的数据 < 1MB）
- 2GB/月 流量（足够 7 人使用）
- 实时订阅功能

### Q3: 可以导出数据吗？
可以。在 Supabase Dashboard 点击 **Table Editor**，选择表后点击 **Export**，支持 CSV/Excel 格式。

### Q4: 如何重置所有数据？
在 SQL Editor 执行：
```sql
DELETE FROM tasks;
DELETE FROM projects;
```
然后刷新页面，会自动重新初始化。

### Q5: Realtime 不工作怎么办？
检查：
1. 是否执行了 `ALTER PUBLICATION supabase_realtime ADD TABLE tasks;`
2. 浏览器控制台是否有错误
3. 网络是否正常（Realtime 使用 WebSocket）

---

## 后续维护

### 更新成员名单
在 HTML 文件中找到：
```javascript
const TEAM_MEMBERS = ['成员 A', '成员 B', '成员 C', '成员 D', '成员 E', '成员 F', '成员 G'];
```
替换为真实姓名，保存后重新提交到 GitHub 即可自动部署。

### 添加新项目
在 `INITIAL_PROJECTS` 数组中添加新项目对象，然后：
1. 清空 Supabase 数据（见 Q4）
2. 刷新页面自动初始化

---

## 技术支持

- Supabase 文档：https://supabase.com/docs
- 项目问题：联系 antigravity
