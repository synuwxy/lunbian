<h1 align="center">轮扁</h1>

---

《庄子》里有个叫轮扁的老木匠，做了一辈子车轮。他说真正的手艺不在书里，在手和木头几十年互相磨合之后长出来的直觉里。轮教会手该用多大力，手教会轮该成什么形。这个仓库想干的，差不多就是这件事。

### 这是什么？

这个仓库记录的是一种协作范式——开发者并非单向指令AI，而是**与AI共同生长**。

它的循环如此运转：你写文档，AI构建。你检视成果，沉淀经验，打磨文档。然后进入下一轮。周而复始，文档在反复浸润中生出分寸，AI的输出开始预判你尚未言明的意图。谁都没有停在原地。每一次意外的输出都在重塑开发者，每一行斟酌过的文档都在重塑模型。

这不是提示词工程。这是**以重复抵达精湛的匠艺**——意图与能力的缓慢互渗，直到"谁在控制"与"谁在回应"之间的那条线，渐渐消融。

### 包结构

```
lunbian/
├── origin/                          # 指导文档
│   ├── AI编程方法论：骑士、缰绳和千里马.md    # 理论框架
│   └── AI编程体系建设：工具体系.md          # 工具体系设计
│
├── skills/                          # 技能实现
│   ├── lb-master/                   # 编排层：统一入口，调度所有 skill
│   │   └── SKILL.md
│   ├── lb-doc-owner/                # 文档层：统一文档维护（含初始化）
│   │   ├── SKILL.md
│   │   ├── subagents/
│   │   │   ├── business.md
│   │   │   ├── infra.md
│   │   │   └── bootstrap.md
│   │   └── templates/
│   │       └── log.md
│   ├── ooda-coder/                  # 执行层：基于OODA-E循环的代码编写
│   │   ├── SKILL.md
│   │   ├── subagents/
│   │   │   ├── scout.md
│   │   │   ├── forger.md
│   │   │   ├── act.md
│   │   │   └── eval.md
│   │   └── templates/
│   │       └── log.md
│   ├── reviewer/                    # 校验层：独立审查代码改动
│   │   ├── SKILL.md
│   │   └── templates/
│   │       └── log.md
│   ├── architecture-guard/          # 校验层：架构约束的机械化检查
│   │   ├── SKILL.md
│   │   └── templates/
│   │       └── log.md
│   ├── debugger/                    # 校验层：系统化问题诊断
│   │   ├── SKILL.md
│   │   └── templates/
│   │       └── log.md
│   └── consolidation/               # 进化层：沉淀分析
│       ├── SKILL.md
│       ├── subagents/
│       │   ├── ooda.md
│       │   ├── doc-debug.md
│       │   └── review.md
│       ├── templates/
│       │   └── log.md
│       └── categories/
│           ├── doc-quality.md
│           ├── skill-design.md
│           ├── capability-limits.md
│           ├── user-input.md
│           └── execution-deviation.md
│
├── deprecated/                       # 日落区：已废弃的技能与文件
│   └── skills/
│       ├── business-doc-owner/
│       ├── infra-doc-owner/
│       ├── product-doc-owner/
│       └── lb-bootstrap/
│
└── README.md
```

### 指导文档

| 文档 | 描述 |
|------|------|
| [AI编程方法论：骑士、缰绳和千里马.md](origin/AI编程方法论：骑士、缰绳和千里马.md) | 理论框架：骑士、缰绳、千里马三个角色，对齐、共演化、回合制等核心机制 |
| [AI编程体系建设：工具体系.md](origin/AI编程体系建设：工具体系.md) | 工具体系设计：基础层、文档层、执行层、校验层、进化层五层架构 |

### 技能说明

| 技能 | 层级 | 描述 |
|------|------|------|
| [lb-master](skills/lb-master/SKILL.md) | 编排层 | 统一入口。感知项目状态、解析用户意图、路由到合适的 skill、汇总执行结果 |
| [lb-doc-owner](skills/lb-doc-owner/SKILL.md) | 文档层 | 统一文档维护。通过并行子代理分别维护业务层和基础设施层文档，协调者负责边界管控和项目级文档维护；当项目文档不存在时，自动调度初始化子代理完成框架搭建 |
| [ooda-coder](skills/ooda-coder/SKILL.md) | 执行层 | 基于OODA-E循环的代码编写。在文档的约束下执行，规范大于自由，基建优先于手写 |
| [reviewer](skills/reviewer/SKILL.md) | 校验层 | 独立审查代码改动。提供客观的质量评估，生成与评估必须分离 |
| [architecture-guard](skills/architecture-guard/SKILL.md) | 校验层 | 架构约束的机械化检查。确保代码合入前符合架构规范 |
| [debugger](skills/debugger/SKILL.md) | 校验层 | 系统化问题诊断。先定位根因再提出修复方案，诊断与修复分离 |
| [consolidation](skills/consolidation/SKILL.md) | 进化层 | 沉淀分析。分析执行日志积累，识别文档框架缺陷，为开发者提供决策依据 |

### 核心思想

**三个角色**：
- **骑士（开发者）**：指挥官，决策者，最终责任人
- **缰绳（文档）**：开发者、AI、代码三者之间的介质
- **千里马（AI）**：在约束范围内执行的行动者

**核心机制**：
- **对齐**：通过文档让开发者和AI对项目达成共同理解
- **共演化**：骑士、缰绳、千里马三者相互磨合、共同进化
- **回合制**：小回合（一次输入一次输出）是最小操作单元

**核心原则**：
1. 方向优先于速度
2. 对齐优先于能力
3. 缰绳是活的，在共演化中持续进化
4. 质量可迁移——缰绳是模型无关的资产
5. 骑士是唯一有权修改缰绳的人
6. 生成与评估分离
7. 提示词会过时，缰绳不会

### 日落区

已废弃的技能存放在 `deprecated/skills/` 目录下，保留供参考：

| 技能 | 原层级 | 描述 | 废弃原因 |
|------|--------|------|----------|
| [product-doc-owner](deprecated/skills/product-doc-owner/SKILL.md) | 文档层 | 项目级文档owner | 功能已整合入 lb-doc-owner |
| [business-doc-owner](deprecated/skills/business-doc-owner/SKILL.md) | 文档层 | 业务级文档owner | 功能已整合入 lb-doc-owner |
| [infra-doc-owner](deprecated/skills/infra-doc-owner/SKILL.md) | 文档层 | 基础设施文档owner | 功能已整合入 lb-doc-owner |
| [lb-bootstrap](deprecated/skills/lb-bootstrap/SKILL.md) | 基础层 | 项目初始化 | 功能已整合入 lb-doc-owner 的初始化子代理 |
