# 贡献指南 (Contributing)

感谢参与 PPBExt-Knowledge。本指南说明如何报告问题、提交改动与通过评审。

## 报告问题

- **Bug 报告**：使用 Issue 模板（含复现步骤、环境、预期与实际行为）
- **功能建议**：使用功能请求模板，说明使用场景与期望行为
- **安全漏洞**：**不要**开公开 Issue，见 [SECURITY.md](SECURITY.md)

## 提交改动

1. Fork 并创建分支：`git checkout -b fix/描述` 或 `feat/描述`
2. 遵循提交规范（Conventional Commits）：
   - `feat:` 新功能 · `fix:` 修复 · `docs:` 文档 · `refactor:` 重构
   - `perf:` 性能 · `test:` 测试 · `chore:` 杂项
3. 提交前自检：
   - [ ] 测试通过（bash experiments/phase1_runner.sh  # 或按 README 逐脚本运行）
   - [ ] 新功能附测试；数据/结论附可复现脚本
   - [ ] 文档同步更新（README 结果表 / CHANGELOG）
4. 发起 Pull Request，描述中说明：**改了什么、为什么、如何验证**

## 版本与发布

遵循 [Semantic Versioning](https://semver.org/lang/zh-CN/)；变更记录见 [CHANGELOG.md](CHANGELOG.md)
（[Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/) 格式）。

## 项目特有的评审标准

本项目为实证研究型仓库，评审侧重：

- **结论必须可复现**：新结论需附脚本与原始运行数据
- **表述必须带条件限定**：不做绝对化/首创性声明；未复现的不写"已验证"
- **数据必须完整披露**：指标口径、样本量、成本一并给出
- **AI 协作须声明**：使用 AI 辅助的内容遵循 [AI_DISCLOSURE.md](AI_DISCLOSURE.md) 的披露要求

## 行为准则

参与本项目即表示同意遵守 [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)。
