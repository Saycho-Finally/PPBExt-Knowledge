# 变更记录 (Changelog)

本文件遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/) 格式，
版本号遵循 [Semantic Versioning](https://semver.org/lang/zh-CN/)。

## [Unreleased]

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

