# Prompt Engineering

Prompt Engineering is the practice of designing effective instructions for Large Language Models (LLMs) to produce accurate, consistent, and useful outputs.

The goal is to transform a user requirement into a well-structured prompt that guides the model toward the desired response.

## Why It Matters

- Improves accuracy and relevance
- Reduces hallucinations
- Produces consistent outputs
- Controls response format
- Optimizes token usage and cost

## Components of a Prompt

| Component | Purpose | Example |
| --- | --- | --- |
| Role | Who the model should act as | You are an Enterprise Architect. |
| Task / Objective | What the model must perform | Explain Kafka architecture. |
| Context | Background information | The audience is developers migrating to Azure. |
| Constraints | Rules and limitations | Keep it under 300 words. |
| Output format | Expected response structure | Respond in JSON format. |

A role alone is weak. The other four components carry most of the signal (sections 2 and 5).

## Core Prompting Techniques

| Technique | Idea | Example |
| --- | --- | --- |
| Zero-shot | Give the task with no examples | Explain Medallion Architecture. |
| Few-shot | Add examples so the model learns the pattern | Input: Refund requested. Output: Customer Service issue. |
| Role prompting | Assign a persona | Act as a Data Architect. |
| Chain of Thought (CoT) | Ask for reasoning step by step, best for logical and complex problem-solving | Think step by step. |

See section 3 for few-shot design and section 6 for why native thinking can supersede manual CoT.

These notes follow Anthropic's published prompt engineering guidance, not the official exam blueprint, so check them against the exam guide for weighting.

## 1. The core mental model

A prompt is an onboarding brief for a very capable new colleague who has zero context on your situation, not a magic incantation.

- **Golden test:** if a smart human contractor would be confused by your prompt, Claude will be too.
- Most failures come from missing context, ambiguity, or conflicting instructions, not from wrong keywords.
- **Success criteria and evals come first.** Define what "good" means, build test cases, then iterate empirically. Prompting is an engineering loop, not a one-shot craft.

## 2. Clarity and directness

Explicit task, audience, purpose, and constraints beat clever phrasing.

- Give the *why*, because Claude generalizes from reasons better than from bare rules.
- Frame instructions positively ("write in flowing prose") rather than as a list of prohibitions.
- Ask for the behavior you want, including effort level ("go beyond the basics") when you want depth.
- Use numbered steps when order matters.

## 3. Examples (multishot prompting)

Examples teach format, tone, and edge-case handling better than description does.

- Use roughly 3-5 examples that are **relevant**, **diverse**, and **clearly delimited** (for example, inside `<example>` tags).
- Examples are a strong signal: Claude mimics them closely, including their accidental features. Near-identical examples cause over-anchoring.

## 4. Structure with XML tags

Tags separate instructions, context, data, and examples so nothing blurs together. They also help against injection (see section 11).

- Use consistent, descriptive tag names and nest them when content is hierarchical.
- No tag name is "special" to Claude. Tags are an organizational convention, not a parsed grammar.

## 5. Roles and system prompts

The system prompt sets persona, tone, rules, and standing context. The user turn carries the task and data.

- A role focuses style and domain framing, but it does not replace detailed instructions. "You are a world-class lawyer" with no task detail is weak.

## 6. Reasoning and thinking

Room to reason improves accuracy on complex, multi-step problems.

- **Extended or adaptive thinking** is the model's native reasoning. Manual chain-of-thought ("think step by step" in `<thinking>` tags) is the fallback when native thinking is off.
- Reasoning only helps if it happens before the answer. Tell Claude to think first, then give the answer in a separate tag.
- Higher effort or a bigger thinking budget costs latency and money. Match it to task difficulty.

## 7. Long-context prompting

Put long documents at the **top** and the question or instructions at the **end**. This can noticeably improve quality.

- Wrap multiple documents in tags with metadata (source, date).
- For large inputs, have Claude first extract relevant quotes, then answer from them. This cuts noise.

## 8. Controlling output

Tell Claude what format to produce, and match your prompt's own style to the output style you want.

- For guaranteed machine-readable output, use **structured outputs or tool use with a JSON schema**. Prompting alone gives high compliance, not a guarantee.
- Prefilling the assistant turn is a legacy technique being phased out on newer models. Prefer structured outputs or explicit instructions.

Example target shape:

