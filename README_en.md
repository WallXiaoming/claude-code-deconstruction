# Claude Code Deep Analysis: Deconstructing an AI System Through the Lens of Information Entropy

> Generated: 2026-06-30
> Based on complete source code analysis (leaked version 2026-03-31) and capability comparison three months later

---

## I. The Essential Definition of Claude Code

### Core Formula

```
Claude Code =
  Context Background Info (reduces input task entropy)
  + Tool Set (reduces execution process entropy)
  + while(true) loop (continuously reduces entropy until completion)
```

Founder Boris Cherny's original statement from an interview aligns perfectly:

> **"You have a model, you give it tools, you give it some context and a task, and it uses the tools to accomplish the task."**

**All of Claude Code's evolution over 16 months has not changed this core pattern.** The permission system, CLAUDE.md, AgentTool, Skill, MCP, Hook, Compact, Plan Mode, Advisor — all are optimizations around this loop, not a restructuring of it.

### Core Loop in Source Code (`src/query.ts`)

```typescript
while (true) {
  // Send: system prompt + context + history + tool definitions → API
  const response = await callModel({ system, tools, messages })
  
  // Decide: does the output contain a tool_use block?
  if (response.any(block => block.type === "tool_use")) {
    // Model needs more info → execute tool → inject result → continue
    for (const toolCall of response.toolCalls) {
      const result = executeTool(toolCall)
      messages.push(result)
    }
    continue
  } else {
    // Model believes task is complete → output text → end
    displayToUser(response.text)
    break
  }
}
```

---

## II. Deconstruction Through Information Theory

### 2.1 Shannon Entropy Definition

$$H(X) = -\sum_{i=1}^{n} p(x_i) \log_2 p(x_i)$$

**Entropy is the receiver's measure of uncertainty about a message.** The same message has different entropy values for different receivers — "commercial deployment" is high-entropy (5 equally probable interpretations) for a business owner, but low-entropy (one dominant interpretation) for a testing expert.

### 2.2 The System's Essence: An Entropy-Reducing Pipeline

```
User Input (extremely high entropy H ≈ 3-5 bit)
  │
  ├── Pre-context Collection (first layer of entropy reduction)
  │   ├── git status/log → repository state determined
  │   ├── CLAUDE.md → project conventions determined
  │   ├── memdir/ → persistent memory determined
  │   └── env info → runtime environment determined
  │   → Entropy reduced to H ≈ 1.5-3 bit
  │
  ├── while(true) Loop (layer-by-layer entropy reduction until H ≈ 0)
  │   Each iteration:
  │     Model outputs text (communicating with user)
  │        or
  │     tool_use → Execute Tool → Inject deterministic information → Entropy further reduced
  │
  └── Termination Condition: Model no longer outputs tool_use
      → H ≈ 0 (system believes task is complete)
```

### 2.3 Entropy Role of Each Component

