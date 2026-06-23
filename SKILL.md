---
name: prompt-framework-optimizer
description: >-
  「问到位」— Automatically select the best Prompt framework from 12 options
  (TAC, APE, RACE, ERA, RISE, ROSES, COAST, BROKE, TRACE, CARE, ICIO, C.R.I.S.P.E)
  based on the user's natural-language task. This skill first extracts signals from the raw
  user input, then asks at most 3 clarifying multiple-choice questions before matching to
  the optimal framework and outputting a fully structured, ready-to-use prompt. The user
  never needs to learn or choose among frameworks — they simply describe what they want
  in plain language.
agent_created: true
---

# Prompt Framework Optimizer

Automatically determine the best Prompt framework and produce an optimized prompt. The user writes naturally; this skill handles all framework selection behind the scenes.

## Trigger Conditions

Invoke this skill when:
- The user asks to write, optimize, or structure a prompt for an AI/LLM
- The user describes a task and explicitly says "帮我优化prompt" / "用最好的框架写prompt" / "帮我结构化这个需求"
- The user asks which framework to use: "用什么框架比较好" / "哪个框架适合"
- The user pastes a rough idea and says "帮我完善成好用的prompt"
- The user says "用prompt-framework-optimizer" or similar

## Workflow

### Phase 1 — Signal Extraction

Read the user's raw input. Scan for the following signals WITHOUT asking any questions yet:

| Signal | Keywords/Patterns | Implication |
|--------|-------------------|-------------|
| Complexity: HEAVY | "深度分析", "研究报告", "策略", "创意", "哲学", "设计", "论文" | → C.R.I.S.P.E candidate |
| Complexity: MEDIUM | "报告", "分析", "对比", "方案", "计划", "总结", "拆解" | → Need Q1 |
| Complexity: LIGHT | "你好", "是什么", "怎么", "帮我查", "翻译", "一句话" | → TAC/APE candidate |
| Structured output | "JSON", "表格", "CSV", "模板", "格式", "字段", "转换成" | → ICIO/TRACE candidate |
| Role required | "作为XX", "假装你是", "用XX的语气", "以XX视角", "模仿", "角色扮演" | → ERA/RISE candidate |
| Example needed | "参考这个", "像这样", "类似这个", "示例", "样例" | → TRACE/CARE candidate |
| Iteration needed | "初稿", "迭代", "持续优化", "先出一版", "再改" | → BROKE candidate |
| Scenario heavy | "在XX情况下", "对于XX场景", "面向XX用户", "使用场景", "受众" | → COAST candidate |

For each detected signal, assign a confidence score:
- **HIGH confidence** → auto-select the matching framework, skip related questions
- **MEDIUM confidence** → flag for the relevant Q question

### Phase 2 — Ask Clarifying Questions (max 3)

Only ask questions for signals that could NOT be auto-resolved in Phase 1 with HIGH confidence.

**Question priority: Q1 > Q2 > Q3.** Stop asking when the framework is uniquely determined.

#### Q1 — Complexity Level

Ask ONLY if the complexity signal is medium or ambiguous. Skip if Phase 1 already determined heavy/light with HIGH confidence.

```
这个问题有多复杂？

A. 很简单 — 一两句话就能说清，不需要背景铺垫
B. 有点复杂 — 需要说明背景或拆成几个步骤
C. 很复杂 — 需要深度思考、多角度分析或反复打磨
```

Answers map to:
- A → Frameworks: TAC, APE
- B → Frameworks: RACE, RISE, COAST, CARE, ICIO, TRACE → continue to Q2
- C → Frameworks: ROSES, BROKE, C.R.I.S.P.E → continue to Q2 or Q3

#### Q2 — Output Format Requirement

Ask ONLY if the user didn't mention format/structure in their input AND Q1 answer was B or C. Skip if the input already specifies "JSON"/"表格"/"模板" etc.

```
输出有什么硬性要求吗？

A. 需要特定格式 — 比如表格、JSON、结构化字段、特定模板
B. 格式不严格 — 但如果有示例参考会更好
C. 没要求 — 内容为主，格式自由
```

Answers map to:
- A → ICIO, TRACE
- B → TRACE, CARE
- C → RACE, RISE, COAST, ROSES, BROKE → continue to Q3 if still ambiguous

#### Q3 — Role Definition

Ask ONLY if the user didn't specify a role/speaker identity AND multiple frameworks remain.

