# Iteration 0：项目骨架、模型接口与可替换假模型

> 总体范围、上下游映射和当前状态见 [`../iteration-plan.md`](../iteration-plan.md)；本文只记录
> Iteration 0 的原理、实现与验证证据。

## 上游参考核验（实现前置记录）

> 核验时间：2026-08-01（UTC）<br>
> 核验状态：已完成；项目维护者已确认以下仓库为权威上游。<br>
> 本节记录的是网络与 Git 实际查询结果，不是模型对仓库名称的推测。

### 权威来源与固定版本

- 权威仓库：[`bojieli/ai-agent-book`](https://github.com/bojieli/ai-agent-book)。GitHub
  Repository API 返回的 `full_name` 为 `bojieli/ai-agent-book`，该仓库不是 Fork、未归档；仓库
  README 也将其称为全书正文与配套实验的开源仓库。
- 默认分支：`main`。该结论由 GitHub Repository API 的 `default_branch` 字段，以及克隆后
  `refs/remotes/origin/HEAD -> refs/remotes/origin/main` 两项结果交叉核对。
- 本次核验提交：
  [`fc7c166cfdcc32dbd042213a0186056442f9efe0`](https://github.com/bojieli/ai-agent-book/commit/fc7c166cfdcc32dbd042213a0186056442f9efe0)
  （提交时间 `2026-08-01T12:21:14+08:00`，标题
  `docs(i18n): sync figure 2-3 clarification (#573)`）。以下文件、目录和章节链接均固定到该提交，
  不以易变的 `main` 分支内容作为本次迭代依据。

检索时确实发现多个同名仓库；因此没有按热度或描述自行选择。在项目维护者明确回复确认后，才将
`bojieli/ai-agent-book` 作为上游继续核验。

### 实际目录与章节结构

固定提交的[根目录](https://github.com/bojieli/ai-agent-book/tree/fc7c166cfdcc32dbd042213a0186056442f9efe0)
包含以下与本项目路线直接相关的结构：

- 中文正文位于 [`book/`](https://github.com/bojieli/ai-agent-book/tree/fc7c166cfdcc32dbd042213a0186056442f9efe0/book)：
  `introduction.md`、`chapter1.md` 至 `chapter10.md`、`afterword.md` 和参考答案；另有多个
  `book-<locale>/` 翻译目录。
- 配套代码按 `chapter1/` 至 `chapter10/` 分章存放，各章有 README 和若干独立实验目录；
  公共模型提供商代码位于 `agentbook/providers/`，依赖与章节 extras 定义于 `pyproject.toml`，
  锁定依赖位于 `uv.lock`。
- 上游 README 的[内容速览](https://github.com/bojieli/ai-agent-book/blob/fc7c166cfdcc32dbd042213a0186056442f9efe0/README.md#L51-L68)
  列出的技术路线依次是：Agent 基础、上下文工程、用户记忆和知识库、工具、Coding Agent 与
  代码生成、评估、模型后训练、持续进化、多模态与实时交互、多 Agent 协作。

Iteration 0 最直接参考以下材料：

1. [第 1 章正文](https://github.com/bojieli/ai-agent-book/blob/fc7c166cfdcc32dbd042213a0186056442f9efe0/book/chapter1.md)
   的“现代 Agent = LLM + 上下文 + 工具”、LLM、上下文、ReAct 循环和 Harness 工程部分，
   用于界定模型接口在 Agent 中的位置；对应的
   [第 1 章实验目录](https://github.com/bojieli/ai-agent-book/tree/fc7c166cfdcc32dbd042213a0186056442f9efe0/chapter1)
   用于核对正文与代码分离的组织方式。
2. [第 5 章正文](https://github.com/bojieli/ai-agent-book/blob/fc7c166cfdcc32dbd042213a0186056442f9efe0/book/chapter5.md)
   的 Coding Agent 整体流程、Harness 工程、错误恢复、搜索和文件编辑部分，以及其
   [Coding Agent 配套项目](https://github.com/bojieli/ai-agent-book/tree/fc7c166cfdcc32dbd042213a0186056442f9efe0/chapter5/coding-agent)，
   用于确认后续能力的目标形态。

映射关系并非照搬上游章节：上游第 1 章一次介绍完整 Agent 构成，第 5 章展示综合 Coding
Agent；本项目为便于观察和验证底层机制，把这条路线拆成更小的 Iteration 0–11。Iteration 0
只建立项目骨架、模型接口和可替换假模型，不提前引入上游综合示例中的工具集、Agent Loop、
文件修改或命令执行能力。

### 许可证核验与采用边界

- 根许可证文件为固定提交中的 [`LICENSE`](https://github.com/bojieli/ai-agent-book/blob/fc7c166cfdcc32dbd042213a0186056442f9efe0/LICENSE)，
  GitHub Repository API 同时识别为 `Apache-2.0`；文件末尾版权声明为 `Copyright 2025
  Bojie Li`。
- Apache License 2.0 允许在满足许可证、修改声明及归属保留等条件下使用、修改和分发，
  但不授予商标权且不提供担保。本项目当前只记录结构与技术路线，不复制上游代码或正文。
- 上游部分实验目录另有自己的许可证文件；未来若引用或移植具体实验，必须在当次迭代重新检查
  目标文件及其目录内更具体的许可证，不能仅凭根许可证推定所有嵌套第三方内容均可复制。

### 核验方法与可复查命令

本次通过 GitHub Repository API 检查仓库元数据，通过 HTTPS 浅克隆读取 Git 引用与实际文件树，
并运行以下关键命令交叉复核：

```bash
git clone --depth=1 https://github.com/bojieli/ai-agent-book.git /tmp/ai-agent-book-verify
git -C /tmp/ai-agent-book-verify symbolic-ref refs/remotes/origin/HEAD
git -C /tmp/ai-agent-book-verify show -s --format='%H%n%cI%n%s' HEAD
find /tmp/ai-agent-book-verify -maxdepth 2 -type d -not -path '*/.git*' -print
find /tmp/ai-agent-book-verify -maxdepth 4 -type f \
  \( -iname 'LICENSE*' -o -iname 'COPYING*' -o -iname 'NOTICE*' \) -print
rg -n '^#{1,5} ' book/chapter1.md book/chapter5.md \
  chapter1/README.md chapter5/README.md chapter5/coding-agent/README.md
```

本记录只完成“实现前上游核验”这一前置增量；Iteration 0 的原理讲解、最小实现、验证、代码反解、
统一示例和总结将在后续实现增量中补齐，不能以本节代替完成标准。
