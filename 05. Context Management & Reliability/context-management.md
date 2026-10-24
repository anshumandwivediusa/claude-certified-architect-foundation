# Context Management

**Context** is all the information available to understand something correctly.
Words, requests, and data are ambiguous on their own. Context is what removes the ambiguity.
*Example:* The word "bank" means different things next to "river" and next to "loan." Likewise, "fix it" is meaningless unless you know what "it" refers to.

## 01. "Context", "Context Management" and "Memory" in Claude?

### What "context" means in Claude?
Context is all the information (text, files, images, tool results, prior messages, and Claude’s own output so far) supplied to the model in a given call, up to the context-window limit. It is the only information Claude can use directly, apart from what it learned in training. It is Claude's working desk: only what is on the desk can be used.

It includes:
- System prompt (rules, role, instructions)
- Tool definitions
- The conversation so far (your messages and Claude's replies)
- Tool results (search results, file contents, code output)
- Attached documents or images
- Claude's own output as it is being written

It is **temporary and limited**. It has a token limit (the context window), and Claude does not "know" anything outside it unless it is placed there.

### What "memory" means?
**Memory** is information stored outside the context window that persists across turns or sessions. It is the filing cabinet next to the desk. Examples are saved notes, user preferences, files, and a database.

Memory does nothing until something is **retrieved and placed into context**. Stored facts only affect an answer once they are loaded onto the desk.

| | Context | Memory |
|---|---|---|
| Where it lives | Inside the model's window | Outside (files, store, database) |
| Lifespan | Current task or conversation | Persistent across sessions |
| Size | Limited by token window | Practically unlimited |
| Used by Claude | Directly, always | Only after being loaded into context |
| Analogy | Desk | Filing cabinet |

### Why the definition says "deciding what is in the window at each step"?
Context Management is deciding what is in the window at each step.

Claude is **stateless**: every step starts fresh with only what is in the window. So someone (the app, the developer, or Claude via tools) must decide, at every step:

1. **What to keep** (recent turns, key instructions)
2. **What to compress** (summarize old history)
3. **What to remove** (stale tool outputs)
4. **What to store** (move durable facts to memory)
5. **What to load** (retrieve only the relevant memory or documents)

### Why this keeps Claude "accurate and efficient"
- **Accurate:** The right facts are present, and irrelevant clutter does not distract the model or push important details out of attention.
- **Efficient:** Fewer tokens mean lower cost and faster responses.
- **Long tasks:** Without management, the window fills up, early details are lost or diluted, and quality drops.

### Example
In a long coding session, Claude reads 50 files. Good context management keeps only the 3 relevant ones in the window, summarizes earlier decisions, and saves "project uses TypeScript, tests in /tests" to memory, to be loaded again in the next session.


***Context is what Claude sees now; memory is what is saved for later; context management decides what moves between them.***


## 02. **Core Concepts**
- **Context window:** The maximum tokens Claude can "see" at once. It covers the system prompt, tool definitions, conversation history, tool results, attachments, and the model's own output.
- **Statelessness:** The model keeps nothing between calls. The full relevant context must be resent each turn (or supplied by the app).
- **Context rot / dilution:** As context grows, the signal-to-noise ratio drops. Cost and latency rise, and attention to important details can weaken. More tokens is not always better.
- **Goal:** The smallest set of high-signal tokens that lets Claude do the task well.

## 03. **Context Windows**

The **context window** is the maximum amount of text (measured in **tokens**) that Claude can process in a single call. It is the size limit of the "desk" we discussed earlier.

Context = *what is in the window*. Context window = *how big the window is*.

### What counts toward the limit
Everything in a call shares one budget:
- System prompt
- Tool definitions
- Conversation history
- Tool results and attached files or images
- Claude's own output (the reply being generated)

**Input + output together must fit inside the window.**

### Tokens
- A token is a chunk of text: roughly a word fragment, about 3-4 characters of English, or about 0.75 of a word.
- Limits, pricing, and speed are all measured in tokens, not words.

### Key properties
- **Fixed upper limit:** The window size differs by model and version. When it is full, the app must truncate, summarize, or clear content, or the call fails or stops.
- **Not memory:** The window resets every call. The app resends the history each time, and Claude does not retain it between calls.
- **Bigger is not automatically better:** Longer context means higher cost and latency, and quality can drop as details get diluted (context rot).
- **Not the same as training knowledge:** Knowledge in the model's weights does not use window space.

### Window vs. context vs. memory
| Term | Meaning |
|---|---|
| **Context window** | The size limit (capacity) |
| **Context** | The actual content inside it right now |
| **Memory** | Information stored outside, loaded in when needed |

### Why it matters
Because the window is finite, **context management** is needed: compaction, pruning, retrieval, external memory, and subagents all exist to fit the right information into a limited window.

**One-liner:** *The context window is the token budget for one call; context management is how you spend it wisely.*

Exact window sizes vary by model, so check Anthropic's docs for current numbers if your exam needs them.

## 4. Context Window Internals

<img width="937" height="630" alt="image" src="https://github.com/user-attachments/assets/00502421-6b0f-4280-8994-d9cd8f1e6ff8" />


Claude Code's context window holds 200k tokens by default, and your messages, Claude's replies, and everything Claude reads or runs all draw from that one budget.

- **Fresh each session.** A new session starts with an empty window. Nothing carries over automatically, so durable facts belong in files such as CLAUDE.md.
- **Input and output share the budget.** Claude's replies, tool calls, and tool results usually consume far more tokens than your prompts.
- **Context vs. window.** Context is what sits in the window right now. The window is the size limit.
- **200k is the default, not a fixed rule.** Newer models can run a 1M window in Claude Code, so say "200k by default" unless a question fixes the model.

### Anatomy of the 200k window

About 20k of the window is spent before you type anything, and the growing layers (history and tool results) fill most of the rest.

| Layer | Approx. size | Behavior | What it is |
| --- | --- | --- | --- |
| System prompt | \~6k | Fixed | Built-in instructions that define how Claude Code behaves. |
| Tool schemas | \~8k | Fixed | Definitions of built-in tools (read, edit, search, shell). |
| CLAUDE.md | \~1k (illustrative) | Persistent | Your project instructions, re-read every session. |
| Skills + MCP | \~5k (illustrative) | Loads at startup | Skill descriptions and connected MCP tools. Varies widely. |
| Conversation history | \~45% of window in long sessions | Grows | Your messages and Claude's replies, resent every call. |
| Tool results | Grows with each action | Grows | File reads, bash output, grep matches. |
| Free space | Whatever remains | Shrinks | Room for new work before compaction. |
| Autocompact buffer | \~33k | Reserved | Held back so compaction can run. Not usable for work. |

### How the window fills

In long sessions, conversation history reaches about 45% of the window, and Claude's replies are the main cause, not your prompts.

- **Fixed overhead comes first.** The system prompt, tool schemas, CLAUDE.md, and startup skills and MCP load before your first message.
- **History grows every turn.** The full conversation is resent on each call, so every reply keeps costing tokens for the rest of the session.
- **Tool results are the sneakiest bloat.** One file read, bash command, or grep can add thousands of tokens that stay in context afterward.
- **Free space is the only part you spend.** Every layer above competes for it.

### The autocompact buffer

About 33k tokens are reserved and unusable, so a 200k window gives roughly 167k of working room.

- **Why it exists.** Compaction needs space to run and write its summary, so that room is held back in advance.
- **Effect.** Compaction triggers early, at about 167k used, not at the full 200k.
- **Arithmetic.** 200k window − \~33k buffer = \~167k before autocompact fires.

### How to manage each layer

You can't shrink the fixed layers, so manage the growing ones and keep what you add at startup lean.

| Layer | Lever |
| --- | --- |
| System prompt, tool schemas | Fixed. Treat as rent paid every session. |
| CLAUDE.md | Keep it short and high-signal, since it taxes every turn. |
| Skills + MCP | Disconnect servers you don't need. One heavy server can cost tens of thousands of tokens. |
| Conversation history | Use /compact to summarize, /clear between unrelated tasks, and ask for concise answers. |
| Tool results | Read targeted line ranges, limit search output, and delegate noisy exploration to subagents. |
| Whole session | Start a fresh session for a new task, since each one begins with a clean window. |


### Exam takeaways

A "200k" window is not fully usable, and Claude's side of the conversation fills it faster than yours does.

1. Fixed overhead (\~20k) and the autocompact buffer (\~33k) come off the top.
2. Growth comes from Claude's replies and tool results, not mostly from your typing.
3. The levers are /compact, /clear, targeted tool use, subagents, and a lean CLAUDE.md and MCP setup.
4. A window is not fully usable: fixed overhead and the buffer come off the top.
5. Growth comes from Claude's output and tool results, not mostly from your typing.
6. The levers are `/compact`, `/clear`, targeted tool use, subagents, and a lean CLAUDE.md and MCP setup.

## 05. Using the context levers in Claude Code

### 1. `/compact`: summarize before it gets full
Run it manually at a natural checkpoint instead of waiting for autocompact (~167k):

```
/compact Keep the decisions, the file list, and the failing test names. Drop exploration noise.
```

The text after `/compact` steers what the summary keeps. Compact when a sub-task is done, not mid-debugging, because compaction is lossy.

### 2. `/clear`: reset between unrelated tasks
```
/clear
```
Use it when you switch tasks (for example, from a bug fix to a new feature). Old history only adds noise and cost. Anything worth keeping should already be in CLAUDE.md or committed code.

### 3. Check where the tokens are going
```
/context
```
This shows how much of the window each layer uses (system prompt, tools, MCP, history), so you can see which lever to pull. I'm going from memory on the exact output, so check it in your version.

### 4. Targeted tool use: stop bloat at the source
Steer Claude in your prompt:

```
Read only lines 120-200 of src/auth.ts, not the whole file.
```
```
Grep for "refreshToken" and show at most 20 matches.
```
```
Run the tests but only show the failing ones.
```

Habits that help:
- Ask for specific line ranges and capped search output.
- Pipe noisy commands through `head`, `tail`, or `grep`.
- Avoid "read the whole repo". One broad exploration can burn tens of thousands of tokens.

### 5. Subagents: keep noisy work in a separate window
A subagent has its own context and returns only a summary. Define one as a markdown file, for example `.claude/agents/code-searcher.md`:

```markdown
---
name: code-searcher
description: Searches the codebase and reports findings concisely. Use for broad exploration.
tools: Read, Grep, Glob
---
Search the codebase for what the task asks. Return only file paths,
line numbers, and a 5-line summary. Do not paste large code blocks.
```

Then ask in your prompt:
```
Use the code-searcher subagent to find everywhere we validate JWTs.
```
The main window gets a short answer instead of 40 file reads.

### 6. Lean CLAUDE.md
It loads on every session and every call, so keep only what Claude can't infer:

```markdown
# Project
- Stack: TypeScript, Node 20, pnpm
- Run tests: `pnpm test` (single file: `pnpm test path/to/file`)
- Never edit `/generated`
- Style: follow existing patterns in `src/`
```

Rules of thumb:
- Commands, conventions, and hard constraints only.
- No long prose, no pasted docs, no things the code already shows.
- Link to a doc file instead of inlining it, and let Claude read it only when needed.

### 7. Lean MCP setup
- Run `/mcp` to see what's connected, and disconnect servers you aren't using this session.
- One heavy server can cost tens of thousands of tokens at startup.
- Prefer a few focused servers over many broad ones.

### 8. Session hygiene for long work
- One task per session, then `/clear` or start fresh.
- To pick up later, let Claude write a short progress note (for example `NOTES.md`) and load only that next time. This is external memory in practice.

### Quick decision table

| Symptom | Lever |
| --- | --- |
| Session getting long, same task continues | `/compact` with focus instructions |
| Switching to unrelated work | `/clear` |
| Don't know what's eating space | `/context` |
| Huge file or search output | Ask for ranges and caps |
| Broad "find everything" exploration | Subagent |
| High cost before you type anything | Trim CLAUDE.md, disconnect MCP servers |

Command names and options change between Claude Code versions, so confirm details with `/help` or the Claude Code docs. If you meant the **API or Agent SDK** instead of the CLI, the equivalents are context editing, the memory tool, prompt caching, and subagents, and I can map each lever to those.

Perfect — below is a polished section you can add directly to your notes.

## 06. Production Reliability Patterns for Agentic Systems

Context management is not only about fitting tokens into a window. In production, reliability means the system keeps working even when tools fail, inputs are noisy, tasks are interrupted, or an agent runs long enough to hit edge cases. Claude Architect-level reasoning includes both "good context" and "good operational behavior."

### 1. Retries

Retries are how a system responds to transient failure.

What counts as transient:
- Network hiccups
- API rate limits
- Temporary service outages
- Timeout during a tool call

Good retry patterns:
- Retry only on transient failures, not on permanent ones
- Use exponential backoff and jitter
- Cap retry attempts to avoid infinite loops
- Record the failure and whether it was retried

Example:
- A tool call to a remote API times out once. Retry after a short delay, but if the same request fails repeatedly, stop and surface a clear error.

Exam framing:
- Retries improve resilience but must be bounded and careful. Blind retry loops can amplify costs and create duplicate actions.

### 2. Idempotency

Idempotency means repeating the same action does not create a different result.

Why it matters:
- A tool may be retried after a timeout even though it actually succeeded
- The same instruction may be executed more than once
- Long-running agents may re-trigger side effects

Example patterns:
- Use a stable request ID or operation ID for writes
- "Create if absent" semantics instead of "create always"
- Use upsert patterns for database operations
- Make delete/update operations safe to repeat

Example:
- A payment or file creation action should not create two copies if the request is retried after a connection failure.

Exam framing:
- Context management reduces confusion, but idempotency makes systems safe under retries and partial failures.

### 3. Failure Boundaries

Failure boundaries are places where one failure is isolated and does not cascade across the whole system.

Why it matters:
- One bad tool or one broken subtask should not crash the entire workflow
- Agent systems are composed of many moving parts: LLM calls, tools, storage, APIs, file edits

Good patterns:
- Isolate tool execution from core reasoning
- Treat a failed step as a recoverable state, not a fatal system crash
- Fail a subtask with a clear status instead of poisoning the main task
- Keep a "last known good state"

Example:
- A search tool fails to read a file. The agent should mark that one investigation as failed and continue with alternative retrieval paths, rather than stopping the whole job.

Exam framing:
- The system should fail locally and gracefully, not globally and catastrophically.

### 4. Observability

Observability is the ability to answer: what happened, why it happened, and what state the system was in?

Core signals:
- Logs
- Structured events
- Trace IDs and request IDs
- Metrics
- Status summaries
- Tool call outcomes
- Prompt and token usage

Why it matters:
- In agents, errors can be hidden inside long chains of reasoning and tool interactions
- Without observability, debugging is guesswork

Useful fields to record:
- task ID
- step number
- tool name
- inputs and outputs
- success/failure
- duration
- relevant context summary
- final result or error

Exam framing:
- An AI system without observability is not production-ready. You need to inspect state and trace decisions.

### 5. Task Isolation

Task isolation means separating work into independent units so that one task does not corrupt another.

Why it matters:
- Long agent sessions accumulate state
- Old context can leak into unrelated tasks
- One task's side effects may affect another

Patterns:
- One task = one session or one bounded work unit
- Separate scratch work from durable work
- Clear or reset state when switching goals
- Keep write operations scoped to specific artifacts
- Avoid shared mutable state unless explicitly designed

Example:
- A bug fix task and a feature branch task should not share the same working memory without clear boundaries.

Exam framing:
- Isolation reduces cross-contamination and keeps reasoning focused. It is often a practical complement to context compaction and /clear workflows.

### 6. Restart Recovery

Restart recovery is the system’s ability to continue after interruption, crash, or shutdown.

Why it matters:
- Agentic workflows can be interrupted at any step
- LLM sessions may reset
- Tools may fail halfway through an operation

Recovery patterns:
- Persist the task state and key artifacts
- Save checkpoints of progress
- Record what was completed and what remains
- Resume from a durable checkpoint, not from memory alone
- Keep operations atomic where possible

Example:
- A workflow writes a partial file or database mutation, then crashes. Recovery should know whether the mutation succeeded, whether it must be rolled back, and what step should run next.

Exam framing:
- Durable state plus checkpointing turns an interruptible workflow into a recoverable one.

### Reliability Pattern Summary

The strongest production systems combine:
- Good context management
- Safe retries
- Idempotent actions
- Isolated failure domains
- Strong observability
- Clear task boundaries
- Restartable state

In short:
- Context keeps the agent aligned
- Reliability keeps the system alive
- Recovery keeps the work trustworthy

### Exam Takeaways

1. A production-grade agent is not just a model with tools; it is a system with bounded failure modes.
2. Retries without idempotency are risky.
3. Failure boundaries prevent one bad tool from collapsing the whole workflow.
4. Observability is essential for debugging and trust.
5. Task isolation prevents cross-task pollution and memory contamination.
6. Restart recovery is what makes long-running work survivable.

These are the operational principles that make context management effective in real deployments, not just in a single conversation window.

If you want, I can also give you:
- a shorter version to paste into your existing markdown
- a one-page “high-yield” cheat sheet
- 10 likely exam questions based on these concepts


## 07. **Main Techniques**
1. **Compaction / summarization:** Condense old history into a summary and continue with that. It frees space but is lossy.
2. **Context editing / pruning:** Drop stale items, such as old tool results or irrelevant turns, while keeping recent and important ones.
3. **External memory:** Store durable facts outside the window (files, notes, a memory tool, a database) and retrieve them when needed. This gives persistence across sessions.
4. **Retrieval (RAG) / just-in-time loading:** Load documents or data only when relevant instead of preloading everything.
5. **Prompt caching:** Reuse a stable prompt prefix to cut cost and latency. Stable content goes first and changing content last.
6. **Subagents:** Delegate subtasks to separate context windows and return only a short summary, which keeps the main context clean.
7. **Structured prompts:** Put key instructions in a clear system prompt and long documents in labeled sections. Use concise tool outputs.

**Key trade-offs**
- Keeping more context improves continuity but costs more tokens and risks noise.
- Compressing saves tokens but can lose details.
- Memory persists information but needs good retrieval and upkeep.

Context management is curating, compressing, and externalizing information so Claude always has the right context, not the most context.

## 08. Context Management & Reliability — Scenario-Based Notes

### 1) Session is filling up and Claude is near the context limit

What to do:
- Compact old, low-value exploration
- Keep only:
  - the active goal
  - current constraints
  - key files or decisions
  - active failing tests or blockers
- Drop:
  - dead-end searches
  - stale tool output
  - repeated failed attempts
  - irrelevant conversation churn

Trap:
- “Compact everything” is wrong. This loses the current task state and constraints.
- “Keep every search result” is wrong. This creates context rot.

Reason:
- Compaction should preserve intent and continuing work, not every raw historical detail.



### 2) The user asks for a new feature, but the previous session was debugging a bug

What to do:
- Use /clear or start a fresh session if the tasks are unrelated
- Keep only durable project facts in memory or notes
- Do not carry stale bug-fix context unless it is still relevant

Trap:
- “Keep all previous context just in case” is wrong.
- This creates cross-task contamination and noise.

Reason:
- Unrelated tasks need isolation and a clean context boundary.



### 3) Tool outputs are inflating the context window

What to do:
- Ask for narrow reads, not whole files
- Limit grep output
- Read exact ranges only
- Avoid broad repo exploration
- Use subagents for noisy exploration
- Compact stale tool results

Trap:
- “Read more to be safe” is wrong.
- More context often hides the real signal.
- “Just ask the model to be smarter” is wrong; the issue is context volume, not intelligence.

Reason:
- Targeted retrieval is better than broad retrieval.



### 4) We need persistent facts across sessions

What to do:
- Store them outside the live window:
  - CLAUDE.md
  - docs
  - notes
  - memory store
  - retrieval database
- Load only what is needed when the task starts

Trap:
- “Keep it in chat history” is wrong.
- “It will be in the model’s memory” is wrong.

Reason:
- Memory is external. Context is temporary.



### 5) A tool call times out, but the action may have succeeded anyway

What to do:
- Treat retry as a reliability issue
- Use idempotent patterns:
  - stable operation IDs
  - check whether the result already exists
  - create-if-absent semantics
  - safe upsert behavior
  - avoid duplicate writes

Trap:
- “Retry the same request immediately because it probably failed” is wrong.
- This can create duplicate side effects.

Reason:
- Retries are safe only when the operation is idempotent or verified.



### 6) One failing tool crashes the whole workflow

What to do:
- Isolate the failure
- Record the error
- Fall back or continue with an alternate path
- Stop only that subtask, not the whole workflow

Trap:
- “If one tool fails, the whole pipeline is dead” is wrong.

Reason:
- Good system design creates failure boundaries and graceful degradation.



### 7) A long task gets interrupted halfway through

What to do:
- Save progress checkpoints
- Persist task state
- Record completed steps and remaining work
- Resume from durable state, not live memory alone

Trap:
- “Just restart and hope the model remembers” is wrong.
- Stateless systems do not preserve workflow state unless it is stored.

Reason:
- Restart recovery requires durable checkpoints.



### 8) The agent has done many steps, but nobody can explain what happened

What to do:
- Add observability:
  - logs
  - tool call traces
  - request IDs
  - task IDs
  - step markers
  - success/failure status
  - timing
- Trace state through each step

Trap:
- “The model said it did it, so it must be fine” is wrong.

Reason:
- Without observability, debugging is guesswork.



### 9) A broad search is returning too much noise

What to do:
- Narrow search terms
- Restrict match count
- Read exact files or ranges only
- Delegate to a subagent if the task is exploratory

Trap:
- “Let’s read everything and sort it later” is wrong.
- This burns tokens and hides the signal.

Reason:
- Search and read should be targeted, not exploratory by default.



### 10) We are reusing facts from a prior task in a new task

What to do:
- Reset context boundaries
- Preserve only durable facts in memory
- Keep only current task-related context live

Trap:
- “The model should know the difference by itself” is wrong.
- Models do not inherently separate tasks unless the application enforces boundaries.

Reason:
- Task isolation is an engineering control, not a model feature.



### 11) The agent keeps re-running the same failed command

What to do:
- Stop blind retries
- Inspect whether the failure is transient or permanent
- Add retry policy with backoff and cap
- Check whether the side effect already happened

Trap:
- “Retry endlessly until it works” is wrong.
- This creates duplicate effects and wastes time.

Reason:
- Retry should be disciplined and safe.



### 12) A long-running workflow is expensive, slow, and full of stale details

What to do:
- Compact history
- Remove stale tool results
- Keep only the current task state
- Rely on external memory for durable facts
- Use a subagent for noisy exploratory work

Trap:
- “Store more in the live context because it might help” is wrong.
- This increases cost and weakens attention.

Reason:
- The correct pattern is smaller, better-targeted context.



### High-Yield Pattern Map

- Window pressure → compact, prune, or clear
- Persistence across sessions → external memory
- Noisy tool output → narrow reads, caps, subagents
- Retry failures → idempotency + bounded retries
- One failing step → failure boundary
- Long-running task → checkpoints and restart recovery
- Unclear behavior → observability


