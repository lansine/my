---
name: search-daily
description: 每日媒体报道搜索。全量搜索廖兰新的公开报道（不限时间），对比已有记录去重，发现新内容后报告给用户确认，用户确认后再归档。用法：/search-daily
---

# 每日媒体报道搜索

## 核心原则
- **只搜索、只报告，不自动归档** — 发现新报道后列出来给用户确认
- **全量搜索** — 不限时间范围，新报道和历史遗漏都要找
- 目标：找到 100 篇真实有效的报道链接（当前进度见 goals.md）

## 当前阶段（2026-05-15 起）

第 1-2 层（人名直搜 / 人名+公司）连续 5 天饱和，新增极少。**切换搜索重心**到下面三个专项方向，第 1-2 层降为兜底：

### 🎯 专项 A：P3 高产域名定向搜索
对历史命中 ≥ 2 的域名做 `site:` 精准搜索，覆盖搜索引擎索引未及的页面：
- `site:finance.people.com.cn "廖兰新"`
- `site:chd.in-en.com "廖兰新"`
- `site:finance.sina.com.cn "廖兰新"`
- `site:tech.cnr.cn "廖兰新"`
- `site:ifeng.com "廖兰新"`
- `site:www.eeo.com.cn "廖兰新"`
- `site:www.inengyuan.com "廖兰新"`
- `site:www.yilantop.com "廖兰新"`
- `site:www.pai.com.cn "廖兰新"`
- `site:chejiahao.autohome.com.cn "廖兰新"`

### 🎯 专项 B：品牌升级（2025.11.25）补挖
当前仅 7 篇，业内预期 20–30 篇，目标补 10+ 篇：
- 核心词：`"滴滴充电" "廖兰新"`，加时间筛选 `tbs=qdr:y`（近一年）
- 配套：`"小桔充电" "更名" "滴滴充电"`、`"滴滴充电" 品牌升级 廖兰新`

### 🎯 专项 C：2025 Q1–Q2 空白填补
2025 年最早事件是 7 月，前半年完全空白：
- `"廖兰新" 2025 中电联`（中电联大会通常 3-4 月）
- `"廖兰新" 2025 充换电桩展`
- `"廖兰新" 2025 年度总结`
- `"廖兰新" 滴滴充电 2025年初`

## 操作流程

### 1. 加载策略和现状

读取 `search-strategy.md` 获取当前搜索策略。
读取 `goals.md` 获取当前进度。
读取 `sources/TopicReportList.MD` 提取所有已收录 URL（用于去重）。

### 2. 执行搜索

**当前优先顺序**（2026-05-15 起）：
1. 先跑 **专项 A / B / C**（按当日轮换，至少覆盖一个专项的全部关键词）
2. 兜底再跑第 1-2 层默认关键词（每周 1-2 次即可，无需每天）

**搜索工具矩阵**：

| 工具 | 用途 | 调用方式 | 角色 |
|------|------|----------|------|
| Tavily Search | 主力全网搜索 | mcp__tavily__tavily_search | **主力** |
| Tavily Extract | 正文提取 | mcp__tavily__tavily_extract | 主力 |
| multi-search-engine | 国内引擎 + 站内搜索 + 高级语法 | `/multi-search-engine` skill 或 WebFetch 调用搜索引擎 URL | **补充**（17 个引擎，无 API） |
| weixin-search | 微信公众号 + 腾讯体系 | `/weixin-search 廖兰新` | 关键补充 |
| WebSearch / WebFetch | 兜底 | 直接调用 | 备选 |

**multi-search-engine 用法要点**：
- 17 个搜索引擎（8 国内 + 9 国际），无需 API 密钥
- 优势：支持 `site:`、`tbs=qdr:y` 等高级语法；覆盖百度/搜狗/360/头条等 Tavily 不强的国内引擎
- 调用：通过 WebFetch 直接拉搜索结果页，或用 `/multi-search-engine` skill 包装
- 适用场景：
  - 专项 A 的 `site:` 精准搜索（百度/搜狗对中文站点索引比 Tavily 全）
  - 专项 B 的时间筛选（`tbs=qdr:y` 近一年过滤）
  - 微信、头条号等 Tavily 触达不到的内容

**默认关键词（兜底用，不再每天都搜）**：

第一层（人名直搜）：`"廖兰新"`

第二层（人名+公司）：
- `"廖兰新 滴滴"` / `"廖兰新 滴滴充电"` / `"廖兰新 滴滴能源"` / `"廖兰新 小桔充电"` / `"廖兰新 小桔能源"`

第三层（职位）：`"滴滴能源 CTO"` / `"滴滴充电 CTO"` / `"小桔充电 CTO"`

第四层（产品/事件，轮换）：`Orange Unicorn 充电桩` / `滴滴 车网互动` / `滴滴 超充` / `小桔充电 智能运维`

第五层（行业大会，轮换）：`中国汽车充换电生态大会 滴滴` / `中国汽车论坛 滴滴充电`

**搜索节奏**：
- 不着急，控制 API 调用频率
- 每天聚焦 1 个专项（A/B/C 轮换），第 1-2 层每周兜底 1-2 次
- 宁可慢一点，不要频繁调用接口

**每个关键词都要记录效果**：搜到几条、新发现几条、来自什么来源。

### 3. 去重

- URL 精确匹配已有记录 → 排除
- 内容明显不相关 → 排除
- 同一篇文章不同 URL（转载） → 都算有效，都保留

### 4. 写搜索日志

追加一行到 `search-log.jsonl`：
```json
{
  "date": "2026-05-15",
  "search_angle": "专项A高产域名 / 专项B品牌升级 / 专项C 2025 Q1-Q2 / 兜底默认词",
  "tools_used": ["tavily_search", "multi-search-engine:baidu", "weixin-search"],
  "keyword_results": {
    "site:finance.people.com.cn 廖兰新": {"total": 5, "new": 1, "duplicates": 4}
  },
  "source_hits": {"chd.in-en.com": 2, "ifeng.com": 1},
  "new_reports": [
    {"url": "...", "source": "来源名", "title": "标题", "keyword_found_by": "...", "tool": "tavily/multi-search-engine/weixin"}
  ],
  "strategy_version": "search-strategy.md 的上次更新日期",
  "notes": "观察记录"
}
```

### 5. 输出报告

**有新发现：**
```
📰 报道监测日报 [日期]
今日搜索角度：[专项A/B/C 或 兜底]
使用工具：Tavily / multi-search-engine（百度/搜狗/...）/ weixin-search

发现 X 篇新报道，请确认是否归档：

1. [来源] 标题
   🔗 URL

2. ...

回复"确认"我来归档，或告诉我哪些不要。

📊 进度：XX / 100（XX%）
```

**无新发现：**
```
📰 报道监测日报 [日期]
今日搜索角度：[xxx]
使用工具：[xxx]
未发现新报道。
📊 进度：XX / 100（XX%）
```
