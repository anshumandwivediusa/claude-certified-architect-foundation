# Domain 1: Agentic Architecture & Orchestration (27%)

## 01. What is an AI Agent?

An **AI Agent** is an AI system that can **understand a goal, make decisions, use tools, take actions, observe the results, and continue working until the goal is completed** with minimal human intervention.

Unlike a traditional chatbot that simply answers a question, an AI agent is **Autonomous Goal-oriented**. It can plan, execute tasks, and adapt based on what happens during execution.
<p align="center">
<img width="500" height="350" alt="image" src="https://github.com/user-attachments/assets/29e5693b-433a-411b-9a9d-f1df75660aad" />
</p>

### Knowledge Bases: 
From information theory, a knowledge base is essentially a structured encoding of information designed to minimize uncertainty.

Modern KBs often integrate with ML models:
  - Semantic search using embeddings (vector representations of text).
  - Knowledge graphs linking entities and relationships.
  - Contextual retrieval (e.g., RAG — Retrieval-Augmented Generation).

Conceptual Role in AI:
  - Acts as external memory for LLMs.
  - Provides ground truth to reduce hallucinations.
  - Enables reasoning over structured data (graphs, ontologies).


## 02. Learning Objectives

Based on the exam guide and prep sources, here's the breakdown of **Domain 1: Agentic Architecture & Orchestration (27%)** — the highest-weighted domain on the exam:

**1. Foundational agent concepts**
- Distinguishing agents from workflows and conversational systems
- The four pillars of agentic behavior: perception, selection (reasoning), execution, iteration
- How tool use closes the gap between an LLM and the "action environment" — i.e., what turns a model into something that can actually *act* on the world, not just respond to it

**2. The agentic loop lifecycle**
- Sending requests, inspecting `stop_reason` (`tool_use` vs. `end_turn`) to decide the next step
- Executing tools and returning results back into the loop
- Loop control: defining termination conditions, turn budgets, and escalation paths — covering task-complete, error-threshold, budget-exhausted, and explicit stop signals (this is squarely about preventing infinite or runaway loops)

**3. When to use agentic architecture at all**
- Agentic vs. simpler patterns — recognizing when a fixed workflow beats a model-driven loop
- Comparing pre-scripted workflows against model-driven agentic sequences, with emphasis on tool description quality and safety tradeoffs

**4. Task decomposition and execution strategy**
- How a coordinator breaks a complex request into distinct concerns, identifies dependencies, and decides what runs in parallel vs. sequentially
- Recognizing the anti-pattern: skipping decomposition and handing an underspecified task straight to execution

**5. Orchestrator–subagent design**
- Coordinator/subagent roles and responsibilities
- Configuring subagent invocation, spawning strategy, and context sharing
- Multi-agent topology choice — hub-and-spoke vs. pipeline vs. peer-to-peer — and what each implies for failure containment and coordination overhead

**6. Session and state management**
- Session state, resumption, and forking workflows
- Managing context handoff between steps and between agents

**7. Enforcement and hooks**
- Applying Agent SDK hooks to intercept and enforce business rules programmatically (e.g., forcing a specific tool sequence before a sensitive action executes) — a recurring exam theme is *deterministic enforcement vs. relying on prompt-based instruction*

### What the exam actually does with this domain

Sample question framing tends to look like: *"Design an agentic loop with tool integration, structured error handling, and escalation logic"* or *"Choose the correct enforcement mechanism when a specific tool sequence is required for critical business logic."* The recurring theme — confirmed across independent prep sources — is that Domain 1 rewards recognizing when *programmatic* guarantees are needed versus when prompt-level guidance is sufficient, and this domain frequently overlaps with Domain 2 (Tool Design & MCP) and Domain 5 (Context Management) in the same scenario.


## 03. Foundational agent concepts

**Agentic architecture** is the foundational design pattern behind modern AI systems that can work **autonomously (no human or logic dependent) toward a goal**. Instead of simply answering questions, these systems can **goal**, **plan**, **reason**, **take actions**, **evaluate outcomes**, and **adapt their approach** until the task is successfully completed.

<p align="center">
  <img width="500" height="350" alt="image"
       src="https://github.com/user-attachments/assets/73c5687c-7951-4ea4-9e2c-c6d85f2340c5" />
</p>


