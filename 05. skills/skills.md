# Skills in Claude: Exam Notes

A **Skill** is a reusable folder of instructions, and optionally scripts and reference files, that teaches Claude how to perform a specific type of task consistently.

## 2. Key Points to Memorize
- **Core file:** `SKILL.md` (required)
- **Metadata:** every skill has a **name** and a **description**
- **Loading:** Claude loads a skill only when a task matches its description
- **Purpose:** consistency, repeatability, and domain expertise without re-prompting
- **Persistence:** a skill is saved and reusable, unlike a prompt, which lives in one chat

## 3. Structure of a Skill

| Component | Role |
|---|---|
| `SKILL.md` | Main instructions: steps, rules, examples |
| Name + description | Tells Claude *what* the skill is and *when* to use it |
| Scripts (optional) | Code Claude can run, e.g. to build files |
| Reference files (optional) | Templates, guidelines, sample data |

## 4. How Skills Work: Progressive Disclosure
1. Claude initially sees **only the name and description** of each skill.
2. When a request matches, Claude **reads the full `SKILL.md`**.
3. If needed, it then opens **extra files or scripts** in the folder.

**Why it matters:** this saves context space, since only relevant instructions are loaded.

## 5. Types of Skills
- **Built-in skills:** provided by Anthropic, e.g. Word (.docx), PDF, PowerPoint (.pptx), Excel (.xlsx)
- **Custom skills:** created by a user for personal workflows
- **Organization skills:** shared across a team or company

## 6. Invocation
- **Automatic:** Claude matches the task to a skill's description
- **Manual:** the user calls it by name, often as a slash command (`/skill-name`)

## 7. Skills vs. Related Concepts

| Concept | Difference |
|---|---|
| **Prompt** | One-time instruction, not reusable |
| **Skill** | Saved, reusable procedure, loaded on demand |
| **Connector (MCP)** | Gives Claude *access* to external apps and data |
| **Memory** | Stores facts and preferences about the user |

**Memory trick:** *Connectors = access, Skills = know-how, Memory = personal facts.*

## 8. Benefits
- Consistent output quality and format
- Saves time (no retyping instructions)
- Efficient context usage
- Shareable across teams
- Encodes expert or company-specific processes

## 9. Example Use Cases
- Writing reports in a company house style
- Reviewing contracts against a checklist
- Generating formatted Excel or PowerPoint files
- Following a standard research or analysis process

## 10. Minimal `SKILL.md` Example
```markdown
---
name: weekly-report
description: Use when the user asks for a weekly status report.
---

1. Summarize completed work in 3 bullets.
2. List blockers.
3. End with next week's priorities.
```

## 11. Likely Exam Questions

**Q1. What is the required file in a skill?** `SKILL.md`.

**Q2. What does Claude see before loading a skill?** Only its name and description.

**Q3. Why use progressive disclosure?** To use context efficiently by loading only relevant instructions.

**Q4. Skill vs. prompt?** A skill is reusable and saved; a prompt applies to one conversation.

**Q5. Two ways to trigger a skill?** Automatic matching, or manual invocation by name or slash command.

## 12. One-Line Summary
> Skills package expertise into reusable folders that Claude loads only when needed, giving consistent, high-quality results for repeated tasks.

If these notes were meant for a different exam topic, tell me the subject and level and I'll rewrite them. I can also turn these into a Word document or a practice quiz.
