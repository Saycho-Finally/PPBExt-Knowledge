# PPBExt-Knowledge

**一句话**：把大语言模型当作一本**已经印刷定稿的书**——不重印、不改字，新知识以**补遗**的形式增补：内容门核证、行为锚定、路由隔离、技能模块跨层注入。全部流程在一个本地部署的 4B 模型上做了**初步验证**：**6GB 消费级显卡**、单实验 1.5~8 分钟、脚本与结果随仓库附带（本机验证可复现）。

> 作者：Saycho-Finally（独立研究者） ｜ AI 使用声明见 [AI_DISCLOSURE.md](AI_DISCLOSURE.md) ｜ License: MIT ｜ 依赖：见 requirements.txt ｜ Python ≥3.10

---

## 这个系统能做什么（四件事，均有实验数字）

1. **教模型新知识，不用重新训练基座**——16 条新事实 1 分钟学会（0%→100%），20 道旧问题在测试集上 0 退化，主干权重哈希前后一致（逐参数未触碰）。
2. **教它"手艺"，不只是"知识"**——一张从不在 prompt 出现的随机密码表，训练后模型能对**从未见过的数字**逐位编码（unseen exact 0→**1.000**）；该结果支持"学到的是程序而非死记"的解读，边界见「已知局限」。
3. **它按检索结果选择模块，未路由输入的输出与原模型逐位一致（实测）**——路由决策零错误；对照：全局激活会使旧任务漂移 -13.8（16/20 退化）。
4. **有看门的：过滤什么配被学会**——多源佐证/官方源/确定性校验三规则的内容门，17 条真实网络声明（含真实冲突与投毒）全部正确裁决；投毒与真知在无门时**同速固化**，准入判定必须在权重之外。

## 结果速览

**架构稳定容器**：trained vs random-init 残差流走向皮尔逊 **0.814**（36 层逐层，Qwen3-4B 实机）。

**不遗忘 + 相变曲线**（跨层联合分支，unseen exact / 逐位，随机水平 0.10）：

| 训练样本 | 16 | 64 | 128 | 256 |
|---|---|---|---|---|
| 密码任务 unseen exact | 0.000 | 0.375 | 0.938 | **1.000** |
| unseen 逐位 | 0.122 | 0.722 | 0.865 | **0.973** |

**漂移三分解**（全局激活，20 条旧任务）：执行噪声（bf16，数学恒等的模块也产生）**-2.17 / 80.2%**；真实语义效应仅 +0.53。→ 测量全局激活必须三分解，否则测的是噪声。

**对照（无锚定/无路由/无门）**：真事实训练致旧任务 -7.455（14/20）；投毒与真知同速固化；全局激活旧任务 -13.8（16/20）。

完整分析见 [reports/实验总结报告.md](reports/实验总结报告.md)。

## 这个仓库不是什么

先说边界，免得浪费你的时间，也免得我被打脸：

- **不是"解决了灾难性遗忘"**。验证的是：冻结基座 + 附加模块在本文条件下（Qwen3-4B、6GB、特定任务族），旧能力可以逐位不变、新知识可以注入。通用持续学习未解决，业界也未解决。
- **不是新 SOTA 方法**。全部组件思路来自公开研究（见「引用的先行者」），本仓库的增量是：全链路闭环验证、投毒门、bf16 执行噪声的三分解测量、记忆/程序相变曲线。
- **不是大规模评估**。单 GPU、任务为玩具级探针 + 少量真实核证事实；多数实验点单种子（相变曲线 64/128 两点已有 3 种子误差棒）；在密码探针上实跑过一次 LoRA 同预算对照（阶段一 E3），O-LoRA 等标准 CL 基线与标准基准协议仍在规划中。
- **不是可直接上生产的系统**。采集层（自动从交互流获取知识）未实现——知识写入仅在人工供给并核证后发生；全局激活存在 bf16 噪声地板；门是演示级规则门，不是完备的事实核查系统。
- **不是该方向的首个方案或综述**。2026 年该方向研究极度活跃（见下方引用清单新增条目：Brainstacks、Engram Adapter、Markov Matrix、DMoE、PaST 等），本仓库定位是**独立验证与边界测绘**：在消费级硬件上复核"冻结基座+附加模块"模式的关键性质，并记录测量陷阱。
- **不是"有长期维护承诺的产品"**。个人研究项目，业余时间维护——issue 与讨论的响应可能很慢甚至不响应；`results/` 内是 2026-09-30 的研究快照，不构成持续更新承诺。

反过来说，它**是**什么：一个每张表都能在 6GB 显卡上 10 分钟内复现的最小闭环，和一份把"哪条结论有多硬"写清楚的实验笔记。

## 使用流程（人机分工）