Traditional chatbots follow a simple request-response model: the user asks a question, the model generates a single answer, and the interaction ends. Agentic systems, however, are designed for **goal-oriented execution** rather than **one-time response generation**. They continuously analyze the current state of a task, decide what to do next, use available tools when necessary, assess the results of those actions, and repeat the process until the objective is achieved or human intervention is required.

In other words, a traditional chatbot focuses on **"What should I answer?"**, whereas an agentic system focuses on **"What should I do next to accomplish the goal?"** This shift from generating responses to executing tasks is what distinguishes agentic architecture from conventional conversational AI.

For example:

* **Traditional Chatbot:**
  *User:* "Write a Python function to reverse a string."
  *Behavior:* Generates the function and stops.

* **Agentic System (Claude Code):**
  *User:* "Upgrade my Python project to Python 3.13, fix any compatibility issues, run the tests, and update the documentation."
  *Behavior:* Reads the project, creates a plan, modifies multiple files, runs tests, analyzes failures, applies fixes, updates documentation, and finally presents a summary of the completed work.

This ability to **Goal → Plan → Reason → Act → Evaluate → Adapt → Repeat → Complete** is the defining characteristic of an agentic architecture and forms the foundation of autonomous AI systems such as Claude Code.

For the Claude Code Architect Foundation exam, candidates should understand:

* The characteristics of an AI agent and how it differs from a traditional chatbot or workflow.
* The architecture of iterative reasoning and execution loops.
* The role of planning, observation, action, and evaluation during task execution.
* How execution models determine task decomposition, orchestration, and completion.
* The relationship between the agent, tools, memory, permissions, and human oversight.
* When autonomous execution is appropriate versus when human approval or intervention is required.

A solid understanding of agentic architecture and execution models provides the conceptual foundation for every other topic in the certification, including multi-agent systems, tool orchestration, context management, reliability, and governance.

## Introduction to Agentic AI

### Evolution of AI Systems
- **Traditional Software** → deterministic, rule-based, predictable, no learning.
- **Rule-Based AI** → expert systems, thousands of rules, hard to maintain, no learning.
- **Machine Learning** → learns from data, better generalization, narrow tasks.
- **Deep Learning** → CNNs, RNNs, Transformers; breakthroughs in vision, speech, language.
- **Large Language Models** → one model, many tasks via prompting; limited to parameters.
- **Retrieval-Augmented Generation** → adds external knowledge, reduces hallucinations.
- **Tool-Using AI** → LLMs call APIs, databases, calculators; introduces planning.
- **Agentic AI** → continuous loop: reason → plan → execute → observe → reflect.
- **Multi-Agent Systems** → specialized agents collaborate; parallelism, scalability.

### Reactive vs Autonomous Systems
- **Reactive** → event-driven, deterministic, no planning.  
  Example: API validating payment.
- **Autonomous** → goal-driven, plans, adapts, uses tools.  
  Example: AI travel planner.
- **Comparison Table**:  
  Trigger (event vs goal), Decision-making (rules vs reasoning), Planning (none vs multi-step), Adaptation (minimal vs high).

### Agent vs Workflow
- **Workflow** → fixed sequence of steps (ETL, invoice processing).  
- **Agent** → dynamic decision-making, chooses next action at runtime.  
- **Key difference** → workflow = static path, agent = adaptive path.

### What Makes a System Agentic
- Goal-directed behavior  
- Dynamic planning  
- Tool selection  
- State management  
- Reflection & self-correction  
- Controlled autonomy  
- Not every LLM app is agentic (single prompt ≠ agent).

### Characteristics of Agentic Systems
- Goal Oriented
- Autonomy  
- Dynamic Reasoning & Planning  
- Tool use  
- Memory  
- Adaptability  
- Observability  
- Collaboration  
- Safety

### Agent Taxonomy
- **By intelligence** → reactive, deliberative, goal-based, utility-based, learning.  
- **By architecture** → single-agent, multi-agent, hierarchical, peer-to-peer, swarm.  
- **By specialization** → research, coding, data analysis, customer support, planning, orchestrator.


### Design Principles
1. Prefer workflows if sufficient.  
2. Separate reasoning from execution.  
3. Constrain tool access (least privilege).  
4. Maintain explicit state.  
5. Design for observability.  
6. Fail safely (retries, approvals, guardrails).  
7. Build modular agents.  
8. Optimize iteratively (latency, cost, accuracy).

