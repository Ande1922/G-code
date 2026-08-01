# G-code 总体迭代规划

## 文档职责

本文维护 G-code 的**可演进路线**：上游章节如何映射到本项目、各迭代新增什么能力、明确不做
什么，以及如何验收。稳定的项目目标、单次迭代步骤、安全规则和完成标准仍由根目录
[`AGENTS.md`](../AGENTS.md) 规定；每次迭代的实际讲解、实现证据和结果写入
`docs/iterations/iteration-XX.md`。

规划状态含义：

- **前置核验**：只建立可追溯事实，不计作功能实现；
- **待开始**：边界已定义，但尚无完成证据；
- **进行中**：只能有一个迭代处于此状态；
- **已完成**：对应迭代文档满足 `AGENTS.md` 的全部完成标准。

## 已核验的上游基线

- 权威仓库：[`bojieli/ai-agent-book`](https://github.com/bojieli/ai-agent-book)；默认分支为
  `main`。
- 规划基线：固定提交
  [`fc7c166cfdcc32dbd042213a0186056442f9efe0`](https://github.com/bojieli/ai-agent-book/commit/fc7c166cfdcc32dbd042213a0186056442f9efe0)，
  而不是易变的分支链接。
- 原始中文正文位于
  [`book/`](https://github.com/bojieli/ai-agent-book/tree/fc7c166cfdcc32dbd042213a0186056442f9efe0/book)，
  配套实验位于 `chapter1/` 至 `chapter10/`，公共提供商代码位于 `agentbook/providers/`。
- 根许可证为
  [`Apache-2.0`](https://github.com/bojieli/ai-agent-book/blob/fc7c166cfdcc32dbd042213a0186056442f9efe0/LICENSE)。
  规划只借鉴结构和技术路线；若以后引用具体实现，仍须核对目标目录内的许可证。

核验过程、目录证据、提交时间和许可证边界详见
[`iterations/iteration-00.md`](iterations/iteration-00.md)。

## 上游实际路线与本项目拆分原则

固定提交的上游 README 将全书组织为十章：

1. Agent 基础知识；
2. 上下文工程；
3. 用户记忆和知识库；
4. 工具；
5. Coding Agent 与代码生成；
6. Agent 的评估；
7. 模型后训练；
8. Agent 的持续进化；
9. 多模态与实时交互；
10. 多 Agent 协作。

上游按知识主题组织，并在第 5 章给出功能较完整的 Coding Agent。G-code **遵循上游的概念和依赖
关系，但不机械照抄章节顺序**：书可以在一章中解释完整系统，教学代码则要按可观察、可测试的
运行时能力重新排序，每次只增加一个闭环。第 1、2、4、5 章构成基础 Coding Agent 主线；第
3、6、8、10 章分别在基础闭环之后引入记忆、评估、持续改进和多 Agent。第 7 章的模型训练与第
9 章的语音、GUI、机器人不是最小 Coding Agent 的前置依赖，当前只保留扩展接口，不纳入
Iteration 0–11 的实现范围。

沙箱是这条主线的必要组成，而不是“受控命令”的一个实现细节。上游第 4 章明确区分 venv 与真正
的沙箱，并给出 OS 级隔离、容器、microVM 和资源配额的层级；第 5 章进一步要求默认断网、最小
文件挂载、资源/超时限制，以及让持久终端的生命周期不超过沙箱。因此本项目在命令执行前安排一个
独立的沙箱迭代：先用固定探针验证隔离边界，再允许模型提出的命令进入该边界。这样既跟随上游的
安全路线，也避免在没有隔离兜底时先暴露任意代码执行能力。

## 迭代路线

| 迭代 | 状态 | 单一可验证增量 | 主要上游依据 | 明确边界与验收重点 |
| --- | --- | --- | --- | --- |
| 0 | 前置核验 | 项目骨架、同步模型接口与可替换假模型 | [第 1 章](https://github.com/bojieli/ai-agent-book/blob/fc7c166cfdcc32dbd042213a0186056442f9efe0/book/chapter1.md)的 LLM、上下文与 Harness；[第 5 章](https://github.com/bojieli/ai-agent-book/blob/fc7c166cfdcc32dbd042213a0186056442f9efe0/book/chapter5.md)的 Coding Agent 架构 | 先完成上游核验；实现阶段只验证模型请求/响应契约及确定性假模型，不做 CLI、历史或工具。 |
| 1 | 待开始 | 单轮命令行问答 | 第 1 章的 LLM 与 Harness | 一次输入只产生一次模型调用；覆盖正常响应、模型失败和退出码，不保留历史。 |
| 2 | 待开始 | 多轮消息历史与上下文管理 | [第 2 章](https://github.com/bojieli/ai-agent-book/blob/fc7c166cfdcc32dbd042213a0186056442f9efe0/book/chapter2.md) | 明确定义 system/user/assistant 消息、顺序与上下文预算；暂不调用工具。 |
| 3 | 待开始 | 单工具调用与最小 Agent Loop | [第 4 章](https://github.com/bojieli/ai-agent-book/blob/fc7c166cfdcc32dbd042213a0186056442f9efe0/book/chapter4.md)和第 1 章 ReAct 循环 | 只接入一个无副作用工具；区分模型意图、应用校验和真实工具结果，并设置最大循环次数。 |
| 4 | 待开始 | 受限读取、目录遍历与代码搜索 | 第 4 章感知工具、第 5 章搜索工具 | 所有路径限制在工作区；测试 `..`、绝对路径和符号链接越界；不写文件。 |
| 5 | 待开始 | 受控文件编辑、diff 与人工确认 | 第 4 章执行工具、第 5 章文件编辑和安全 | 写入前校验并展示 diff，只有真实用户确认才可执行；拒绝越界、未确认覆盖和模型伪造确认。 |
| 6 | 待开始 | 沙箱执行环境与隔离探针 | [第 4 章沙箱分层](https://github.com/bojieli/ai-agent-book/blob/fc7c166cfdcc32dbd042213a0186056442f9efe0/book/chapter4.md#L251-L260)、[第 5 章沙箱选型](https://github.com/bojieli/ai-agent-book/blob/fc7c166cfdcc32dbd042213a0186056442f9efe0/book/chapter5.md#L112-L120) | 定义可替换沙箱接口并选择当前环境可验证的隔离后端；固定探针必须证明宿主路径不可见、网络默认关闭、进程/CPU/内存/磁盘受限且超时可终止。此阶段不接受模型生成的任意命令。 |
| 7 | 待开始 | 沙箱内受控命令执行 | 第 4 章执行工具、第 5 章 Coding Agent 安全与错误恢复 | 所有命令只能进入 Iteration 6 的沙箱；限制工作目录、参数、环境、时长和输出，完整返回 stdout、stderr、退出码及超时/资源终止原因；测试沙箱不可用时默认拒绝而非回退宿主机。 |
| 8 | 待开始 | 任务计划、步骤状态、重试与终止 | 第 5 章整体流程和错误恢复、第 8 章轨迹反馈 | 状态转换可审计；限制重试和总步数；失败不得被模型成功声明覆盖。 |
| 9 | 待开始 | 记忆、摘要与上下文窗口治理 | [第 3 章](https://github.com/bojieli/ai-agent-book/blob/fc7c166cfdcc32dbd042213a0186056442f9efe0/book/chapter3.md)和第 2 章上下文压缩 | 先做短期摘要，再引入显式长期记忆；测试来源、淘汰、污染和隐私边界。 |
| 10 | 待开始 | 多 Agent 委派与结果汇总 | [第 10 章](https://github.com/bojieli/ai-agent-book/blob/fc7c166cfdcc32dbd042213a0186056442f9efe0/book/chapter10.md) | 子任务有清晰输入、权限和预算；每个子 Agent 继承不超过主 Agent 的沙箱权限；汇总保留来源、冲突和失败，不共享未授权上下文。 |
| 11 | 待开始 | 评估集、轨迹、可观测性与安全加固 | [第 6 章](https://github.com/bojieli/ai-agent-book/blob/fc7c166cfdcc32dbd042213a0186056442f9efe0/book/chapter6.md)和第 8 章持续进化 | 固定基准、指标和轨迹模式；覆盖提示注入、工具拒绝、沙箱逃逸探针和事实来源冲突；不自动修改模型参数。 |

Iteration 0 的“前置核验”状态表示仓库基线已经确认，但骨架、模型接口和假模型尚未实现；只有对应
文档补齐原理、实现、测试、代码反解、统一示例与总结后，才能改为“已完成”。

## 贯穿各迭代的统一基准

固定任务不随实现变化：

> 阅读工作区中的一个小型示例项目，找到指定行为的实现位置，解释其工作方式，进行一处受控修改，
> 并运行相关验证。

各阶段只扩展当前能力允许的部分：Iteration 0–2 只能解释输入与模型交互；Iteration 3 能观察
工具闭环；Iteration 4 能实际定位实现；Iteration 5 能经确认修改；Iteration 6 只能用固定探针
证明沙箱边界；Iteration 7 才能在沙箱内运行验证；Iteration 8–11 依次增加可恢复执行、上下文
治理、委派和可量化评估。每篇迭代文档必须记录固定输入、预期能力、真实输出、尚未完成的部分和
相对上一阶段的差异。

## 规划维护规则

1. 开始新迭代前重新核对相关上游文件；若提交变化，先在当次迭代文档记录新提交和差异，再决定
   是否更新本规划基线。
2. 路线、顺序、范围或状态变化必须与实现变更分开说明，并记录原因、依赖、学习影响和验证变化。
3. “已完成”只由实际证据决定，不能根据模型声明、计划文本或代码存在性推断。
4. 不在本文复制实现细节；不在 `AGENTS.md` 重复本表。稳定流程归 `AGENTS.md`，动态路线归本文，
   单次事实与结果归 `docs/iterations/`。