```
① 决定教什么          ← 人（方向与裁决，永远留给人）
② 供给知识源          ← 人指挥 agent 检索/整理（未来：采集层自动化）
③ 核证（过门）        ← curator.py 规则门 + 人最终确认
④ 训练（怎么教）      ← 机器：梯度下降写入模块（模块已挂在模型上，零初始化=不存在）
⑤ 验收（回归测试）    ← 机器跑旧任务回归 + 人看结果
```

关键性质：**被教的是模块，不是模型**。主干 36 层全程冻结（哈希作证）；教错了摘模块，模型无损。**架构不会自动获取知识源**——知识写入仅发生在人主动"教"时，推理过程纯只读；采集层自动化（交互流→门）是设计中的下一部件，且必须与自动回归同时上线。

---

## 环境要求

- NVIDIA GPU，显存 ≥ 6GB（4bit 量化）；仅推理，无需训练基座
- Python 3.10+，Windows/Linux 均可
- 磁盘 ≥ 10GB（模型权重 8.04GB）

### 安装

```bash
# ① PyTorch（注意：PyPI 默认给 CPU 版，4bit 量化必须 CUDA 版，从官方源装）
pip install torch==2.11.0+cu128 --index-url https://download.pytorch.org/whl/cu128

# ② 其余依赖
pip install transformers accelerate bitsandbytes safetensors tokenizers huggingface_hub numpy requests
```

> [注意] **已知的坑**（都踩过，写在这里省你半天）：
> - `pip install torch` 默认装 **CPU 版**（`+cpu`），bitsandbytes 4bit 无法工作，必须从 pytorch 官方源装 CUDA 版
> - 若装过与 torch 版本不匹配的 **torchvision**，会报 `operator torchvision::nms does not exist` 并连带炸掉 transformers——卸载 torchvision/torchaudio 即可
> - transformers 5.x 移除了 `quantize_model`；bitsandbytes 的 `Params4bit` 本身是 Parameter 子类，直接赋值不要再用 `nn.Parameter()` 包装
> - HF 下载走镜像时加 `HF_ENDPOINT=https://hf-mirror.com` 和 `HF_HUB_DISABLE_XET=1`（镜像无法代理 Xet 存储，否则 401）

### 下载模型（8.04GB，支持断点续传）

```bash
python experiments/download_model.py
# 等价于 snapshot_download('Qwen/Qwen3-4B-Instruct-2507') 到 experiments/model_cache/
# 注意仓库名必须带 -2507 后缀（不带后缀的旧 ID 在镜像上 404）
```

### 复现实验

```bash
# 全部脚本从仓库根目录运行；结果 JSON 输出到运行目录
python experiments/xinhua_exp.py            # ① 走向测试 (~1 min)
python experiments/mem_continual_test.py    # ② 不遗忘 (~2 min)
python experiments/curator.py               # ③ 规则门演示 (即时)
python experiments/auto_pipeline_test.py    # ④ 端到端 (~2 min)
python experiments/routing_test.py          # ⑤ 路由 (~2 min)
python experiments/skill_test2.py           # ⑥ 密码技能-顶层 (~3 min)
python experiments/skill_mid_test.py        # ⑦ 密码技能-中层 (~4 min)
python experiments/skill_multi_test.py      # ⑧ 跨层+数据量曲线 (~8 min/点)
#   数据量可用环境变量调节: N_TRAIN=256 STEPS=800 python experiments/skill_multi_test.py
python experiments/pegp_multilayer_test.py  # ⑨ PEGP 零空间投影 (~7 min)
python experiments/diag_pegp.py             # ⑩ PEGP 诊断工具 (~2 min)
python experiments/xinhua_exp_numpy.py      # ⑪ numpy 方法学模拟 (无 GPU 要求)
```

---

## 实验清单

