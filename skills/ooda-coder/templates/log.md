# ooda-coder 执行日志模板

时间戳格式：UTC 紧凑时间戳，格式为 `YYYYMMDDHHmmss`（如 `20260617093045`），每次执行唯一。

日志目录命名：`{UTC紧凑时间戳}-代码编写`

## 日志目录结构

```
{UTC紧凑时间戳}-代码编写/
├── context-brief.md       # scout 输出：项目结构、规约路径、业务背景
├── code-plan.md           # forger 输出：执行计划、完整代码、依赖签名
├── execution-report.md    # act 输出：写入文件、编译结果、自我纠正
├── verify-result.md       # eval 输出：交叉验证、回归测试、覆盖率分析
└── execution-log.md       # 本文件：执行日志汇总
```

## execution-log.md 模板

```markdown
# 执行日志

## 任务信息
- 时间戳：{UTC紧凑时间戳（YYYYMMDDHHmmss）}
- 时间：{YYYY-MM-DD HH:mm:ss UTC}
- 用户输入：{完整的用户输入，不得缩减，如果使用skill只记录用户输入内容}
- 使用skill：{skill名称，无则写"无"}
- 模型：{模型名称}

## 文件链路
- context-brief.md：[简述scout输出的关键信息]
- code-plan.md：[简述forger的执行计划]
- execution-report.md：[简述act的执行结果]
- verify-result.md：[简述eval的验证结果]

## 执行过程
- ooda-scout：{顺利/卡点，具体描述}
- ooda-forger：{顺利/卡点，具体描述}
- ooda-act：{顺利/卡点，具体描述}
- ooda-eval：{顺利/卡点，具体描述}

## 卡点纪要
[只记录问题，顺利的不记录。没有则写"无"]
```
