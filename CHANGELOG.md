# 变更记录 (Changelog)

本文件遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/) 格式，
版本号遵循 [Semantic Versioning](https://semver.org/lang/zh-CN/)。

## [Unreleased]

### Added

- `reports/外部对照_查新_2026-10-06.md`：广域时效性查新（Engram Adapter 细化、
  DeepSeek Engram、DMoE、Gavel、Knowing-Using Gap、TokenMem、bf16 精度不变性三篇）。
  定位校准：本仓库增量收紧为"注入决策的测量学"；README 先行者清单与测量方法论节
  同步补引

### Fixed

- E2a（SmolLM2 跨厂商）的口径更正：报告此前把"**10/20 条下降超过 0.3**"误写为
  "10/20 条 |d|>0.3"——按原始数据复算，|d|>0.3 实为 16/20（降 10、升 6）。
  数值本身无误，口径表述已更正，并在报告中补两份数据文件的口径说明
  （`_raw` 为脚本原始产出，`A_items` 字段是训练前 logp 而非漂移量；
  无后缀的摘要版为当时的 stdout 补录，两份并列保留）
- 补录摘要的误导性字段名 `A_items_abs_gt_03` 更名为 `A_items_drop_gt_03`，
  并补 `A_items_rise_gt_03` 与指向原始文件的引用

## [0.3.1] - 2026-10-05

### Added

- `experiments/` 补统一的 `OUT` 归档目录定义（指向仓库根的 `results/`）

### Fixed

- 实验脚本的产出路径统一到 `results/`：此前 16 个脚本写入 `experiments/`，
  其中 2 个用裸相对路径（会落到当前工作目录），与产物归档位置不一致
- 环境变量名去除历史品牌前缀，统一为 `PPB_MODEL_ID` / `PPB_MODEL_DIR` / `PPB_TAG`
- 根目录的游离数据文件移入 `results/`（`mem_continual_result_smollm2_raw.json`，
  为全精度原始版；`results/` 中原有同名文件是取整摘要版）
- README 中 `web_gate_fix.py` 的产出列改为标注真实状态（该文件未随仓库归档）

## [0.3.0] - 2026-10-02

- 阶段 2：文献对齐（R1 精读 / R2 双门对照 / R3 PKM 粒度 / R5 ET 形态对照）+ bf16 方法论成稿

## [0.2.0] - 2026-10-02

- 阶段 1：E1 多种子 / E2 跨模型 / E3 LoRA 基线 / E4 真实知识全链路 / E5 双技能干扰

## [0.1.0] - 2026-09-30

- 阶段 0：五件套闭环（内容门 / 锚定 / 路由 / 跨层技能 / 相变曲线 / bf16 三分解）