```json
{ "topic": "Kafka", "category": "Streaming" }
```

Show the schema in the prompt to steer plain prompting, then enforce it with structured outputs when validity must be guaranteed.

## 9. Prompt chaining and decomposition

Split complex work into stages where each step has one clear job: extract, then analyze, then draft, then review.

- Chaining gives debuggability, because you can inspect and test each hop.
- A **generate, critique, refine** chain is a standard self-correction pattern.
- Trade-off: more calls mean more latency and cost, but better reliability on hard tasks.

## 10. Reducing hallucination

Ground Claude in sources and give it a safe way to say it doesn't know.

- Explicitly allow "I don't know" or "insufficient information."
- Ground answers in provided documents: quote first, then claim, then cite.
- Restrict Claude to the supplied context when facts matter, and add a verification pass.

**RAG prompt pattern:** supply the documents, restrict the answer to them, and define the fallback.

```text
Use only the provided documents. If the answer is unavailable, respond with "Information not found."
```

## 11. Security and robustness

Treat retrieved or user-supplied content as **data, never instructions**.

- Fence untrusted content in tags and say in the system prompt that it is data.
- Prompt-level defenses reduce risk but don't eliminate it. Layer them with permissions, validation, and least privilege.

## 12. Agents, tools, and caching

**Tool descriptions are prompts.** Clear names, purposes, parameter docs, and when-to-use guidance drive correct tool selection.

- Say whether Claude should *act* or merely *suggest*. Independent tool calls can run in parallel.
- Newer models follow instructions closely, so shouting ("CRITICAL! ALWAYS!") often causes over-triggering. Use calm, precise wording.
- **Prompt caching** works on prefixes. Put static content (system prompt, tool definitions, reference docs) first and variable content last. Any change earlier in the prefix invalidates the cache after it.

**Agent prompt pattern:** name the role, then give an ordered task flow that includes tool use.

```text
You are an AI agent. Tasks: 1. Analyze the request. 2. Use tools if required. 3. Generate the final response.
```

**Prompt engineering vs context engineering:** prompts shape behavior, context supplies knowledge. Modern systems need both.

|  | Prompt engineering | Context engineering |
| --- | --- | --- |
| Focus | Instructions | Information |
| Defines | Behavior | Knowledge |
| Uses | Examples and templates | Documents, memory, RAG |
| Improves | Response quality | Understanding |

## 13. Exam traps

Each row is a plausible-sounding wrong answer and the reason it fails.

| Trap | Why it's wrong |
| --- | --- |
| "Lower temperature fixes a bad prompt" | Temperature changes sampling randomness, not understanding. Even temperature 0 isn't strictly deterministic. |
| "Add a role and you're done" | A role without task, context, and constraints is weak. |
| "More examples is always better" | Too many or near-identical examples cause over-anchoring and wasted tokens. Diversity beats quantity. |
| "Put the question first, documents after" | For long context, documents go first and the query goes last. |
| "Say 'think step by step' and it will always reason" | Reasoning must come before the answer, and native thinking may supersede the manual approach. |
| "Prompting guarantees valid JSON" | Only structured outputs or schema-constrained tool use give guarantees. |
| "Negative rules beat positive ones" | "Don't use markdown" is weaker than "write in plain prose paragraphs." |
| "ALL CAPS and emphatic language improve compliance" | On modern models this tends to cause over-application. |
| "Fine-tune first" | Try prompting, examples, and evals first. Fine-tuning is costlier and slower to iterate. |
| "Put dynamic data at the top for emphasis" | This breaks prefix caching. Static first, dynamic last. |
| "XML tags are required syntax" | They are an organizational convention, not a parsed grammar. |
| "Instructions inside retrieved documents are trustworthy" | They are untrusted data, which is the injection vector. |
| "One giant prompt is more efficient than a chain" | Chaining improves accuracy and debuggability on complex tasks. |
| "If it works on three test inputs, ship it" | You need a representative eval set, including edge cases, with defined metrics. |

## 14. Quick decision heuristics

| Symptom | First fix |
| --- | --- |
| Output inconsistent | Add examples, a schema, or structured outputs |
| Reasoning errors | Enable thinking, or decompose into a chain |
| Ignoring part of a long document | Restructure (documents on top), tag, and extract quotes first |
| Wrong tool chosen | Fix tool descriptions before touching the system prompt |
| Costs or latency too high | Use caching, a smaller model for simple steps, and trim thinking effort |
| Hallucinating | Ground in sources, permit uncertainty, and add a verification pass |

