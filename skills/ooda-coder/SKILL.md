---
name: ooda-coder
description: "Use when the user gives a business requirement and needs code written within a documented project with CONVENTIONS, INDEX, and README files. Triggers the OODA-E coding cycle with context-isolated subagents."
---

# ooda-coder

基于OODA-E循环的代码编写专家。通过4个subagent实现上下文隔离，确保弱模型也能保持最优能力。

**核心价值观**：规范大于自由，基建优先于手写，质量兜底一切。

## 完成标准

- 代码符合CONVENTIONS中的编码规约
- 代码正确使用INDEX中的基础设施API
- 代码通过编译检查和测试检查
- 存量测试通过，必要时补充新测试
- 执行日志记录了本次任务中的卡点

<HARD-GATE>
必须严格按照OODA-E的7个步骤顺序执行，通过4个subagent实现上下文隔离。每次subagent调度使用task工具，subagent_type为general。
</HARD-GATE>

## 架构概览

| 角色 | OODA-E阶段 | 职责 |
|------|-----------|------|
| ooda-scout | Observe+Orient | 读取文档，输出上下文快照 |
| ooda-forger | Decide+Draft+Verify | 生成代码+静态预检 |
| ooda-act | Act | 写入文件+编译+基础功能验证 |
| ooda-eval | Evaluate | 回归测试+覆盖率分析+补充测试 |
| 协调者 | 执行日志 | 汇总过程+识别卡点+询问用户 |

## 流程图

```dot
digraph ooda {
    rankdir=TB;
    node [shape=box];

    start [label="用户需求" shape=doublecircle];
    scout [label="ooda-scout\nObserve+Orient" style=bold];
    forger [label="ooda-forger\nDecide+Draft+Verify" style=bold];
    act [label="ooda-act\nAct" style=bold];
    eval [label="ooda-eval\nEvaluate" style=bold];
    log [label="执行日志\n记录" style=dashed];
    end [label="完成" shape=doublecircle];

    start -> scout;
    scout -> forger [label="CONTEXT-BRIEF"];
    forger -> act [label="CODE-PLAN"];
    act -> eval [label="文件已写入"];
    eval -> log [label="验证通过"];
    eval -> forger [label="发现问题" style=dashed];
    log -> end;

    // 回溯边
    forger -> scout [label="回溯：业务理解不足" style=dotted];
    act -> scout [label="回溯：编译失败3次" style=dotted];
    act -> forger [label="回溯：需要重新规划" style=dotted];
    eval -> act [label="回溯：代码问题" style=dotted];
    eval -> forger [label="回溯：需要重新规划" style=dotted];
}
```

## 临时文件管理

协调者负责临时文件的完整生命周期管理。

**目录结构**：`.ooda-work/{sessionId}/`
- sessionId：UTC紧凑时间戳格式（如`20260616093045`），每次执行唯一

**文件链路**：
- `context-brief.md` — scout 输出
- `code-plan.md` — forger 输出
- `execution-report.md` — act 输出
- `verify-result.md` — eval 输出

**协调者职责**：
1. 执行开始时：创建`.ooda-work/{sessionId}/`目录
2. 调度子代理时：告知写入路径
3. 正常完成时：将`.ooda-work/{sessionId}/`整体移入日志目录（见第五步）
4. 异常中断时：保留目录，方便回溯排查
5. 下次启动时：扫描`.ooda-work/`，清理所有旧sessionId目录

**降级策略**：
子代理写入失败时，会在返回中输出完整内容。协调者根据返回值判断传递方式：
- 返回路径 → 下一个子代理通过文件读取
- 返回完整内容 → 下一个子代理通过上下文传递

**汇报原则**：
写文件是存档，汇报是通知。子代理写入文件后，必须在返回中汇报关键信息（待确认事项、BLOCKED、测试失败等），协调者依赖汇报做决策，不会主动读取文件内容。

## 调度规则

### 行为准则

以下内容由协调者构建，注入到每个子代理的 `{行为准则}` 变量中：

