---
name: calibrate
description: 校准已有报道。检查现有 61 篇报道的链接有效性、内容相关性，将旧 HTML 转为 Markdown + 图片新格式。用法：/calibrate
---

# 校准已有报道

## 目标
将 sources/ 下现有的 60 个原始 HTML/PDF 文件全部转换为新的 Markdown + 图片格式，同时校验每篇报道的有效性。

## 校准流程

### 1. 列出所有待校准文件

```bash
ls sources/*_offline.html sources/*_offline.pdf 2>/dev/null
```

结合 TopicReportList.MD 中的记录，建立完整待校准列表。

### 2. 逐篇校准

对每篇报道：

**a) 检查原始 URL 是否可访问**
从文件名还原原始 URL（`__` → `://`，`_` → `/`，去掉 `_offline.html` 后缀）
```bash
curl -sL -o /dev/null -w "%{http_code}" "<URL>"
```
- 200：可访问 ✅
- 404/其他：已失效 ⚠️（但我们有离线版，内容不丢）

**b) 用 Tavily Extract 重新提取**
如果 URL 可访问，用 Tavily Extract 获取最新正文（比旧 HTML 更干净）。
如果 URL 已失效，从本地旧 HTML 文件中提取正文（用 Bash 清理 HTML 标签）。

**c) 确认内容相关性**
检查正文是否真的提到了廖兰新或其职务。标记：
- ✅ 核心报道（廖兰新是主角或重点提及）
- ⚠️ 简单提及（只是在嘉宾列表或一句话带过）
- ❌ 不相关（误收录）

**d) 提取关键图片**
如果页面有演讲照片、产品图等重要图片，下载并用 vision_analyze 分析。

**e) 保存新格式 Markdown**
按 /archive skill 的格式保存到 `sources/articles/<slug>.md`

**f) 删除旧 HTML 文件**
转换成功后删除对应的 `*_offline.html` 文件

### 3. 生成校准报告

```
🔍 校准报告 [日期]

已校准：XX / 60
- ✅ 有效核心报道：XX 篇
- ⚠️ 简单提及：XX 篇
- ❌ 不相关/移除：XX 篇
- 🔗 链接已失效（但有离线存档）：XX 篇

新格式文件：sources/articles/ 下 XX 个 .md
已删除旧文件：XX 个 _offline.html

📊 校准后进度：XX / 100（XX%）
```

### 4. 更新 TopicReportList.MD

如果发现不相关的报道，从索引中移除。更新报道总数。

### 5. 更新 goals.md

校准后可能进度数字会变（去掉不相关的）。

### 6. Git 提交

```bash
cd ~/projects/github/my
git add -A
git commit -m "refactor: 校准报道库 - 转换为 Markdown 格式，校验 XX 篇"
git push origin main
```

## 分批执行

60 篇一次做完可能太久。建议分批：
- 每次校准 10-15 篇
- 按事件主题分批（先校准报道最多的主题）
- 每批完成后输出中间报告