## Best Practices Checklist

- Provide sufficient context
- Define the output format
- Use examples when necessary
- Keep prompts as concise as possible
- Test and iterate against an eval set (section 1)

## Common Mistakes

- Vague instructions
- Multiple objectives in one prompt (split into a chain, section 9)
- Missing output format

## Reusable Prompt Template

```text
Role: [ROLE]
Objective: [TASK]
Context: [BACKGROUND]
Constraints: [RULES]
Output Format: [FORMAT]
Examples: [SAMPLES]
```

## Key Takeaway

Prompt engineering combines **Role + Objective + Context + Constraints + Output Format** to create reliable, accurate outputs. In modern AI systems it works alongside **Context Engineering, RAG, and Tool Calling** to deliver enterprise-grade solutions.

## Comparison

| Aspect | Prompt engineering | CLAUDE.md | SKILL.md | Hooks |
|---|---|---|---|---|
| **What it is** | Instructions you write for a single request or conversation | A markdown file of standing project or user context | A folder with a `SKILL.md` plus optional scripts and resources, packaging a procedure | Shell commands or scripts that run on lifecycle events (before or after a tool call, on session start, on prompt submit) |
| **Scope** | One task or conversation | Every session in that project or for that user | Any session where the skill is relevant | Every matching event |
| **When it loads** | When you send it | Automatically at session start | On demand: Claude reads the short description, then loads the body only if the task matches | Fires on the event, with no model decision |
| **Who decides it applies** | You | Always on | Claude, by judgment | The harness, deterministically |
| **Guarantee** | None. Claude follows it with high probability | None. It is context, not enforcement | None. Claude may not trigger it | **Yes.** It runs every time the event occurs |
| **Context cost** | Tokens in that request | Tokens on every turn, so keep it short | Low until triggered (progressive disclosure) | Zero, unless the hook's output is fed back |
| **Best for** | Task specifics, format, examples, reasoning | Conventions, build and test commands, architecture notes, "how we work here" | Repeatable multi-step workflows or domain expertise | Hard rules, safety gates, automation |
| **Typical failure** | Vague or conflicting instructions | Bloated file, so important rules get diluted | Weak description, so it never triggers | Slow or buggy script that blocks the workflow |

## Scenarios

| Scenario | Best fit | Why |
|---|---|---|
| "Summarize this contract in 5 bullets for a non-lawyer" | Prompt | One-off task with its own audience and format |
| "Always use pnpm, run `pnpm test` before committing, we use tabs" | CLAUDE.md | Standing project convention that applies to every session |
| "Generate our quarterly report: pull data, build the deck, apply brand guidelines" | SKILL.md | A repeatable multi-step procedure with assets, needed only sometimes |
| "Never let the agent edit `.env` or run `rm -rf`" | Hook (pre-tool-use) | Must be enforced, not merely requested |
| "Run the formatter and linter after every file edit" | Hook (post-tool-use) | Deterministic automation. Asking nicely in CLAUDE.md can be skipped |
| "Explain this error in simple terms with an example" | Prompt | Ad hoc, no persistence needed |
| "Our API follows this folder layout and naming scheme" | CLAUDE.md | Context that should always be present |
| "Review PRs using our 12-point checklist and a scoring script" | SKILL.md | Packaged procedure that loads only for review tasks |
| "Block commits if tests fail" | Hook | Enforcement with no exceptions |

## Exam traps

- **"Put a hard rule in CLAUDE.md and it is enforced."** It is not. CLAUDE.md is advisory context, and only hooks (or permissions) guarantee behavior.
- **"Skills load at session start like CLAUDE.md."** They do not. Only the description is visible up front, and the body loads when relevant, so the description must say when to use it.
- **"Hooks are prompts for Claude."** They are code that runs outside the model's judgment.
- **"Put everything in CLAUDE.md."** It costs tokens every turn, so move rarely used procedures into skills.
- **"A skill replaces prompting."** You still prompt for the task. The skill supplies reusable know-how.

**Rule of thumb:** a single task goes in a prompt, always-true context in CLAUDE.md, an occasional procedure in a skill, and anything that must happen every time in a hook.