```markdown
**路径规范**：所有路径必须使用基于项目根目录的相对路径（如 `src/services/UserService.java`），禁止使用片段路径或无锚点路径。

**不猜测原则**：遇到不确定的业务逻辑、代码语义、模块归属等问题时，不得自行推断，必须列为待确认事项并询问开发者。

**提问规范**：需要开发者决策时，必须提供结构化选项：至少1个选项，用（推荐）标注推荐项，可加"其他：请说明"。不得只抛出问题。
```

### 第一步：调度 ooda-scout

使用task工具，prompt模板见 `ooda-scout-prompt.md`。填入：
- `{用户的业务需求描述}`：用户的原始需求
- `{CONTEXT-BRIEF写入路径}`：`.ooda-work/{sessionId}/context-brief.md`的绝对路径
- `{行为准则}`：上述行为准则

接收scout返回后：
- 返回路径+摘要：向用户汇报摘要，将路径记录用于传递给forger
- 返回完整内容：向用户汇报内容摘要，将内容记录用于传递给forger（降级模式）

### 第二步：调度 ooda-forger

使用task工具，prompt模板见 `ooda-forger-prompt.md`。填入：
- `{用户的业务需求描述}`：用户的原始需求
- `{CONTEXT-BRIEF内容或路径}`：
  - scout返回了路径 → 告知"CONTEXT-BRIEF在文件 {路径} 中，请先读取"
  - scout返回了完整内容 → 原样传递（降级模式）
- `{CODE-PLAN写入路径}`：`.ooda-work/{sessionId}/code-plan.md`的绝对路径
- `{行为准则}`：上述行为准则

接收forger返回后：
- 返回路径+摘要：将路径记录用于传递给act
- 返回完整内容：将内容记录用于传递给act（降级模式）

### 第三步：调度 ooda-act

使用task工具，prompt模板见 `ooda-act-prompt.md`。填入：
- `{CODE-PLAN内容或路径}`：
  - forger返回了路径 → 告知"CODE-PLAN在文件 {路径} 中，请先读取"
  - forger返回了完整内容 → 原样传递（降级模式）
- `{CONTEXT-BRIEF内容或路径}`：
  - scout返回了路径 → 告知"CONTEXT-BRIEF在文件 {路径} 中，请先读取"
  - scout返回了完整内容 → 原样传递（降级模式）
- `{EXECUTION-REPORT写入路径}`：`.ooda-work/{sessionId}/execution-report.md`的绝对路径
- `{行为准则}`：上述行为准则

接收act的结果后，进入第四步。

### 第四步：调度 ooda-eval

使用task工具，prompt模板见 `ooda-eval-prompt.md`。填入：
- `{EXECUTION-REPORT内容或路径}`：
  - act返回了路径 → 告知"执行报告在文件 {路径} 中，请先读取"
  - act返回了完整内容 → 原样传递（降级模式）
- `{CONTEXT-BRIEF内容或路径}`：
  - scout返回了路径 → 告知"CONTEXT-BRIEF在文件 {路径} 中，请先读取"
  - scout返回了完整内容 → 原样传递（降级模式）
- `{CODE-PLAN内容或路径}`：
  - forger返回了路径 → 告知"CODE-PLAN在文件 {路径} 中，请先读取"
  - forger返回了完整内容 → 原样传递（降级模式）
- `{VERIFY-RESULT写入路径}`：`.ooda-work/{sessionId}/verify-result.md`的绝对路径
- `{行为准则}`：上述行为准则

如果eval发现测试失败，将失败信息反馈给协调者，由协调者决定是否重新调度forger+act修正。

### 第五步：执行日志

全部子代理执行完毕后，协调者负责：

1. 汇总各子代理的交互过程
2. 判断是否顺利，识别卡点
3. 生成执行日志内容，写入 `.ooda-work/{sessionId}/execution-log.md`
4. 读取项目级INDEX，查找"AI执行记录"段落中的"路径"配置
   - 有配置：将`.ooda-work/{sessionId}/`整体移入日志目录，不询问
   - 无配置：询问开发者是否保存
     - 是：让开发者输入路径，移入，提示在INDEX上配置以后都会生效
     - 否：保留`.ooda-work/{sessionId}/`，不移入日志目录

