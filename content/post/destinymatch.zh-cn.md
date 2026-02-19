---
title: "从需求到部署：我用 Claude Code 三天开发了一个 AI 应用"
date: 2026-02-19T12:48:59+08:00
draft: false
tags: [ai, claude code, nodejs]
categories: [tech]
---
> 完整记录 Destiny Match 项目的开发流程：需求定义 → AI 原型 → 工程化 → 生产部署

## 前言

最近我完成了一个有趣的 Side Project —— **Destiny Match（命运匹配）**。这是一个 AI 驱动的娱乐应用：用户上传自己的照片，AI 生成一张"未来伴侣"的照片，并给出般配度分析和缘分解读。

作为一个全栈开发经验不多的工程师，我用 **3 天时间**完成了从需求文档到生产部署的全过程。这要归功于一套新的开发工作流：**Claude Code Plan Mode + AI Studio 原型 + 渐进式工程化**。

这篇文章记录了整个开发流程，适合想了解 AI 辅助开发的初学者参考。

---

## 项目概览

**Destiny Match** 的技术栈：

| 层级 | 技术 | 说明 |
|-----|------|-----|
| 前端 | React 19 + TypeScript + Vite | 函数式组件 + Hooks |
| 后端 | Express + TypeScript | RESTful API |
| AI 图像 | 即梦 (Dreamina) | 字节跳动图像生成 API |
| AI 文本 | DeepSeek | 硅基流动提供的文本分析 API |
| 部署 | Nginx + Node.js | 手动部署，PM2 进程管理 |

**核心功能**：
- 6 步用户流程：首页 → 隐私协议 → 照片上传 → 风格选择 → 生成中 → 结果展示
- 4 种伴侣风格：温柔型、阳光型、知性型、神秘型
- 分享功能：生成分享卡片（html-to-image）
- 使用限制：每日 3 次生成（防滥用）

---

## Phase 1：需求定义（Plan Mode）

### 从一句话到完整文档

项目的起点非常简单。我在 `docs/ideal.md` 中写下了这个想法：

```markdown
我想做一个 web 应用，用户上传自己的照片，然后应用进行预测，
生成用户未来伴侣的照片，并给出算命类型的描述，说明两人的般配度。

这是一个趣味性的应用，并不是严肃的婚恋软件，
所以预测出的照片不需要匹配用户本身的相貌。
般配度是一种似是而非的大概描述，让人看的舒服。
```

接下来，我使用 **Claude Code 的 Plan Mode** 来完成需求定义。

### 什么是 Plan Mode？

Plan Mode 是 Claude Code 的一种工作模式，它会：
1. 先深入理解你的需求
2. 生成详细的实现计划
3. 等你确认后再执行

这非常适合项目启动阶段——你可以和 AI 协作梳理思路，而不是直接写代码。

### 三轮对话产出三份文档

**第一轮：用户流程**

我要求 Claude 根据 ideal.md 创建用户流程文档。经过几轮讨论，我们产出了 `docs/user-flow.md`（667 行）。

这份文档包含了：
- 用户旅程地图（6 个阶段的情绪曲线）
- Mermaid 流程图定义完整交互流程
- 每个页面的详细布局（ASCII 图示）
- 异常流程处理（网络错误、限流、生成失败）

关键决策示例：
```markdown
### 3.1 首页 (Home)
#### 页面布局
┌─────────────────────────────────────┐
│           [星空背景动效]             │
│                                     │
│            ✨ 命运匹配 ✨            │
│                                     │
│    探索你的缘分，预见未来的 TA        │
│                                     │
│         [命运之门动画]               │
│                                     │
│      ┌─────────────────┐           │
│      │   开启命运之门   │           │
│      └─────────────────┘           │
└─────────────────────────────────────┘
```

**第二轮：功能需求**

接着产出 `docs/functional-requirements.md`（389 行），定义了：
- 4 个功能模块（用户/照片/AI 生成/结果展示）
- P0/P1/P2 优先级分类
- AI 服务选型（即梦 + DeepSeek）

**第三轮：技术规范**

最后产出 `docs/technical-spec.md`（872 行），包含：
- 系统架构图
- 数据模型定义
- API 接口设计
- 第三方服务集成方案

### Plan Mode 的价值

这个阶段花了大约 **2 小时**，但产出了 90KB 的文档。好处是：

1. **思路清晰**：在写代码前，所有交互细节、数据结构、API 接口都已确定
2. **降低返工**：后期开发时很少出现"这个没考虑到"的情况
3. **便于沟通**：如果要找合作者，这些文档可以直接用

---

## Phase 2：原型生成（AI Studio）

