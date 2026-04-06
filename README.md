# Exploro

**Exploro** 是一个轻量化、iOS 风格的 Web 随笔工具。

## 功能特性

- iOS 风格的用户界面设计
- GitHub OAuth 2.0 登录认证
- 实时发布和查看随笔内容
- 个人资料编辑功能
- 用户可以编辑自己的用户名、姓名、个人简介、个人网站和位置信息
- 支持删除自己发布的帖子
- 头像图片上传时会进行压缩处理，以符合存储空间限制
- 点赞功能：支持对帖子进行点赞和取消点赞，并显示点赞数

## System Architecture & Constraints

### 1. 技术架构 (Tech Stack)

- **Frontend**: Vanilla JS + CSS (iOS Design System), hosted on Vercel (Hobby Tier)
- **Backend/BaaS**: Supabase (Free Tier)
- **Auth**: GitHub OAuth 2.0
- **Database**: PostgreSQL with Row Level Security (RLS) enabled

### 2. 数据库逻辑 (Data Schema)

- `public.profiles`: 存储用户信息（id, username, avatar_url, full_name, bio, website, location），通过 Trigger 与 auth.users 同步
- `public.posts`: 存储动态（id, content, user_id, created_at），user_id 关联 profiles.id

### 3. 免费套餐红线 (Tier Constraints) 

**Supabase Free Tier:**
- Database Size: 500MB (别存太大的 Blob 或 Base64 图片)
- API Requests: 50k monthly (请求要克制，尽量合并查询)
- Auth: 50,000 MAU (目前够用，但代码要处理好登录状态持久化)

**Vercel Hobby Tier:**
- Serverless Execution: 10s timeout (如果以后加后端函数，逻辑必须快)
- Bandwidth: 100GB/month

### 4. 开发规范 (Development Rules)

- **Security**: 严禁在代码中泄露 service_role key；所有敏感操作必须通过 RLS 校验
- **Style**: 保持苹果风格的极简布局，严禁使用花里胡哨的第三方 UI 库

## 数据库表结构

### `posts` 表
- `id`: UUID - 主键，唯一标识每条帖子
- `content`: TEXT - 帖子内容
- `user_id`: UUID - 外键，关联到 profiles 表的 id
- `created_at`: TIMESTAMP - 创建时间

### `profiles` 表
- `id`: UUID - 主键，与 Supabase auth.users 表的 id 同步
- `username`: TEXT - 用户名
- `avatar_url`: TEXT - 头像链接
- `full_name`: TEXT - 全名
- `bio`: TEXT - 个人简介
- `website`: TEXT - 个人网站
- `location`: TEXT - 位置

## 关联关系

- `posts.user_id` → `profiles.id` (一对多关系)
- 通过 Supabase 的 RLS (Row Level Security) 确保用户只能操作自己的数据

## 登录逻辑

1. 用户通过 GitHub OAuth 进行身份验证
2. 登录成功后，系统获取用户信息并检查是否有对应的 profiles 记录
3. 如果没有对应的 profiles 记录，则创建一条新的记录
4. 用户可以访问和修改自己的帖子以及个人资料

## CSS 风格规范

### 颜色变量
- `--ios-bg`: #F2F2F7 (背景色)
- `--ios-card`: #FFFFFF (卡片背景色)
- `--ios-blue`: #007AFF (主要操作色)
- `--ios-text`: #000000 (文字颜色)
- `--ios-secondary`: #8E8E93 (次要文字颜色)

### 圆角大小
- 卡片圆角: 18px-20px
- 按钮圆角: 10px-20px
- 头像圆角: 50% (圆形)

### Hover 逻辑
- 帖子卡片上的删除按钮默认 opacity: 0，hover 时 opacity: 1
- 按钮有过渡效果 (transition: all 0.2s)

### 布局规范
- 最大宽度: 600px
- 内边距: 20px 16px
- 字体: -apple-system, BlinkMacSystemFont, "SF Pro Text", sans-serif
- 行高: 1.4
