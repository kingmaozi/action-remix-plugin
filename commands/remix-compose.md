---
description: 按配方组合自拍素材批量成片（原创）：抽取→变异→timeline→渲染→QC
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, mcp__plugin_video-agent-kit_video-edit__validate_timeline, mcp__plugin_video-agent-kit_video-edit__render_preview, mcp__plugin_video-agent-kit_video-edit__qc_preview, mcp__plugin_video-agent-kit_video-edit__timeline_diff, mcp__plugin_video-agent-kit_video-edit__speech_synthesize
argument-hint: [配方名或商品名] [数量，默认10]
---

# 组合成片

对「$ARGUMENTS」执行 action-remix 工作流 B（组合生成）：

1. 读取 `action-remix/recipes/` 对应配方（无配方则提示先跑 /remix-analyze），检查 `library/clips.jsonl` 各槽位标签素材是否 ≥3 条，不足则列缺口清单停止
2. 按数量 N 抽取素材（seed 记录）+ 变异（入出点抖动/变速/镜像）+ LLM 按六段式改写文案
3. 组 timeline JSON → validate_timeline → render_preview → qc_preview；QC 不过走 timeline_diff 修复
4. 交付 `action-remix/output/<gen_id>/`：成片 + 剪映草稿（pyJianYingDraft）+ 素材清单 + seed，向我汇报每条的标签序列与 QC 结果

铁律：素材必须全部来自 library/（自拍）；每条带 variant_seed 可追溯；发布由我人工确认。
