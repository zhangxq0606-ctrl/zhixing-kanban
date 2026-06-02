# 知行计划看板 - Cloudflare Pages 部署指南

## 为什么要从 Workers 迁移到 Pages？

| 对比项 | Workers | Pages |
|--------|---------|-------|
| 适用场景 | 需要服务端逻辑的API/应用 | 纯静态网站 |
| 本项目匹配度 | 不匹配（无需服务端代码） | 完美匹配 |
| 配置复杂度 | 需写 wrangler.toml + Worker 脚本 | 零配置，自动部署 |
| 免费额度 | 10万请求/天 | 无限请求 + 无限带宽 |
| 域名管理 | 占用主域名 | 可用子域名（如 kb.你的域名.com） |

---

## 第一步：推送代码到 GitHub/GitLab

```bash
git add .
git commit -m "项目初始化：知行计划团队看板"
git branch -M main
git remote add origin https://github.com/你的用户名/你的仓库名.git
git push -u origin main
```

---

## 第二步：在 Cloudflare Pages 创建项目

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com)
2. 左侧菜单点击 **Workers & Pages** → **Pages**
3. 点击 **连接到 Git**
4. 选择你的 GitHub/GitLab 账号 → 选择仓库
5. 配置构建设置：

| 配置项 | 值 |
|--------|-----|
| 框架预设 | **None**（无框架） |
| 构建命令 | **留空** |
| 构建输出目录 | **/** （根目录） |

6. 点击 **保存并部署**

> Cloudflare Pages 会自动检测到根目录的 `index.html` 并部署。

---

## 第三步：配置自定义域名（kb 子域名）

1. 部署成功后，进入项目 → **自定义域**
2. 点击 **设置自定义域**
3. 输入你的子域名，例如：`kb.你的域名.com`
4. 如果域名已在 Cloudflare 管理，DNS 记录会自动添加
5. 等待 SSL 证书自动签发（约1-2分钟）

---

## 第四步：移除旧的 Workers 部署

1. 进入 **Workers & Pages** → 找到旧的 Worker
2. 删除或禁用旧的 Worker 路由
3. 确认 Pages 的 `kb.你的域名.com` 已生效

---

## 项目最终结构

```
你的仓库/
├── index.html              # 主页面（Cloudflare Pages 自动识别）
├── Supabase 配置指南.md      # 后端配置文档
├── .gitignore               # Git 忽略规则
└── README.md                # 项目说明
```

---

## 注意事项

- 纯静态 HTML 无需任何构建步骤，Pages 会直接托管
- 每次 `git push` 到 main 分支，Pages 会自动重新部署
- Supabase 后端保持不变，无需任何修改
