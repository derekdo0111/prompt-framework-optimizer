# 问到位 — Prompt Framework Optimizer

自动匹配12种Prompt框架的WorkBuddy Skill库。

## 核心理念

你只需要用自然语言描述需求，剩下的交给「问到位」：

1. **自动提取信号** — 从你的输入中判断复杂度、格式需求、角色需求
2. **最少追问** — 最多3道选择题，能猜到的绝不多问
3. **智能匹配** — 从12个框架中选出最适合的那个
4. **一键输出** — 生成结构化、可直接使用的优化Prompt

## 12个框架速览

| 框架 | 适用场景 |
|------|----------|
| TAC | 极简三要素，一句话任务 |
| APE | 行动导向，有目的的任务 |
| RACE | 通用均衡，日常首选 |
| ERA | 角色优先，风格模仿 |
| RISE | 步骤清晰，分步执行 |
| ROSES | 问题解决，复杂方案 |
| COAST | 场景驱动，重视上下文 |
| BROKE | 迭代优化，多轮打磨 |
| TRACE | 示例驱动，精确输出 |
| CARE | 结果导向，明确交付物 |
| ICIO | 结构化I/O，数据处理 |
| C.R.I.S.P.E | 深度对话，洞察思辨 |

## 安装

```bash
# 克隆到 skills 目录
git clone https://github.com/derekdo0111/prompt-framework-optimizer.git ~/.workbuddy/skills/prompt-framework-optimizer

# 或在 WorkBuddy 中直接用
/prompt-framework-optimizer 你的自然语言需求
```

## 示例

用户输入："帮我分析新能源汽车竞争格局"

→ 自动提取：复杂分析、无格式要求
→ 追问3题：复杂度/格式/角色
→ 匹配框架：RISE
→ 输出结构化Prompt，直接复制使用

## 设计哲学

> 不要让用户学习12个框架——让框架自动适配用户。
