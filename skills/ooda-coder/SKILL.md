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
| 主session | 执行日志 | 汇总过程+识别卡点+询问用户 |

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
}
```

## 调度规则

### 第一步：调度 ooda-scout

使用task工具，prompt模板见 `ooda-scout-prompt.md`。将用户的业务需求填入 `{用户的业务需求描述}`。

接收scout返回的OODA-CONTEXT-BRIEF后，向用户简要汇报侦察结果，然后进入第二步。

### 第二步：调度 ooda-forger

使用task工具，prompt模板见 `ooda-forger-prompt.md`。填入：
- `{用户的业务需求描述}`：用户的原始需求
- `{scout返回的完整快照}`：原样传递，不做任何修改

接收forger返回的OODA-CODE-PLAN后，进入第三步。

### 第三步：调度 ooda-act

使用task工具，prompt模板见 `ooda-act-prompt.md`。将forger的完整代码计划填入 `{forger返回的完整代码计划}`。

接收act的结果后，进入第四步。

### 第四步：调度 ooda-eval

使用task工具，prompt模板见 `ooda-eval-prompt.md`。将act返回的文件路径列表填入 `{act返回的文件路径列表}`。

如果eval发现测试失败，将失败信息反馈给主session，由主session决定是否重新调度forger+act修正。

### 第五步：执行日志

全部subagent执行完毕后，主session负责：

1. 汇总各subagent的交互过程
2. 判断是否顺利，识别卡点
3. 生成执行日志内容
4. **询问用户**：是否需要将执行日志记录到文件？

执行日志格式：

```markdown
## 执行摘要
- 任务：{用户需求的一句话描述}
- OODA-E结果：{通过/需要修正}
- 涉及文件：{文件列表}

## 过程记录
- ooda-scout：{顺利/卡点，具体描述}
- ooda-forger：{顺利/卡点，具体描述}
- ooda-act：{顺利/卡点，具体描述}
- ooda-eval：{顺利/卡点，具体描述}

## 卡点纪要
[只记录问题，顺利的不记录]

## 决策建议
[建议开发者考虑：是否需要更新规约？是否需要补充文档？]
```

用户确认后写入执行日志文件（路径从项目级INDEX获取）。用户拒绝则不记录。

## 错误恢复机制

1. **编译失败**：将错误信息追加到CONTEXT-BRIEF，重新调度forger→act
2. **存量测试失败**：分析是代码问题还是测试问题
   - 代码问题：重新调度forger→act→eval
   - 测试问题：由eval修正测试
3. **规约违规**：重新调度scout补充遗漏的规约，再走完整流程
4. 最多重试2轮，超过后向用户报告详细错误

## 硬性规定

1. **越界禁止**：如需改动用户未明确说明的地方，必须经用户同意
2. **造轮子禁止**：禁止手写基础设施已有的能力，必须调用INDEX中的API
3. **文档保护**：所有subagent无权修改文档，发现不一致时汇报给主session
4. **精度零丢失**：API签名和规约条款在subagent间传递时必须原样传递
5. **上下文控制**：大文档读取由scout在独立上下文中完成，主session不直接读取项目大文档

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
