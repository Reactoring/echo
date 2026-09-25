# Echo

**A self-taught Claude Code agent that learns your codebase one ticket at a time.**

Every ticket leaves an echo: a lesson, a map, a better tool. The next ticket starts from there.

Echo is the method I built to contribute to complex Java telecom microservices with Claude Code, without prior knowledge of the modules involved. It is not a RAG pipeline. It is a flat set of Markdown files, one index, and a loop that makes the agent better after each ticket.

---

## Why not RAG

| Classic RAG | Echo |
|---|---|
| Vector DB, embeddings, chunking pipeline | Plain Markdown files + one index |
| Retrieves chunks by similarity, often out of context | Claude reads the index and opens the exact file it needs |
| Knowledge is frozen at ingestion time | Knowledge is rewritten after every ticket |
| Hard to inspect or correct | Human-readable, editable, versionable |
| Infrastructure to run and maintain | Nothing to run |

Claude Code already knows how to read, search and reason. Echo gives it a good map instead of a search engine.

---

## Architecture

```
                         MEMORY.md  (index)
                              │
   ┌──────────┬───────────┬───┴───────┬─────────────┬─────────────┐
lesson_*    ticket_*     tool_*     module_*     playbook_*
 business    status &    CLI and    map of a     step-by-step
 & tech      discussion  scripts    microservice procedures
 knowledge
```

- **One index.** `MEMORY.md` is loaded at the start of every session. One line per file: link + keywords + short hook.
- **Flat files, no folders.** A folder forces a single classification. A lesson about SIM activation is both *billing* and *provisioning*. Keywords in the file name and index let Claude find it from either angle, with one `grep` or one glance at the index.

### Index example

```markdown
- [SIM activation flow](lesson_provisioning_sim-activation.md) — async, retried by scheduler, never call twice
- [PROJ-1234](ticket_PROJ-1234.md) — add APN field to eSIM profile — IN REVIEW
- [Bitbucket CLI](tool_bitbucket.md) — clone, branch, PR, read old PR comments
- [sim-provisioning-service](module_sim-provisioning-service.md) — entry points, key classes, Maven profiles
- [Ticket playbook](playbook_ticket.md) — from ticket ID to pull request
```

---

## File types

Every file starts with a short frontmatter so Claude can judge relevance without reading the body.

```markdown
---
name: lesson_provisioning_sim-activation
description: SIM activation is async and retried; calling it twice creates duplicates
type: lesson
keywords: [sim, activation, provisioning, retry, idempotency]
---
```

| Type | Naming | Content |
|---|---|---|
| **Lesson** | `lesson_<topic>_<subject>.md` | One business or technical fact learned the hard way, with **Why** and **How to apply**. |
| **Ticket** | `ticket_<ID>.md` | Goal, status, decisions, discussion summary, branch, PR link, lessons produced. |
| **Tool** | `tool_<name>.md` | How to call a CLI or script, allowed operations, examples, known errors. |
| **Module map** | `module_<service>.md` | Entry points, key classes, data flow, config, build and test commands, pitfalls. |
| **Playbook** | `playbook_<procedure>.md` | Step-by-step procedure the agent follows (ticket, PR review, bug fix). |

**Module maps** are what make contributing to unknown code possible: the first ticket on a module pays the exploration cost once, and every next ticket starts with a map.

**Playbooks** make the agent's behavior predictable: same steps, same checks, same PR format, every time.

---

## Ticket lifecycle

```
Ticket ID
  → fetch ticket (Jira CLI)                 → create/update ticket_<ID>.md
  → read index, open matching lessons, module maps
  → read specs (Confluence CLI) and similar past PRs (Bitbucket CLI)
  → plan → implement → build & test
  → open PR (Bitbucket CLI)
  → retro: write back what was learned      → the echo
```

The agent follows `playbook_ticket.md`. The human validates the plan and reviews the PR.

---

## Tools: CLI and scripts, not MCP

Echo talks to Jira, Confluence, Bitbucket and servers through small CLIs and shell scripts, each documented in a `tool_*.md` file.

- **Jira**: read ticket, comments, status; update status.
- **Confluence**: read specs and technical pages.
- **Bitbucket**: clone, branch, commit, open PR, read PR discussions, **read old PRs to learn team conventions**.
- **SSH**: logs and environments, through `~/.ssh/config` aliases.

### Why CLI over MCP

- **Context cost.** MCP tool schemas sit in the context for the whole session. A CLI costs nothing until it is called.
- **Deterministic and auditable.** Scripts are versioned, reviewed and behave the same every time.
- **Least privilege.** A script exposes only the operations you allow. No third-party server holding broad credentials.
- **Native to Claude Code.** Claude is very good at shell. No extra server to install, run or trust.

### Security

- Tokens live in environment variables or the OS keychain. Scripts read them; **Claude never sees them**.
- No secret is ever written in a memory file.
- SSH uses keys through `ssh-agent`. No password in a prompt.
- Read-only by default. Write operations (status change, PR creation) are separate, explicit commands.

---

## The echo: how the agent improves itself

At the end of each ticket, the playbook's last step is a retro. The agent:

1. **Writes lessons** from what surprised it or what went wrong.
2. **Updates the module map** with the classes and flows it discovered.
3. **Turns review comments into conventions**, so the same remark is never made twice.
4. **Fixes the tools** that failed or were missing.
5. **Updates the ticket file** and the index.

Ticket 1 on a module is exploration. Ticket 10 starts with a map, known pitfalls, team conventions and working tools. The agent gets faster and its PRs need fewer review rounds, without any retraining.

---

## Getting started

1. Create `MEMORY.md` and `playbook_ticket.md`.
2. Write one `tool_*.md` per system you use (Jira, Bitbucket…), with a wrapper script that reads its token from the environment.
3. Give Claude a ticket ID and the playbook. Let the retro step create the rest.

## Limits

- Works best on well-scoped tickets. Large features still need human design.
- Memory quality depends on the retro step: skip it and there is no echo.
- The index must stay short. Merge and prune regularly.

---

*Author: Amine Dkhili. [linkedin.com/in/dkhiliamine](https://www.linkedin.com/in/dkhiliamine)*