### 从文档到可运行的原型

有了详细的需求文档，下一步是生成产品原型。我的流程是：

```
user-flow.md → https://stitch.withgoogle.com/ → AI Studio → 原型代码
```

**Stitch 的作用**：
将文本需求转化为视觉设计。我上传了 user-flow.md，Stitch 生成了每个页面的设计图。

**AI Studio 的作用**：
基于设计图生成完整的前端代码（HTML + TSX）。

### 初始原型的特征

Git 提交记录显示了这次生成的痕迹：

```bash
$ git show 126488a --stat
commit 126488acaa16275c9f00502a3fe452443ec86e18
Author: mahaoliang <mahaoliang@gmail.com>
Date:   Mon Feb 16 20:28:25 2026 +0800

    Initial commit: Destiny Match - AI Powered Romance Discovery

    Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

初始提交包含：
- **30 个文件**，5908 行代码
- 完整的 6 步用户流程（Home → Privacy → Upload → SelectVibe → Loading → Result）
- Glassmorphism 设计风格（毛玻璃效果）
- Tailwind CSS + Material Icons
- Mock AI 服务（前端直接调用，返回假数据）

### 原型代码的局限性

虽然原型能跑通，但作为生产代码还有明显问题：

1. **纯前端架构**：API 密钥直接暴露在浏览器端
2. **Base64 图片**：用户上传的照片以 base64 存储，占用内存大
3. **无持久化**：数据存在 localStorage，换浏览器就丢失
4. **无使用限制**：任何人可以无限次调用，API 费用不可控

这些问题在 Phase 3 逐一解决。

---

## Phase 3：工程化（Claude Code 渐进增强）

### 架构重构：从纯前端到前后端分离

**决策记录**（来自 technical-spec.md）：

| 问题 | 方案 | 原因 |
|-----|------|-----|
| API 密钥安全 | 添加 Express 服务端 | 密钥只在服务端，前端通过 API 调用 |
| 图片存储 | 文件系统 + URL | 避免 base64 占用内存，支持大图片 |
| 使用限制 | 文件-based 限流 | 每个用户每日 3 次，防 API 滥用 |
| 并发处理 | Node.js + 异步 | 支持 100+ 并发用户 |

### 关键改动详解

**改动 1：图片存储流程**

原型阶段：
```typescript
// 前端直接读 base64
const reader = new FileReader();
reader.onload = () => setImage(reader.result); // base64 字符串
reader.readAsDataURL(file);
```

生产阶段：
```typescript
// 1. 前端上传到服务端
const formData = new FormData();
formData.append('image', file);
const response = await fetch('/api/upload-image', {
  method: 'POST',
  body: formData
});
const { imageUrl } = await response.json(); // 返回 /images/xxx.jpg

// 2. AI 生成后保存到磁盘
const partnerImageBuffer = await generatePartnerImage(imageUrl, vibe);
const partnerImageUrl = saveImage(partnerImageBuffer); // 返回 URL
```

**改动 2：限流实现**

```typescript
// server/src/services/usageTracker.ts
export const checkAndIncrementUsage = (userId: string) => {
  const today = getToday(); // "2026-02-17"
  const usage = readUsage(userId); // { count: 2, date: "2026-02-17" }

  if (!usage || usage.date !== today) {
    writeUsage(userId, { count: 1, date: today });
    return { allowed: true, remaining: 2 };
  }

  if (usage.count >= 3) {
    return { allowed: false, remaining: 0 };
  }

  usage.count++;
  writeUsage(userId, usage);
  return { allowed: true, remaining: 3 - usage.count };
};
```

**改动 3：AI 调用并行化**

```typescript
// 图像生成和文本分析并行执行，减少等待时间
const [partnerImageBase64, analysis] = await Promise.all([
  generatePartnerImage(userImageBase64, vibe),    // 即梦 API
  generateDestinyAnalysis(vibe, score)             // DeepSeek API
]);
```

### 遇到的坑和解决方案

**坑 1：即梦 API 需要公网图片地址**

本地开发时，用户上传的图片是 `localhost:3000/images/xxx.jpg`，即梦 API 无法访问。

**解决**：使用 ngrok 暴露本地服务
```bash
# 安装 ngrok
brew install ngrok

# 启动 ngrok
ngrok http 3002

# 修改 .env
PUBLIC_BASE_URL=https://abc123.ngrok.io
```

**坑 2：iPhone 的 HEIC 格式**

iPhone 默认拍摄 HEIC 格式，浏览器无法直接预览。

**解决**：添加 heic2any 转换
```typescript
import heic2any from 'heic2any';

