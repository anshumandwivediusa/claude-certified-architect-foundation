# Context Management

**Context** is all the information available to understand something correctly.
Words, requests, and data are ambiguous on their own. Context is what removes the ambiguity.
*Example:* The word "bank" means different things next to "river" and next to "loan." Likewise, "fix it" is meaningless unless you know what "it" refers to.

## "Context", "Context Management" and "Memory" in Claude?

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


## **Core Concepts**
- **Context window:** The maximum tokens Claude can "see" at once. It covers the system prompt, tool definitions, conversation history, tool results, attachments, and the model's own output.
- **Statelessness:** The model keeps nothing between calls. The full relevant context must be resent each turn (or supplied by the app).
- **Context rot / dilution:** As context grows, the signal-to-noise ratio drops. Cost and latency rise, and attention to important details can weaken. More tokens is not always better.
- **Goal:** The smallest set of high-signal tokens that lets Claude do the task well.

### **Main Techniques**
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