| 脚本 | 验证内容 | 关键结果 | 结果文件 (results/) |
|---|---|---|---|
| `xinhua_exp.py` | 架构层走向是否由架构决定 | 走向相关 0.814；bf16 ULP 平台 | `xinhua_exp_result.json` |
| `xinhua_exp_numpy.py` | numpy 方法学模拟（多种子） | 多种子走向相关 0.97-0.98 | `xinhua_exp_numpy_result.json` |
| `mem_continual_test.py` | additive 注入不遗忘 + 哈希 | B 0→100%，A 0/20，哈希不变 | `mem_continual_result.json` |
| `web_gate_test.py` | 真事实也漂移 + 投毒同速 | A -7.455 (14/20)；投毒 0→1.00 | `web_gate_result.json` |
| `web_gate_fix.py` | replay 锚压漂移 | A 漂移 +0.212（35 倍改善） | （脚本产出，未随仓库归档） |
| `curator.py` | 规则门自动熟化 | 17 raw → 5 准入 5 拒绝，全对 | `curated_claims.json` |
| `auto_pipeline_test.py` | raw→门→训练 端到端 | B 0→1.00，A +0.201，1/20 | `auto_pipeline_result.json` |
| `routing_test.py` | 路由结构性隔离 | A 逐位为零 (bit-exact) | `routing_result.json` |
| `skill_test2.py` | 顶层=记忆 only | unseen 0.000 | `skill_result_v2.json` |
| `skill_mid_test.py` | 中层修复记忆不修复泛化 | seen 1.000 / unseen 0.000 | （stdout） |
| `skill_multi_test.py` | 跨层×数据量相变曲线 | 256 样本 unseen 1.000 | `skill_multi_result_*.json` |
| `pegp_multilayer_test.py` | PEGP 零空间投影 | **verdict=CHECK（未达标）**：锚投影按构造保持，但任务 A 均值漂移 **5.30**（6/20 条退化）；记录见 `pegp_result.json` | `pegp_result.json` |
| `pegp_decomp_test.py` | 漂移三分解 | 执行噪声占 80.2% — **该脚本未随仓库归档，此数字当前不可复现**，见 [reports/可复现性声明_2026-10-07.md](reports/可复现性声明_2026-10-07.md) | `pegp_decomp_result.json` |
| `diag_pegp.py` | 逐层诊断工具 | 定位执行噪声根因 | （stdout） |
| `pkm_vs_dense_test.py` | PKM 乘积键 vs 稠密分支 | **两种配置均未泛化**：seen 1.000 但 unseen_exact **0.000**（稠密分支 0.375）；A 退化 7/12 条 | `pkm_vs_dense_result{,_granular}.json` |

**清单口径**：上表只列"结果已在正文被引用"的脚本；仓内另有若干探索性脚本与运行器
（`phase1*.sh` / `skill_test2` 等）未逐一列出。**负结果与失败配置同样列出**
（PEGP 为 CHECK、PKM 两配置未泛化），不作为可选项省略。

---

## 测量方法论（重要）

全局激活实验的漂移测量**必须做三分解**：`直通（off）`、`挂零初始化分支（zero，数学恒等但执行上下文相同）`、`挂训练后分支（trained）`。
`zero − off` 是 bf16 执行噪声（分支矩阵乘改变 cuBLAS 上下文，大激活坐标 ~7000 处 ULP=32 产生 ±32 噪声），实测占表观漂移的 **80.2%**（**口径见下方声明**）；`trained − zero` 才是模块的真实语义效应。
不做三分解，会把机器抖动误判为模型学坏。路由（不执行分支矩阵乘）是唯一逐位精确的隔离方案。

**可复现性声明（2026-10-07）**：产生 80.2% 的脚本 `experiments/pegp_decomp_test.py`
**未随仓库归档**（`results/pegp_decomp_result.json` 在仓，但无对应脚本），
故该数字目前**不可从仓内脚本复现**。三分解的**方法**已在实验过程中使用并被后续
实验沿用（`results/pegp_decomp_result.json` 保留原始分解数据，可用 `diag_pegp.py`
的逐层口径部分复核）。重建脚本列入待办，重建前引用该数字请标注"脚本未归档"。

**外部实证（2026-10 查新）**：Greedy Decoding Is Not Precision-Invariant
（arXiv:2609.26621，TMLR）实测同 checkpoint 同提示下 BF16 与 FP16 的贪心生成
49%-100% 发散，且 **Qwen2.5-3B 在 GSM8K 上 19% 的提示最终答案翻转而聚合精度只动约
1 个点**（"漂移藏在基准分数里"）；arXiv:2510.04212 进一步证明低精度累加偏置是
**系统性而非随机**的。这两条把"必须逐位测量"从本仓库的经验观察升级为公开结论。

---

## 已知局限（诚实边界）

- 技能模块需要 O(100)+ 多样样本才诱导泛化电路；事实类只需 O(10)
- 全局激活存在 bf16 噪声地板（可统计正则压语义、无法消除执行噪声）；路由是结构解
- 只验证了事实注入与单步程序（密码编码）；多步推理类技能未测
- 词法路由阈值需对背景分布校准（见 routing 实验的 0.20 边界案例）
- 核心机制已在 4 个 ≤4B 模型实测（Qwen3-4B / Qwen2.5-3B / SmolLM2-1.7B / Phi-3-mini，两厂商三代）；更大规模与更深跨家族未测

---

## 引用的先行者（部分思路来自这些工作）

