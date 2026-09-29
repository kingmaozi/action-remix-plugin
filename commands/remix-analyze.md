---
description: 拆解商品爆款视频：下载 TOP10 → 切镜头 → 动作标签 → 文案结构 → 词表增量
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, mcp__plugin_video-agent-kit_video-edit__detect_shots, mcp__plugin_video-agent-kit_video-edit__video_read_frames, mcp__plugin_video-agent-kit_video-edit__video_watch_segment, mcp__plugin_video-agent-kit_video-edit__inspect_media
argument-hint: [商品关键词]（默认 纳宝帝玉米棒）
---

# 爆款拆解

对关键词「$ARGUMENTS」执行 action-remix 工作流 A（拆解）：

1. 按 skill 的获取步骤**单次**抓取搜索页 TOP10 并下载到 `action-remix/downloads/`（若 `downloads/` 已有本轮视频则跳过获取，直接拆解）
2. 逐视频 detect_shots → 镜头中点取帧打动作标签（词表见插件 `data/baseline-teardown.json`，新动作记增量）
3. 画面字幕逐句抄录，按六段式/三段式归位文案结构
4. 汇总写 `action-remix/analysis/拆解_<日期>.json`，并向我汇报：每条视频的镜头数、标签序列、文案六段式填充结果、词表新增了哪些动作

纪律：单次搜索不翻页；视频画面只用于分析，不进入任何成片。
