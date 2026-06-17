# consolidation 执行日志模板

时间戳格式：UTC 紧凑时间戳，格式为 `YYYYMMDDHHmmss`（如 `20260617093045`），每次执行唯一。

日志目录命名：`{UTC紧凑时间戳}-沉淀分析`

## 日志目录结构

```
{UTC紧凑时间戳}-沉淀分析/
├── ooda-analysis.md         # ooda-coder 分析结果
├── doc-debug-analysis.md    # lb-doc-owner + debugger 分析结果
├── review-analysis.md       # reviewer + architecture-guard 分析结果
├── consolidation-report.md  # 汇总分析报告
└── execution-log.md         # 本文件：执行日志汇总
```

## execution-log.md 模板

```markdown
# 执行日志

## 任务信息
- 时间戳：{UTC紧凑时间戳（YYYYMMDDHHmmss）}
- 时间：{YYYY-MM-DD HH:mm:ss UTC}
- 用户输入：{完整的用户输入，不得缩减}
- 使用skill：consolidation
- 模型：{模型名称}

## 分析范围
- 已分析的日志数量：[数量]
- 新增日志数量：[数量]
- 跳过已分析日志：[数量]

## 文件链路
- ooda-analysis.md：[简述 ooda-coder 分析结果]
- doc-debug-analysis.md：[简述文档与调试分析结果]
- review-analysis.md：[简述审查与架构分析结果]
- consolidation-report.md：[简述汇总分析结论]

## 执行过程
- 索引读取：{顺利/卡点，具体描述}
- 子代理-ooda：{顺利/卡点，具体描述}
- 子代理-文档调试：{顺利/卡点，具体描述}
- 子代理-审查架构：{顺利/卡点，具体描述}
- 汇总分析：{顺利/卡点，具体描述}

## 卡点纪要
[只记录问题，顺利的不记录。没有则写"无"]
```