> 该方向 2026 年研究极度活跃；以下仅列与本仓库组件直接相关者，未穷尽。
> **定位校准（2026-10-06 查新）**："模块化知识注入进冻结基座"已是拥挤赛道
> （DMoE / TokenMem / GAG / HyperNetwork / Engram Adapter 等）。本仓库的增量
> 收紧为**注入决策的测量学**：样本量相变曲线、知识/程序边界、逐位纪律、
> 消费级硬件坐标。逐篇对照见
> [reports/外部对照_查新_2026-10-06.md](reports/外部对照_查新_2026-10-06.md)。

- Memory Layers at Scale (Meta FAIR, arXiv:2412.09764) — 模块容量形态
- MAC: Memory of Amortized Contexts — 冻结基座免梯度在线适应
- O-LoRA (EMNLP 2023 Findings, arXiv:2310.14152) / PEGP (arXiv:2405.13383) / GPM (ICLR 2021) — 正交零空间防遗忘
- SCALE (ACL 2026 Findings) — 冻结基座宽度扩展与可证保持
- Macaron-V1 (arXiv:2608.09819) — 冻结基座 + 专家路由的生产实践
- CL under Backdoor Attacks (arXiv:2609.06346) — 持续学习投毒防御
- GraftLLM (arXiv, SkillPack) — 模块化技能包 + 免遗忘持续学习 + 路由
- **Engram Adapter** (EMNLP 2026 Findings, arXiv:2608.29327) — 条件记忆适配器（同基座 Qwen3-4B **与 Qwen3-8B**；n-gram 选择先验 + 标量门，OOD 保持 99.4%-100.1%，残差衰减至隐藏态范数 ~0.08%，与本文路由组件同构）
- **DeepSeek Engram** (arXiv + 开源, 2026-01, Conditional Memory via Scalable Lookup) — 条件记忆作为 MoE 之外的新稀疏维度；U 型配比（20-25% 稀疏预算给记忆最优）、O(1) 哈希查找、主机内存预取开销 <3%（**预训练架构**路线，与本仓库的后置注入不同层）
- **Knowing-Using Gap** (arXiv:2607.08393) — "记住但不会用"的形式化与机制解释（知识回路错位）；对应本仓库"程序注入 seen 1.000 / unseen 0.000"的发现，引用时的标准出处
- **The Router Within (Gavel)** (arXiv:2609.15982) — 冻结 LLM 前向传播自带路由信号，两个线性映射即可读出（佐证判定器光谱的内部表示探针档）
- **TokenMem** — 冻结模型知识冲突的独立 softmax 通道注入（冲突遵从 69-70% 对 RAG 20-52%）
- **Rote Learning Considered Useful** (ICLR 2026) — "先机械记忆、后语义泛化"两阶段（与本文合成代号域设计同思路）
- **Memory as a Markov Matrix** (ICML 2026, arXiv:2605.04308) — token-to-dictionary 映射、可证零遗忘与样本复杂度界（与本文"字典+补遗"设定同构的理论版）
- **Brainstacks** (arXiv:2604.01152) — 冻结 MoE-LoRA 栈 + 零空间投影 + outcome 路由
- **DMoE** (arXiv:2606.14243) — 与基座解耦的专家做参数化知识注入（末层 FFN 挂载）
- **PaST** (arXiv:2601.11258) — 知识(SFT)与技能(RL)更新的近正交性与技能向量线性注入
- **JumpLoRA** (arXiv:2604.16171) — JumpReLU 稀疏参数隔离
- **Improving Sparse Memory Finetuning** (arXiv:2604.05248) — 消费级硬件上的稀疏记忆改造 + KL 槽位选择
- **Mechanistic Analysis of Catastrophic Forgetting** (arXiv:2601.18699) — 20 个 LLM 的遗忘机制分析与分层定位

---

## English (Quick Summary)

A full-pipeline experimental validation of **modular continual learning on a frozen 4B LLM**, on consumer hardware (6GB GPU): content gate (multi-source verification, poison defense) → behavioral anchoring (replay KL, 35× drift reduction) → lexical routing (bit-exact isolation, ±0 drift on unrouted inputs) → skill modules (multi-layer joint branches; phase transition from memorization to **100% generalization** at 256 examples) → measurement methodology (3-way decomposition showing **80% of naive drift is bf16 execution noise**). All experiments reproduce in minutes; see the tables above and [reports/实验总结报告.md](reports/实验总结报告.md).

## License

MIT — 见 [LICENSE](LICENSE)。基座模型 Qwen3-4B-Instruct-2507 © Alibaba Cloud, Apache 2.0。


---

## 贡献与引用

- 贡献指南见 [CONTRIBUTING.md](CONTRIBUTING.md)；行为准则见 [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)
- 安全问题请走 [SECURITY.md](SECURITY.md) 的私密渠道（勿开公开 Issue）
- 版本变更见 [CHANGELOG.md](CHANGELOG.md)；学术引用格式见 [CITATION.cff](CITATION.cff)
- 许可：MIT
