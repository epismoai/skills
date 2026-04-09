# Epismo Skills

Reusable skill packages that give AI agents structured capabilities through Epismo MCP or CLI.

## Why Skills

AI agents hit the same problems across teams and tools:

- Operational know-how stays trapped in chat histories.
- Multi-step processes don't transfer when you switch tools.
- Every new project restarts from scratch.

Skills solve this by packaging proven operational patterns into portable instruction sets that any agent with Epismo access can follow.

## Skills

| Skill                                                  | What it does                                                                                  |
| ------------------------------------------------------ | --------------------------------------------------------------------------------------------- |
| [Epismo Basics](./skills/epismo-basics/SKILL.md)       | Platform fundamentals: auth, CLI/MCP conventions, scope, share URL resolution, error handling |
| [Project Tracking](./skills/project-tracking/SKILL.md) | Create and update tasks and goals; plan multi-step work; unblock stalled queues               |
| [Workflow Pack](./skills/workflow-pack/SKILL.md)       | Discover, adapt, and release reusable workflows                                               |
| [Context Pack](./skills/context-pack/SKILL.md)         | Save session context, hand off tasks, load saved context from any tool                        |

**Epismo Basics** is a shared foundation — load it alongside any other skill.

## Use Cases

| Goal                                             | Skills to load                                   |
| ------------------------------------------------ | ------------------------------------------------ |
| Resume work after switching tools                | Context Pack                                     |
| Hand off a task to a teammate                    | Context Pack                                     |
| Add tasks, update status, plan a sprint          | Epismo Basics + Project Tracking                 |
| Find and reuse a community workflow              | Epismo Basics + Workflow Pack                    |
| Capture a proven process and publish it          | Epismo Basics + Workflow Pack                    |
| Delegate work to an AI agent with clear criteria | Epismo Basics + Project Tracking                 |
| Publish a best-practice guide for the community  | Context Pack                                     |
| Full project operations                          | Epismo Basics + Project Tracking + Workflow Pack |

## Quick Start

Tell your agent:

```
Set up Epismo access and load the Skills from https://raw.githubusercontent.com/epismoai/skills/main/README.md.
```

The agent will read this page and complete the steps. Or follow manually:

### 1. Connect

CLI and MCP connect to the same Epismo service. Use CLI if available; MCP otherwise.

**CLI** (preferred):

```bash
npm install -g epismo
epismo login --email you@example.com
epismo whoami
```

**MCP**: add `https://mcp.epismo.ai` as an MCP server in your client. Authentication is handled automatically via OAuth.

### 2. Select a workspace

```bash
epismo workspace list
epismo workspace use --workspace-id <workspace-id>
```

### 3. Load skills

Get the skill files from this repository (clone, download, or copy) and load the relevant `SKILL.md` files into your agent's context. Each skill follows this structure:

```
<skill-name>/
  SKILL.md          ← load this
  references/       ← loaded on demand
  templates/        ← structured output templates
```
