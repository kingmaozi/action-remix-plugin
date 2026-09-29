---
name: action-remix
description: 爆款桌拍视频拆解（切镜头→动作标签→文案结构）与自拍素材组合成片的完整工作流。当用户要求"拆解视频/打标签/建素材库/组合生成带货视频/批量出片"时使用。
---

# action-remix — 爆款拆解 × 动作标签 × 组合成片

把爆款桌拍视频拆成「动作标签序列 + 文案结构」，用户按词表自拍素材入库，再按配方组合成**原创**带货视频。数据基线见 `data/baseline-teardown.json`（2026-09-27 拆解 10 条玉米棒爆款：105 镜头、词表 v1 33 标签、文案六段式模板）。

## 铁律（每次执行前确认）

1. **原创红线**：成片素材只能来自 `library/`（自拍/AI 生成）。爆款视频只产出结构化分析（标签/文案/时长），**任何画面帧不得进入成片**
2. **接口低频**：拉热门视频单次搜索、人工触发；禁止自动翻页/循环抓取（kngx 502 教训）
3. 每条产出带 `variant_seed` 与素材清单，可追溯可重生成
4. 视频剪辑操作优先调用官方 **video-agent-kit** 插件工具（detect_shots / video_read_frames / video_watch_segment / validate_timeline / render_preview / qc_preview / speech_transcribe / speech_synthesize）

## 工作流 A：拆解（/remix-analyze）

输入：商品关键词（如"纳宝帝玉米棒"）或已下载的视频路径。

1. **获取**（无视频时）：单次抓取搜索页（复用 `video/cat/scripts/capture.mjs`，30s 无交互），从 search/item 响应按 `赞+分享×3+评论×2` 排序取 TOP10，用 play_addr 下载（CDN 直链带 Referer，间隔 2s）。存 `downloads/vXX_<aweme_id>.mp4`
2. **切镜头**：逐视频 `detect_shots`（threshold 27, min_len 0.8）→ `analysis/vXX_shots.json`。**n_shots≤2 时用 threshold 15 重切**；仍一镜到底则标记 `one_take`，改用语义分段
3. **打标签**：`video_read_frames` 按镜头中点取帧（≤8 帧/视频，长视频抽样），逐镜头标注：
   - 动作标签（从词表 v1 选，见下）＋ 新动作记入增量词表
   - 机位（俯拍固定/特写推进/手持/半身固定）、时长
   - 画面字幕逐句抄录（= 口播文案，勿用 ASR 也可）
4. **文案结构**：按六段式归位（钩子→数量价值→外观卖点→结构卖点→口感→CTA），真人口播型用三段式（价格对比→试吃→囤货）
5. **产出**：`analysis/拆解_<日期>.json`（schema 同 data/baseline-teardown.json）＋ 词表增量。一镜到底视频做**动作语义切分**（物理 1 镜头可能含 3 个动作段）

## 动作词表 v1（33 标签，执行时以 `data/baseline-teardown.json` 为准并允许增量）

- 手部动作_桌拍：拿起、放下、排列摆放、手持展示、掰断、掰开露夹心、按压、撕袋、开盒、倒出、堆叠、竖举对比、指认细节、转动展示、握把展示
- 场景陈列：整盒展示、箱内满盒陈列、托盘陈列、包装堆、多根并列、截面对镜头
- 人物动作：口播介绍、试吃、咬断、试吃反应、比大小、宠物互动
- 镜头语言：俯拍固定、特写推进、手持跟拍、半身固定

## 工作流 B：组合生成（/remix-compose）

输入：配方（Recipe JSON）+ 素材库 `library/`（clips.jsonl 索引）+ 数量 N。

1. **配方**：从拆解结果生成 `recipes/<商品>.json`：`slots[]`（每槽 action_tag+时长+衔接约束）、文案六段式模板、风格（BGM/LUT/9:16）。衔接约束：`放下`后不得接`放下`；`掰断`后必须接`断面展示`
2. **抽取**：每槽从对应标签素材池抽 1 条（同 seed 可复现；7 日内素材不重复；入出点 ±0.3s 抖动）
3. **变异**：0.9-1.1 变速 / 镜像 / 转场池（硬切为主） / 文案逐句 LLM 改写（保结构换措辞）
4. **成片**：组 timeline JSON → `validate_timeline` → `render_preview` → `qc_preview`（黑帧/静音/冻结检查）→ 不合格走 `timeline_diff` 修复。并行产出 pyJianYingDraft 剪映草稿供人工微调
5. **交付**：`output/<gen_id>/`（成片 mp4 + 剪映草稿 + 素材清单 + seed）。发布必须人工确认

## 素材入库（拍摄后）

自拍素材按 `library/<动作标签>/g<组号>_l<光线号>.mp4` 存放；入库时跑同款拆解（detect_shots + 帧校验）自动写 `clips.jsonl`（clip_id/action_tag/duration/angle/lighting/pHash/quality）；pHash 查重拒绝重复片段。

## 验收口径

- 拆解：给定关键词 → TOP10 视频全部产出标签序列+文案结构，每条镜头级标注，全程 ≤3 次平台请求
- 组合：1 配方 + 每标签 ≥10 条素材 → 一键 10 条互不重复、QC 全过、剪映可打开的成片