const convertHeicToJpeg = async (file: File): Promise<File> => {
  if (file.type === 'image/heic' || file.name.endsWith('.heic')) {
    const converted = await heic2any({
      blob: file,
      toType: 'image/jpeg',
      quality: 0.9
    });
    return new File([converted], file.name.replace('.heic', '.jpg'), {
      type: 'image/jpeg'
    });
  }
  return file;
};
```

**坑 3：分享卡片的图片跨域**

html-to-image 生成分享卡片时，外部图片（如 Unsplash）可能跨域失败。

**解决**：服务端代理图片，统一使用同域 URL
```typescript
// 生成时获取图片为 base64
const fetchImageAsBase64 = async (url: string) => {
  const response = await fetch(url);
  const blob = await response.blob();
  return new Promise((resolve) => {
    const reader = new FileReader();
    reader.onloadend = () => resolve(reader.result);
    reader.readAsDataURL(blob);
  });
};
```

---

## Phase 4：部署

### 服务器配置

**环境**：Ubuntu 22.04 + Nginx + Node.js 20

**部署架构**：
```
用户浏览器
    │
    ▼
Nginx (80/443)
    ├── /          → 前端静态文件 (client/dist)
    ├── /api/*     → 后端服务 (localhost:3001)
    └── /images/*  → 后端服务 (localhost:3001)
```

### Nginx 配置要点

```nginx
server {
    listen 443 ssl;
    server_name destiny.mahaoliang.tech;

    # 前端静态文件
    location / {
        root /home/ubuntu/works/destinymatch/client/dist;
        try_files $uri $uri/ /index.html;
    }

    # 后端 API 代理（长超时）
    location /api/ {
        proxy_pass http://localhost:3001/;
        proxy_read_timeout 60s;  # AI 生成可能较慢
        client_max_body_size 10M;  # 大图片上传
    }

    # 图片服务代理
    location /images/ {
        proxy_pass http://localhost:3001/images/;
        expires 1d;
    }
}
```

### 启动脚本

```bash
# 生产环境启动（加载 .env.production）
cd /home/ubuntu/works/destinymatch/server
export $(cat .env.production | grep -v '^#' | xargs) && \
  nohup node dist/index.js > ../logs/app.log 2>&1 &
```

### 常用运维命令

```bash
# 查看日志（排错第一选择）
tail -f /home/ubuntu/works/destinymatch/logs/app.log

# 重启服务
pkill -f "node dist/index.js" && \
  cd /home/ubuntu/works/destinymatch/server && \
  export $(cat .env.production | grep -v '^#' | xargs) && \
  nohup node dist/index.js > ../logs/app.log 2>&1 &

# 检查服务状态
ps aux | grep "node dist/index.js"
curl http://localhost:3001/api/health
```

---

## 成果与经验总结

### 开发数据

| 指标 | 数值 |
|-----|------|
| 开发周期 | 3 天 |
| 代码提交 | 24 次 |
| 前端代码 | ~6000 行 |
| 后端代码 | ~2000 行 |
| 文档 | ~90KB（6 个 markdown） |

### 工作流价值

1. **Plan Mode 降低返工**
   - 先写文档再写代码，需求变更减少 80%
   - 技术选型在文档阶段完成，避免中途重构

2. **AI Studio 快速启动**
   - 初始原型 1 小时内可运行
   - 有代码基础后，增量修改比从零开始快

3. **渐进式工程化**
   - 先跑通流程，再优化架构
   - 每个改动都有明确的理由（安全/性能/体验）

### 适合什么项目？

这种工作流最适合：
- **MVP 验证**：快速把想法变成可演示的产品
- **Side Project**：个人开发者资源有限，用 AI 补足
- **原型迭代**：需要频繁调整需求的产品初期

不适合：
- **大型团队协作**：文档需要多人评审，AI 生成的代码风格不统一
- **强安全要求**：金融、医疗等领域，AI 可能忽略合规细节
- **性能敏感**：AI 倾向于写"能跑"的代码，而非最优实现

---

## 结语

Destiny Match 是一个小项目，但它验证了一种新的开发范式：**AI 辅助的全流程开发**。

从需求文档到生产部署，AI 工具在每个阶段都提供了 10 倍速的效率提升。但关键点在于——**人类仍然是决策者**。Plan Mode 的文档需要我确认，AI Studio 的代码需要我 review，架构调整需要我判断。

AI 不会取代开发者，但会用 AI 的开发者会取代不会用的。

---

**项目地址**：https://destiny.mahaoliang.tech

**技术文档**：项目根目录 `docs/` 文件夹包含完整的设计文档，欢迎参考。


