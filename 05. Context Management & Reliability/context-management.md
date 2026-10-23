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

### Caveats

Only four of the numbers come from the source slide; the rest are my estimates, and real sessions vary.

- **From the slide:** system prompt \~6k, tool schemas \~8k, history \~45% in long sessions, autocompact buffer \~33k.
- **My estimates:** CLAUDE.md (\~1k) and skills + MCP (\~5k) are illustrative. Heavy MCP setups can cost far more.
- **Layout:** the slide draws free space and the buffer in a different order. Here the buffer sits at the top of the window, where it is reserved.

### Exam takeaways

A "200k" window is not fully usable, and Claude's side of the conversation fills it faster than yours does.

1. Fixed overhead (\~20k) and the autocompact buffer (\~33k) come off the top.
2. Growth comes from Claude's replies and tool results, not mostly from your typing.
3. The levers are /compact, /clear, targeted tool use, subagents, and a lean CLAUDE.md and MCP setup.

**One-liner:** The window is a shared token budget, so the goal is the right tokens in it, not the most.




## Exam takeaways
1. A window is not fully usable: fixed overhead and the buffer come off the top.
2. Growth comes from Claude's output and tool results, not mostly from your typing.
3. The levers are `/compact`, `/clear`, targeted tool use, subagents, and a lean CLAUDE.md and MCP setup.

## **Main Techniques**
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

