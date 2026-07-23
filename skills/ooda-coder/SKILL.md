---
name: ooda-coder
description: "通过 Scout、Forger、Act、Eval 四阶段 OODA-E 流水线，在已有文档的项目中实现业务需求。适用于需要独立完成上下文发现、实现规划、代码修改和验收验证的开发任务。"
---

# OODA Coder

严格按 Scout、Forger、Act、Eval 顺序调度四个上下文隔离的子代理，不得跳过阶段。

## 目标

交付遵循项目约定、复用既有基础设施、经过独立验证且有简明审计记录的代码变更。

## 完成标准

- 变更遵循适用约定和既有 API。
- 构建及相关测试通过。
- Eval 独立检查实际变更。
- 本次会话产出四份简短交接文档和一份执行日志。

## 共享准则

- 所有交接文档中的路径均使用项目根目录相对路径。
- 源码代表当前行为；发现文档冲突时仅报告，不修改文档。
- 不猜测不明确的业务行为，不擅自扩大范围；阻塞时记录待确认项并停止。
- 不修改计划外文件，保留用户的无关变更。
- 报告只包含结果、证据、阻塞项和下一步；交接文档限一页。

## 会话文件

创建 `.ooda-work/{UTC-YYYYMMDDHHmmss}/`，其中包含：

- `context-brief.md`
- `code-plan.md`
- `execution-report.md`
- `verify-result.md`
- `execution-log.md`

完成后将目录移至 `INDEX.md` 配置的 AI 执行日志路径；如未配置，则保留在 `.ooda-work/` 并告知用户。

## 流水线

### 1. Scout：观察与定位

调度 `subagents/scout.md`，传入用户原始需求和 Context Brief 输出路径。

Scout 负责剪枝：优先由 `INDEX.md` 导航，一次性读取任务相关 README 和界定范围所需的源码，写入精简业务结论、涉及路径、风险及供 Forger 阅读的文档路径。Scout 不要求后续阶段重复读取 README。若待确认项阻塞实现，停止并询问用户。

### 2. Forger：决策、草拟与预检

调度 `subagents/forger.md`，仅传入用户原始需求、Context Brief 路径和 Code Plan 输出路径。

Forger 读取简报、必要的目标源码和 API；项目存在 `CONVENTIONS.md` 时必须阅读。它将适用规约压缩到 `code-plan.md`，并写入精确文件清单、变更意图、API 事实和验证命令。不得修改应用代码。若规划被阻塞，停止并询问用户。

### 3. Act：执行

调度 `subagents/act.md`，仅传入 Code Plan 路径和 Execution Report 输出路径。

Act 仅读取计划和目标文件，机械实施变更并运行指定的构建或测试命令；不得重新探索业务文档或编码规约。失败时仅可检查修复计划内变更所需内容，最多尝试三次。写入 `execution-report.md`。

### 4. Eval：评估

调度 `subagents/eval.md`，仅传入 Execution Report 路径和 Verify Result 输出路径。

Eval 独立检查实际 diff 并运行相关既有验证；项目存在 `CONVENTIONS.md` 时必须阅读，用于判断证据、回归与验收门禁。仅当报告和 diff 无法确认意图时，才读取 Code Plan 或 Context Brief。写入 `verify-result.md`。项目没有测试框架时不得新建，只报告缺口。

## 恢复机制

- Scout 或 Forger 阻塞时，需要用户澄清。
- Act 三次修复失败后，需要用户决定。
- Eval 发现实现缺陷时回流 Act，发现计划缺陷时回流 Forger。最多回流两轮，超过后停止并报告。

## 执行日志

Eval 完成后写入 `execution-log.md`，记录四阶段结论、执行命令、阻塞项和最终结论。随后给出简短的提交信息建议；不得自动提交。

## 激活

触发时确认已启用该流水线，并在每次阶段切换时简短汇报进度。