```
你希望回答以什么身份/风格呈现？

A. 特定专业角色 — 比如分析师、工程师、作家、教练
B. 不限角色 — 客观中立即可
C. 需要反复迭代 — 先出初稿，再根据反馈打磨
```

Answers map to:
- A → RISE, ERA, ROSES
- B → RACE, COAST, CARE
- C → BROKE

### Phase 3 — Framework Selection

After Phase 1 & 2, use this decision matrix to pick the final framework:

| Q1 | Q2 | Q3 | Final Framework |
|----|----|----|-----------------|
| A | — | — | **APE** (default for simple tasks with purpose) |
| A | — | — | **TAC** (when no purpose context needed) |
| B | A | — | **ICIO** (data/format heavy) or **TRACE** (example-driven format) |
| B | B | — | **TRACE** (example available) or **CARE** (no example provided) |
| B | C | A | **RISE** (step-by-step with role) |
| B | C | B | **RACE** (balanced general-purpose) |
| B | C | C | **BROKE** (iterative mid-complexity) |
| C | A | — | **ICIO** (structured deep analysis) |
| C | B | — | **TRACE** (deep with examples) |
| C | A/B | — | **ROSES** (complex problem-solving) |
| C | C | A | **ROSES** or **RISE** (role-driven complex) |
| C | C | B | **ROSE** or **COAST** (scenario-driven) |
| C | C | C | **BROKE** (iterative deep work) or **C.R.I.S.P.E** (insight-driven) |

**Tie-breaking rules:**
- When in doubt between two frameworks, prefer the one with fewer parameters (easier to fill)
- For creative/insight-driven tasks → favor C.R.I.S.P.E over BROKE
- For data/structured tasks → favor ICIO over TRACE
- For general-purpose → default to RACE

### Phase 4 — Prompt Generation

1. Load the selected framework template from `references/frameworks/<framework>.md`
2. Populate each field using:
   - The user's original natural-language input
   - Answers from Phase 2 questions
   - Reasonable defaults inferred from context
3. If a field cannot be filled → leave it as `[请根据实际情况补充]` for the user to complete
4. Output the result in this format:

```
## 🎯 推荐框架：<FRAMEWORK_NAME>（<中文名>）

**为什么选它：** <1-2句话解释>

---

## 📝 优化后的 Prompt

[fully populated prompt here]

---

## 💡 使用提示
- <1-2 actionable tips>
```

### Phase 5 — Follow-up

After presenting the optimized prompt, offer one action:

```
需要调整吗？可以跟我说：
- "太长了，精简一下"
- "加上[某方面]的分析"  
- "换个框架试试"
```

---

## Framework Reference

For full templates, load the relevant file from `references/frameworks/`:

| Framework | File | Best For | Parameter Count |
|-----------|------|----------|-----------------|
| TAC | `tac.md` | Ultra-simple one-liners | 3 |
| APE | `ape.md` | Simple tasks with purpose | 3 |
| RACE | `race.md` | Most general-purpose tasks | 4 |
| ERA | `era.md` | Role-playing tasks | 3 |
| RISE | `rise.md` | Step-by-step execution | 4 |
| ROSES | `roses.md` | Complex problem-solving | 5 |
| COAST | `coast.md` | Scenario-driven tasks | 5 |
| BROKE | `broke.md` | Iterative refinement | 5 |
| TRACE | `trace.md` | Example-driven output | 5 |
| CARE | `care.md` | Result-oriented tasks | 4 |
| ICIO | `icio.md` | Structured I/O & data tasks | 4 |
| C.R.I.S.P.E | `crispe.md` | Deep insight & creativity | 5 |

For detailed multi-scenario examples, see `references/examples.md`.

---

## Important Rules

1. **Never make the user choose a framework directly.** The skill's entire purpose is to eliminate that burden.
2. **Maximum 3 questions.** If after 3 questions the framework is still ambiguous, default to RACE.
3. **Skip questions aggressively.** If Phase 1 extracted a HIGH-confidence signal, skip the corresponding question.
4. **Ask questions one at a time.** Don't overwhelm the user with all questions at once. Use `AskUserQuestion` tool for each.
5. **Output in Chinese.** All user-facing output should be in Chinese (简体中文).
6. **Be concise.** The goal is a usable prompt, not a lecture on Prompt Engineering theory.
