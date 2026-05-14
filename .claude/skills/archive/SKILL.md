---
name: archive
description: 报道归档。将确认的报道用 Tavily 提取正文，转为 Markdown + 关键图片，存入 sources/articles/。用法：/archive [URL] 或 /archive-all（批量归档确认列表）
---

# 报道归档

## 核心原则
- 用 Tavily Extract 提取正文，不存原始 HTML
- 重要图片（演讲照片、产品图、数据图表）下载到本地
- 用 vision_analyze 给图片生成描述，确保信息不丢
- 保存为 Markdown + frontmatter，格式统一

## 存储结构

```
sources/
├── articles/                    # Markdown 归档
│   ├── 2023-09-orange-unicorn-launch.md
│   ├── 2024-11-v2g-keynote.md
│   └── ...
├── images/                      # 关键图片
│   ├── 2023-09-orange-unicorn-launch/
│   │   └── img-01-发布会现场.jpg
│   └── ...
└── TopicReportList.MD           # 索引
```

## 单篇归档流程

### 1. 提取内容

使用 Tavily Extract 深度提取 URL 正文：
```
tool: mcp__tavily__tavily_extract
参数:
  urls: ["<URL>"]
  extract_depth: "advanced"
  format: "markdown"
```

如果 Tavily 失败，回退到 WebFetch 或 curl + 手动清理。

### 2. 确定文件名

从内容中推断：
- **slug**: `YYYY-MM-<英文短横线描述>`，如 `2024-11-v2g-keynote`
- **title**: 文章标题
- **source**: 来源媒体名
- **published**: 发布日期
- **topic**: 归属的事件主题（对照 TopicReportList.MD）

### 3. 提取关键图片

识别内容中的重要图片（演讲照片、产品图、数据图表、现场照），排除：
- 网站 logo、广告、二维码
- 宽度 < 200px 的小图
- GIF 动图

下载到本地：
```bash
mkdir -p sources/images/<slug>/
curl -sL -o "sources/images/<slug>/img-01-<描述>.jpg" "<图片URL>"
```

### 4. 图片分析

对每张下载的图片调用 vision_analyze，获取详细描述。

### 5. 保存 Markdown

写入 `sources/articles/<slug>.md`：

```markdown
---
title: <标题>
source: <来源媒体>
source_url: <原始URL>
published: <YYYY-MM-DD>
archived: <今天日期>
topic: <归属事件主题>
---

<提取的正文内容>

## 图片

![简短描述](../images/<slug>/img-01-<描述>.jpg)

> **图片描述**：<vision 分析的详细描述>
```

### 6. 更新 TopicReportList.MD

在对应事件主题下追加或确认该报道条目。

### 7. 更新 goals.md

更新进度数字。

### 8. Git 提交

```bash
cd ~/projects/github/my
git add -A
git commit -m "feat: 归档报道 - <来源> <简要标题>"
```

### 9. 唯一性校验（push 前必做）

提交完后、push 前必须验证 TopicReportList 没有重复 URL：

```bash
DUPS=$(grep -oE 'https?://[^ |]+' sources/TopicReportList.MD | sort | uniq -d)
if [ -n "$DUPS" ]; then
  echo '⚠️ 发现重复 URL，请检查 TopicReportList.MD：'
  echo "$DUPS"
  exit 1
fi
echo '✅ TopicReportList URL 唯一性校验通过'
```

校验通过后才能 push：

```bash
git push origin main
```

> **背景**：曾发生过编辑 TopicReportList 时误粘贴重复行的 bug，导致进度数字虚高。这一步用于拦截类似问题。

## 批量归档

如果用户确认了多篇报道，逐个执行 Step 1-6，最后统一 git 提交一次。