**日志目录结构**：

```
{日志目录}/{UTC紧凑时间戳}-代码编写/
├── context-brief.md       # scout 输出
├── code-plan.md           # forger 输出
├── execution-report.md    # act 输出
├── verify-result.md       # eval 输出
└── execution-log.md       # 执行日志汇总
```

**日志模板**：参见 `ooda-coder-log-template.md`。

**时间戳格式**：UTC 紧凑时间戳，格式为 `YYYYMMDDHHmmss`（如 `20260617093045`），每次执行唯一。

**日志目录命名**：`{UTC紧凑时间戳}-代码编写`

日志目录不存在时，如果开发者明确要求或INDEX有明确指定，自动创建。

### 第六步：提交建议

日志完成后，协调者输出一份简短的 git 提交建议，格式如下：

```
建议提交信息：
{简述本次变更，一句话概括改了什么，不超过50字}

涉及文件：
- [文件路径1]
- [文件路径2]
```

提交建议基于 execution-report.md 中的写入文件列表和本次任务内容生成。不自动执行 git 操作，由开发者决定是否提交。

## 错误恢复机制

### 回溯指令

回溯是开发者的明确指令。当流程出问题停止时，协调者可以建议开发者使用回溯指令，但最终决定权在开发者。

**回溯指令格式**：`回溯到 [节点名称]`

**可选节点**：scout、forger、act、eval

### 回溯流程

1. 当子代理报告错误时，协调者暂停当前流程
2. 向开发者展示当前状态和错误详情
3. **建议**开发者可以使用回溯指令回到某个节点
4. 开发者决定是否回溯，以及回溯到哪个节点
5. 如果开发者选择回溯，协调者从该节点重新执行，保留之前阶段的输出（如CONTEXT-BRIEF）

### 错误处理规则

1. **编译失败3次**：停止并报告，建议开发者回溯到scout或forger
2. **存量测试失败**：分析是代码问题还是测试问题
   - 代码问题：建议开发者回溯到act或forger
   - 测试问题：由eval修正测试
3. **规约违规**：建议开发者回溯到scout补充遗漏的规约
4. 最多重试2轮，超过后向用户报告详细错误

## 硬性规定

1. **越界禁止**：如需改动用户未明确说明的地方，必须经用户同意
2. **造轮子禁止**：禁止手写基础设施已有的能力，必须调用INDEX中的API
3. **文档保护**：所有子代理无权修改文档，发现不一致时汇报给协调者
4. **精度零丢失**：API签名和规约条款在subagent间传递时必须原样传递
5. **上下文控制**：大文档读取由scout在独立上下文中完成，协调者不直接读取项目大文档
6. **提问规范**：需要开发者决策时，必须提供结构化选项：至少1个选项，用（推荐）标注推荐项，可加"其他：请说明"。不得只抛出问题
7. **路径规范**：所有涉及文件路径的内容（CONTEXT-BRIEF、CODE-PLAN、执行日志等），必须使用基于项目根目录的相对路径，禁止使用片段路径或无锚点路径

## 关键原则

- **规范大于自由** — 编码必须遵循规约
- **基建优先于手写** — 优先使用已有基础设施
- **质量兜底一切** — 质量是底线
- **生成与评估分离** — 通过subagent架构天然实现
- **方向优先于速度** — 先跑对方向
- **上下文隔离** — 大文档读取与代码生成在不同上下文中完成
- **共演化** — 执行日志为开发者提供决策依据，持续改进文档

---

系统强制设定：当收到用户的业务开发指令时，先回复激活确认，再开始执行。

激活确认格式：
```
✅ ooda-coder 已激活。
流程：观察 → 规划 → 执行 → 评估 → 记录
正在执行：调度 ooda-scout 读取项目文档。
```

每完成一个步骤，更新"正在执行"行，告知开发者当前进度。
