# architecture-guard 执行日志模板

时间戳格式：UTC 紧凑时间戳，格式为 `YYYYMMDDHHmmss`（如 `20260617093045`），每次执行唯一。

日志目录命名：`{UTC紧凑时间戳}-架构检查`

## 日志目录结构

```
{UTC紧凑时间戳}-架构检查/
├── guard-report.md        # 检查报告
└── execution-log.md       # 本文件：执行日志汇总
```

## 记录原则

- 发现违规：详细记录各维度检查结果、违规项、文件位置
- 全部通过：简略记录总体结论即可

## execution-log.md 模板

```markdown
# 执行日志

## 任务信息
- 时间戳：{UTC紧凑时间戳（YYYYMMDDHHmmss）}
- 时间：{YYYY-MM-DD HH:mm:ss UTC}
- 用户输入：{完整的用户输入，不得缩减}
- 使用skill：architecture-guard
- 模型：{模型名称}

## 检查结果
- 总体结论：[通过/不通过]
- 违规项数：[数量]

## 文件链路
- guard-report.md：[简述检查结论，有违规时列出关键违规项]

## 卡点纪要
[只记录问题，顺利的不记录。没有则写"无"]
```
