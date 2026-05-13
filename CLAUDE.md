# 个人品牌监测项目

## 项目概述
廖兰新（滴滴能源 CTO）公开演讲与媒体报道的自动监测、归档和展示系统。

## 目标
找到 100 篇真实有效的报道链接。当前进度见 goals.md。

## 交互
- 使用中文回复。

## 项目结构
```
├── liao-lanxin-timeline.html    # 时间线展示页面（部署到 liaolan.xin）
├── goals.md                     # 目标与进度追踪
├── search-strategy.md           # 搜索策略（agent 自我维护）
├── search-log.jsonl             # 每日搜索日志
├── sources/
│   ├── TopicReportList.MD       # 已收录报道索引
│   ├── articles/                # Markdown 归档（新格式）
│   ├── images/                  # 关键图片
│   └── *_offline.html           # 旧格式（逐步淘汰）
└── .claude/
    ├── agents/media-monitor.md  # 媒体监测 agent
    └── skills/
        ├── search-daily/        # 每日搜索（只报告，不自动归档）
        ├── search-evolve/       # 搜索策略每日进化
        ├── archive/             # 归档（用户确认后执行）
        └── calibrate/           # 校准旧报道
```

## 归档格式
- 新报道统一用 Tavily Extract 提取正文，存为 Markdown + 关键图片
- 重要图片下载到本地 + vision_analyze 生成描述
- 旧的 _offline.html 文件通过 /calibrate 逐步转换

## 搜索工具优先级
- 优先使用 Tavily MCP 工具（mcp__tavily__tavily_search、mcp__tavily__tavily_extract）
- 补充使用 multi-search-engine（百度、微信搜狗、头条、360）— 通过 WebFetch 调用搜索引擎 URL
- 备选：WebSearch

## Git 工作流
- 远程仓库：GitHub
- 默认分支：main
- 提交规范：feat（新报道）、refactor（校准）、docs（策略更新）

## Agent
- 使用 media-monitor agent 处理搜索、归档、校准、策略进化
