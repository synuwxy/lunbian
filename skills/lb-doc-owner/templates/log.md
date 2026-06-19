# lb-doc-owner 执行日志模板

时间戳格式：UTC 紧凑时间戳，格式为 `YYYYMMDDHHmmss`（如 `20260617093045`），每次执行唯一。

日志目录命名：`{UTC紧凑时间戳}-文档维护`

## 日志目录结构

```
{UTC紧凑时间戳}-文档维护/
├── business-report.md     # 业务层子代理报告
├── infra-report.md        # 基础设施层子代理报告
└── execution-log.md       # 本文件：执行日志汇总
```

## execution-log.md 模板

```markdown
# 执行日志

## 任务信息
- 时间戳：{UTC紧凑时间戳（YYYYMMDDHHmmss）}
- 时间：{YYYY-MM-DD HH:mm:ss UTC}
- 用户输入：{完整的用户输入，不得缩减}
- 使用skill：lb-doc-owner
- 模型：{模型名称}

## 文件链路
- business-report.md：[简述业务层子代理的变更摘要]
- infra-report.md：[简述基础设施层子代理的变更摘要]

## 执行过程
- INDEX探测：{找到/未找到，处理方式}
- 子代理-业务层：{顺利/卡点，具体描述}
- 子代理-基础设施层：{顺利/卡点，具体描述}
- 项目级文档：{无需更新/已建议变更/开发者已确认}
- 跨层影响：{无/有，具体描述}

## 卡点纪要
[只记录问题，顺利的不记录。没有则写"无"]
```