## Balancing Factors
Different execution models balance:
 - Autonomy → how much the agent decides alone.
 - Reliability → safeguards, retries, human approvals.
 - Cost → compute, API calls, tool usage.
 - Oversight → when humans step in to approve or redirect.


Claude (model)
   └── Agent Loop (reasoning + tool-calling pattern)
         ├── Claude.ai Chat (human-driven, turn-by-turn, no persistent agent loop)
         ├── Claude Code (interactive product, human-driven, terminal/IDE)
         ├── Claude Cowork (interactive product, human-driven, desktop GUI — 
         │      "Claude Code without the terminal", same architecture, aimed 
         │      at non-developers, adds scheduled/triggered tasks)
         └── Agent SDK (library, code-driven — the shared runtime that both 
                Claude Code and Cowork are built on)
                ├── MCP → connects to external tools/data/services
                ├── Skills → teaches reusable workflows
                └── Subagents/Hooks/Plugins → fine-grained control





# Example: Loan Approval Assistant (Agentic Architecture)

### User Goal

> **"Can I get a $50,000 personal loan?"**

The user only gives the **goal**.

The agent decides **how** to achieve it.


## Step 1: Understand the Goal

```text
User
 │
 ▼
"I need a $50,000 loan."
```

The agent identifies the objective:

* Check customer identity
* Check credit score
* Check income
* Calculate eligibility
* Make decision


## Step 2: Plan

The agent creates a plan.

```text
Plan

1. Verify customer
2. Retrieve credit score
3. Check salary
4. Calculate eligibility
5. Decide approval
6. Explain result
```

Notice that the user never told the agent these steps.

The agent planned them itself.


## Step 3: Take Actions

The agent starts calling tools.

```text
Agent
 │
 ├── validate_customer()
 │
 ├── get_credit_score()
 │
 ├── get_salary()
 │
 └── calculate_loan_eligibility()
```

Each tool returns information.

Example

```text
Customer Verified ✓

Credit Score = 785

Salary = $8,500/month

Eligible Amount = $62,000
```


## Step 4: Reason

The agent evaluates everything.

```text
Credit Score > 750 ✔

Income sufficient ✔

No overdue loans ✔

Requested = $50,000

Eligible = $62,000
```

Reasoning:

> Customer qualifies.


## Step 5: Final Answer

```text
Congratulations!

Your application qualifies for a
$50,000 personal loan.

Reason:
• Good credit score
• Stable income
• Requested amount is within your limit
```


# What if Something Goes Wrong?

Suppose the salary service fails.

Instead of stopping, the agent adapts.

```text
get_salary()

↓

Timeout
```

The agent thinks:

> "I'll use the last salary recorded in the database."

```text
Get last payroll

↓

Salary = $8,200
```

The workflow continues automatically.

This is **adaptation**, one of the defining characteristics of agentic systems.


# Complete Flow Diagram

```text
                    User
                      │
                      ▼
      "Can I get a $50,000 loan?"
                      │
                      ▼
            Understand Goal
                      │
                      ▼
               Create Plan
                      │
      ┌───────────────┼────────────────┐
      ▼               ▼                ▼
Verify Customer   Check Credit    Check Salary
      │               │                │
      └───────────────┼────────────────┘
                      ▼
          Calculate Eligibility
                      │
                      ▼
             Reason & Evaluate
                      │
        Eligible? ───────────────► No
           │                        │
          Yes                       │
           ▼                        ▼
   Approve Loan          Explain Rejection
```


# Why is this Agentic?

A traditional workflow follows a fixed sequence:

```text
Validate
↓

Credit Check
↓

Salary Check
↓

Decision
```

An **agentic system** is more flexible:

```text
User Goal
      │
      ▼
   Planner
      │
      ▼
"First verify the customer."

      │
      ▼
Customer verified

      │
      ▼
"Now check credit score."

      │
      ▼
Credit service unavailable

      │
      ▼
"Use cached credit report instead."

      │
      ▼
Continue evaluation

      │
      ▼
Make final decision
```

The agent **plans**, **reasons**, **chooses tools**, **adapts when something fails**, and **works toward the goal without the user specifying each step**. That autonomous decision-making is what distinguishes an **agentic architecture** from a simple scripted workflow.
