# action-remix 插件

爆款桌拍视频拆解 × 动作标签素材库 × 组合成片引擎。

## 能力

| 命令 | 作用 |
|------|------|
| `/remix-analyze <关键词>` | 拉该商品热门视频 TOP10 → 切镜头 → 逐镜头动作标签 → 抄录文案按六段式归位 → 词表增量 |
| `/remix-compose <配方> [数量]` | 按配方从自拍素材库抽取组合 → 变异去重 → 自动渲染+质检 → 产出成片+剪映草稿 |

## 工作目录约定

- `action-remix/downloads/` 拆解用爆款视频
- `action-remix/analysis/` 拆解结果（镜头/标签/文案结构）
- `action-remix/library/` 你的自拍素材（按动作标签分目录）+ clips.jsonl
- `action-remix/recipes/` 配方
- `action-remix/output/` 成片（不进 git）

## 依赖

- 官方 **video-agent-kit** 插件（剪辑/质检/转录工具集）——请先安装
- Python: scenedetect（`pip install --no-deps "scenedetect>=0.6"` + opencv-python-headless）
- 剪映（可选，人工二次编辑草稿用）

## 铁律

成片素材 100% 来自自拍库；爆款视频只做结构分析。平台接口低频人工触发。每条产出带 seed 可追溯。
