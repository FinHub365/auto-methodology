# Auto Methodology

**让 AI 在你睡觉时跑 100+ 次实验。**

一个 Claude Code 元技能（meta-skill），能把任何可量化的目标变成自主优化循环。基于 Andrej Karpathy 在 [AutoResearch](https://github.com/karpathy/autoresearch) 中使用的模式——一个 AI agent 跑了 700 次实验，发现了连世界级 ML 研究者都遗漏的优化。

```
人类写 program.md       AI 跑实验          AI 评估           AI 保留赢家
       |                    |                 |                  |
       v                    v                 v                  v
  "优化 X"  ──>  修改 train.py  ──>  测量 val_bpb  ──>  git commit / revert
                       ^                                        |
                       └────────────────────────────────────────┘
                                    永不停止
```

---

## 核心思想

所有优化问题——无论什么领域——都能归结为**三原件**：

| 原件 | 是什么 | 示例 |
|------|-------|------|
| **可编辑资产** | agent 唯一能改的那个文件 | `train.py`、`queries.sql`、`prompts.md` |
| **标量指标** | 一个数字：变好了还是变差了，没有歧义 | `val_bpb`、`p95_latency_ms`、`lighthouse_score` |
| **时间限定循环** | 每次实验固定时长，确保公平比较 | 5 分钟壁钟时间 |

把你的问题映射到这三个要素上，agent 负责剩下的一切——提出改动、测量结果、保留胜者、回滚败者、记录一切。

## 这个技能做什么

它不是一个优化某个具体东西的工具。它是一个**制造优化工具的工具**。

告诉它你想改进什么，它会生成：

```
auto-[你的领域]/
  program.md          # 研究纲领：目标、约束、规则
  [可编辑资产]         # agent 在上面做实验的文件
  evaluate.sh         # 不可变的评分逻辑
  run_loop.sh         # 棘轮循环，带超时和日志
  results.tsv         # 只追加的实验账本
```

然后把任意 coding agent 指向 `program.md`，让它跑就行了。

## 真实案例

| 项目 | 优化对象 | 实验次数 | 成果 |
|------|---------|---------|------|
| Karpathy AutoResearch | LLM 训练效率 | 700 | 训练速度提升 11%，发现 QK-Norm 缺失乘法器 |
| Shopify Liquid | 模板渲染引擎 | ~100 | 渲染速度提升 53%，内存减少 61% |
| SkyPilot（16 GPU 并行） | LLM 训练 | 8小时 910 次 | 大规模并行探索 |

## 六大原型

技能会将你的问题匹配到正确的循环模式：

```
AutoResearch    提出假设 ──> 跑实验 ──> 测量 ──> 保留/丢弃
AutoBuild       改代码/配置 ──> 跑测试/基准 ──> 保留/回滚
AutoSearch      搜索 ──> 过滤 ──> 排序 ──> 综合
AutoOperate     检查状态 ──> 分类 ──> 行动 ──> 验证
AutoContent     生成草稿 ──> 用标准打分 ──> 修订 ──> 保留最佳
AutoAnalysis    候选解释 ──> 用证据检验 ──> 提升最强
```

## 四面架构

严格的职责分离，防止 agent 钻系统漏洞：

```
┌──────────────────────────────────────────────────────┐
│  纲领面 PROGRAM SURFACE（人类控制）                    │
│  program.md：目标、约束、晋升规则                      │
├──────────────────────────────────────────────────────┤
│  可变面 MUTABLE SURFACE（Agent 控制）                  │
│  train.py / queries.sql / prompts.md                 │
├──────────────────────────────────────────────────────┤
│  评判面 ORACLE SURFACE（不可变）                       │
│  evaluate.sh、测试数据、评分函数                       │
├──────────────────────────────────────────────────────┤
│  账本面 LEDGER SURFACE（只追加）                       │
│  results.tsv：每次运行都记录，无论成败                  │
└──────────────────────────────────────────────────────┘
```

Agent **不能修改评判面**。如果它能改，它就会去优化记分牌而不是真正的问题。

## 棘轮机制

进步是单调的。基于 Git 的棘轮确保代码库只往前走：

```
  指标改善 + 约束满足       ──>  保留 commit（新基线）
  指标持平 + 方案更简洁     ──>  保留 commit
  指标退步                 ──>  git reset --hard HEAD~1
  实验崩溃                 ──>  记录错误，回滚，继续
  证据模糊                 ──>  标记 "hold"，继续下一个
```

## 八大领域示例

技能内置了详细的领域适配方案：

| 领域 | 可编辑资产 | 指标 | 循环时长 |
|------|-----------|------|---------|
| ML 训练 | `train.py` | val_bpb | 5 分钟 |
| Web 性能 | `index.html` + 内联资源 | Lighthouse 分数 | 2 分钟 |
| SQL 优化 | `queries.sql` | P95 查询时间 (ms) | 1 分钟 |
| 提示词工程 | `prompts.md` | 标注集 F1 分数 | 3 分钟 |
| 编译优化 | `CMakeLists.txt` | 完整构建时间 (秒) | 60 分钟 |
| 内容文案 | `landing_copy.md` | 复合质量分 | 1 分钟 |
| API 性能 | `handler.go` | P99 延迟 (ms) | 3 分钟 |
| LLM 推理 | `serve_config.py` | tokens/sec（质量门控） | 5 分钟 |

完整配置（含评估脚本和探索方向）见 [`references/domain-examples.md`](references/domain-examples.md)。

## 五条护栏

保证循环诚实运转的硬规则：

1. **不可测则不优化。** 先建立自动化评分，再启动循环。
2. **不可事后改定义。** 指标和晋升规则在循环开始前锁定。
3. **跑不通就不要扩。** 单循环稳定后才能加并行或编排。
4. **更多自主 ≠ 更好方法论。** 根据风险等级设置人类检查点。
5. **先验证再编排。** 一个 agent、一个循环、一个指标先产出价值。

## 安装

```bash
# 克隆并安装
git clone https://github.com/FinHub365/auto-methodology.git
claude skill install ./auto-methodology

# 或者直接把技能目录复制到项目的 .claude/skills/ 下
cp -r auto-methodology /path/to/project/.claude/skills/
```

## 使用方式

直接描述你想优化什么，技能会自动激活：

```
> "我想自动优化我的 SQL 查询性能"
> "帮我搭一个过夜跑实验的提示词优化循环"
> "给我的 API 延迟建一套 AutoBuild 系统"
```

技能会引导你完成：

1. 明确目标和基线
2. 识别三原件
3. 选择原型
4. 生成全部文件
5. 启动前检查清单
6. 启动与监控指南

## 项目结构

```
auto-methodology/
  SKILL.md                          # 技能定义（核心逻辑）
  README.md                         # 你在这里
  references/
    program-template.md             # program.md 填空模板
    domain-examples.md              # 8 个领域的完整配置示例
```

## 什么时候不该用

不是所有问题都适合自主循环：

- **需要人类创意判断** —— 设计美感、品牌调性
- **评估成本极高** —— 需要真实用户反馈或跑一周的 A/B 测试
- **解空间极小** —— 只有两三种可能的配置
- **高风险变更** —— 生产数据库 schema、核心基础设施

## 设计哲学

> "你不再直接编写 Python 文件了。你在编写 program.md —— 你在编程那个程序。"
> —— Andrej Karpathy

范式转变：

| 时代 | 谁做什么 |
|------|---------|
| 传统开发 | 人类写代码、人类测试、人类改进 |
| Vibe Coding | 人类描述需求、AI 写代码、人类审查 |
| **Auto** | **人类定方向、AI 自主实验、AI 自主评估、AI 自主改进、人类偶尔检查** |

你不需要知道最优学习率是多少。你只需要知道"学习率值得探索"。Agent 会找到那个最优值——以及你根本没想到去调的东西。

## 许可证

MIT