| Component | Information Theory Role | Source Location |
|-----------|------------------------|-----------------|
| **System Prompt** | Prior distribution injection (tells the model "who you are, how to behave") | `src/constants/prompts.ts` (600+ lines) |
| **CLAUDE.md** | Reduces conditional entropy H(output \| project knowledge) | `src/utils/claudemd.ts` |
| **memdir/** | Cross-session persistent prior | `src/memdir/` |
| **Tool Schema** | Output space constraint (Zod validation rejects illegal output) | Defined in each tool |
| **Tool Execution** | Turns "facts the model is uncertain about" into "deterministic information" | `src/services/tools/` |
| **Compact** | Preserves high-value information within limited context budget | `src/services/compact/` |
| **Permission** | Introduces human arbitration at critical decision points | `src/hooks/toolPermission/` |
| **Skill** | Methodology injection (tells the model "follow this playbook") | `src/skills/bundled/` |
| **Agent** | Independent entropy-reduction sub-loop (runs in isolated context) | `src/tools/AgentTool/` |
| **Hook** | External deterministic signal injection (script output fed to model) | `src/utils/hooks.ts` |
| **Plan Mode** | Explicitly surface entropy-reduction path before execution | `src/tools/EnterPlanModeTool/` |
| **Advisor** | Introduces stronger model judgment at critical decision points | API-level feature |

---

## III. The Relationship Between Model and Tools

### 3.1 Model Capability is the Foundation, But Not the Core Moat

From the leaked source code, we can confirm: **Claude Code does NOT rely on model fine-tuning to adapt to tools.**

```typescript
// Model invocation is just one line (src/query.ts:659)
for await (const message of deps.callModel({...}))

// The model doesn't know what FileReadTool or BashTool are
// It only knows: there's a tool with this name, its description says it can read files, its schema defines the parameters
// This is the model's native tool-use capability, not the result of fine-tuning
```

The model receives tool definitions (name + description + input_schema) through the API's `tools` parameter and natively understands how to use them. **Tool-use ability is something models learn during base training, not something fine-tuned for Claude Code's 40+ tools.**

### 3.2 Pro vs Flash: Essential Differences

| Dimension | Pro (Flagship) | Flash (Lightweight) | Reason for Gap |
|-----------|---------------|-------------------|----------------|
| HumanEval (function completion) | ~99% | ~97% | **Minimal (deterministic tasks are saturated)** |
| Multi-step reasoning (GPQA Diamond) | 94% | 74% | **Large (fuzzy judgment requires parameter capacity)** |
| Error recovery | Strong (will change approach) | Weak (gives up easily) | Strategy diversity determined by parameter scale |
| Long-document cross-section reasoning | 89% | 79% | Attention maintenance capability gap |

**What Pro buys is not "ability to get work done," but "ability to make better judgments when signals are ambiguous."** On deterministic tasks (HumanEval), the gap between Pro and Flash approaches zero because training signals are clear enough for smaller parameters to converge.

### 3.3 Key Takeaway

> **Flash + good task decomposition + clear methodology ≈ Pro-level results.**

When a task has been sufficiently entropy-reduced, a lightweight model's output quality can approach that of a flagship model. Anthropic's Advisor Strategy, released in April 2026, validates this: Haiku 4.5 + Opus Advisor nearly doubled performance (19.7% → 41.2%) at only 15% of Sonnet's cost.

---

## IV. Programming Capability Evaluation Framework

### 4.1 Evaluation Layers

| Level | Representative Benchmark | What It Measures | Model Saturation |
|-------|------------------------|-----------------|-----------------|
| **L1 Code Generation** | HumanEval / MBPP | Function-level completion | **Saturated (99%+)** |
| **L2 Software Engineering** | SWE-bench Pro / FrontierSWE | Bug fixing, cross-file modification | **Intense competition (60-80%)** |
| **L3 Agent Capability** | Terminal-Bench / MCP-Atlas | Terminal operations, tool invocation | **Rapidly improving** |
| **L4 Ultra Long-Horizon Tasks** | SWE-Marathon | Multi-hour end-to-end development | **Still a large gap (13-26%)** |
| **L5 Aesthetics/Design** | Design Arena / Code Arena | Visual quality of frontend code | **Unsettled** |

### 4.2 The Nature of Evaluation: Verifiable vs. Non-verifiable

**Improvement magnitude = verifiability of the task.**

| Benchmark | Verification Method | GLM-5.1 → 5.2 Improvement |
|-----------|-------------------|---------------------------|
| AIME 2026 (Math) | Deterministic answers | 95.3% → **99.2%** |
| FrontierSWE (with tests) | Tests pass/fail | 30.5 → **74.4** +44 points |
| Terminal-Bench (result verifiable) | Operation result matching | 62 → **81** +19 points |
| CritPt (Critical Thinking) | No objective standard | 4.6 → **20.9** (still low) |

**Deterministic tasks have clear training signals, enabling efficient RL optimization. Open-ended tasks have fuzzy training signals, leading to unstable model performance.** This explains the fundamental reason why LLMs approach perfect scores on deterministic tasks yet struggle with open-ended ones.

---

## V. Current Mainstream Model Capability Comparison (June 2026)

| Dimension | Claude Opus 4.8 | Gemini 3.1 Pro | GLM-5.2 | DeepSeek-V4 Flash |
|-----------|----------------|---------------|---------|-------------------|
| **Agentic Programming** | ✅🥇 | ⚠️ | ✅ | ⚠️ |
| **Abstract Reasoning** (ARC-AGI-2) | 53% | **77%** 🥇 | — | — |
| **Aesthetics/Design** (Design Arena) | 1350 | Not in top 3 | **1360** 🥇 | — |
| **Context Window** | 1M | **2M** 🥇 | 1M | 1M |
| **Tool Call Precision** | ✅🥇 | ⚠️ | ✅ | ✅ |
| **Hallucination Rate** | **Low** 🥇 | Somewhat high | Medium | Medium |
| **Multimodal** | ❌ | ✅🥇 | ❌ | ❌ |
| **Cost / Million tokens** | $15/$75 | $2/$12 | $1.4/$4.4 | **Extremely low** 🥇 |

### 5.1 Optimal Division of Labor

Based on capability distribution, the ideal model分工 is:

```
Project Phase        Best Model        Rationale
─────────────────────────────────────────────────────
Solution Design      Gemini Pro        Knowledge breadth + abstract reasoning + multimodal
Core Coding          Claude Opus       Agentic programming precision + low hallucination
High-frequency Exec  DeepSeek/Flash    Low cost, near-Pro results on deterministic tasks
Quality Audit        Opus/Advisor      Gate-keeping at critical nodes without overspending
Aesthetic Frontend   GLM-5.2           Design Arena #1
```

---

## VI. Fundamental Problems in the Current System

### 6.1 The Flaw of "Model Judges Completion"

```typescript
// src/query.ts:1062-1357
if (!needsFollowUp) {
  // Model didn't call a tool → it believes its task is complete
  return { reason: 'completed' }
}
```

**The criterion is "the model stopped calling tools," not "the task is objectively complete."** These are two entirely different things:

| Model State | Actual Outcome | Problem |
|-------------|---------------|---------|
| "I think I'm done" → no tool calls | Task complete | ✅ |
| "I think I'm done" → no tool calls | Task NOT complete | ❌ Model overconfidence |
| "I don't think I'm done" → calls tools | Task already complete | ❌ Token waste |
| "I don't think I'm done" → calls tools | Task indeed incomplete | ✅ |

**There is no objective "completion criterion."** This judgment relies 100% on the model's internal, unobservable "confidence threshold."

### 6.2 High-Entropy Inputs Depend on Model Guesswork

When a user inputs a high-entropy instruction ("build a Contra game," "commercial deployment"), the system has no structured requirement clarification process. Instead, it lets the model "guess" the user's true intent during reasoning. The stronger the model, the better the guess — but this **masks an architectural problem rather than solving it.**

### 6.3 Imbalanced Distribution of Task Value

```
Value Curve:

  Raw Input (high entropy) → Entropy-reduction Process → Final Output (low entropy)
    Value ≈ 0                  Valuable                    Value ≈ electricity cost

The current system invests most effort in optimizing the "final output"
But real value lies in the "entropy-reduction process" —
  requirement clarification, architecture design, methodology selection
```

---

## VII. Future Evolution Directions

### 7.1 From "One Big Loop" to "Network of Loops"

```text
Now: One model does everything
     → Context gets crammed
     → Both writes code AND does architecture AND tests
     → One thing gets stuck, everything stops

Future: Main loop decomposes tasks + dispatches to specialized sub-loops
     ├── Architecture Loop (Gemini/Opus)
     ├── Implementation Loop (Sonnet/DeepSeek)
     ├── Testing Loop (Haiku)
     └── Audit Loop (Opus)
     Each loop runs independently, verifies independently, uses its best-suited model
```

The `COORDINATOR_MODE`, `AgentTool`, and `WorkflowTool` in the source code are already embryonic forms of this direction.

### 7.2 From "Model as Sole Intelligence" to "Intelligence Distributed Across Layers"

```text
Now: Model is the brain, tools are the hands and feet
     The model must decide at every step

Future: Each tool has an embedded small model
     Tool → Expert (has methodology, can make independent decisions)
     Main Model → Coordinator (only responsible for dispatching and acceptance)
```

### 7.3 From "Natural Language Interface" to "Structured Interface"

When a task has high certainty, execute it directly. Only go through the natural language entropy-reduction process when entering a new domain. Reusable methodologies are encapsulated as Skills/Agents, transforming historical experience from "done and forgotten" to "persistent, reusable low-entropy operation tables."

### 7.4 From "Session Memory" to "Knowledge Infrastructure"

```text
Now: Read CLAUDE.md at each session start
     Memory disappears after session ends

Future: Project knowledge graph accumulates continuously
     On-demand querying (not cramming everything into context)
     Model actively updates the knowledge base
```

### 7.5 From "Passive Response" to "Active Stewardship"

Cron Agent on long-term duty: scheduled security scanning, automated API monitoring and diagnosis, automatic Issue prioritization. The `AGENT_TRIGGERS` and `CRON_TOOLS` in the source code are already groundwork for this step.

---

## VIII. A New Concept: Real-Time Entropy Assessment System

### 8.1 Current Gap

Throughout this analysis, we discovered a **systematic gap** — no one is doing "real-time assessment of the system's own uncertainty":

| Existing Tool | What It Measures | Real-time? | Guides Next Step? |
|--------------|-----------------|-----------|-----------------|
| claude-entropy | Token waste, error patterns | ❌ Post-hoc | ❌ |
| claude-hud | Context usage, time spent | ✅ | ❌ Display only, no decision |
| mete | Code structure complexity | On-demand | ❌ |
| **Gap** | **Task information entropy H(t)** | **✅ Needed** | **✅ Recommends entropy-reduction strategy** |

### 8.2 System Design

```
Core components of a real-time entropy assessment system:

① Entropy Estimator
   Input: current context + history + tool execution results
   Output: H(t) = current task uncertainty
         H_dimension[] = remaining uncertainty per dimension

② Entropy-Reduction Strategy Recommendation
   Input: H_dimension[] + strategy efficiency history
   Output: optimal next action (ask user / read file / search code / switch strategy)

③ Termination Condition Determination
   Input: H(t) < threshold
   Output: task completion determination (objective, data-driven, not model feeling)
```

### 8.3 A Valuable Implementation Direction

A plugin/MCP server that monitors conversation state in real-time, calculates information entropy, and recommends the optimal entropy-reduction strategy when stuck. Combined with existing tools (claude-hud), it would cover the complete "monitor-analyze-decide" chain:

- claude-hud handles **display** (already exists, needs expansion)
- Entropy estimator handles **analysis** (gap, needs development)
- Strategy recommender handles **decision** (gap, needs development)

---

## IX. Summary of Core Insights

### 9.1 Value Shift

> **Code generation is approaching the cost of electricity. What's truly valuable is the ability to transform high-entropy requirements into low-entropy instructions.**

As model output costs approach zero, value shifts upstream — requirement validation, architecture design, knowledge accumulation, methodology encapsulation — these entropy-reduction activities are the true scarce resources.

### 9.2 The Nature of Model Capability

> **Pro and Flash are nearly indistinguishable on deterministic tasks. What Pro buys is a prior distribution for "making better judgments under ambiguous signals," not "stronger execution capability."**

This explains why good methodology + lightweight model can approach flagship model results — methodology has already reduced task uncertainty to a sufficiently low level.

### 9.3 The System's Fundamental Contradiction

> **The biggest problem with current AI systems is not that models aren't strong enough, but that the highest-entropy judgment — "what counts as completion" — is delegated to the most unstable component (model reasoning).**

A truly robust system should base completion criteria on externally verifiable facts, not the model's internal confidence threshold.

### 9.4 The Next Phase

> **Before 2024: compare reasoning ability. After 2025: compare context + tools + physical world access. The next phase is "orchestration" — multi-agent collaboration, long-term stewardship, mixed-model orchestration, human-AI hybrid teams.**

Model reasoning capabilities have approached practical thresholds. The next step is systematic capability — not just "being able to output correct answers," but "being able to reliably complete complex tasks."
