# Epismo Skill

One portable skill that teaches AI agents how to use Epismo's available tools well.

## Why a Skill

AI agents hit the same problems across teams and tools:

- Operational know-how stays trapped in chat histories.
- Multi-step processes don't transfer when you switch tools.
- Every new project restarts from scratch.

The Epismo skill provides a small operating model for deciding where work belongs and how to handle it safely. Live tool schemas or CLI help remain the source of truth for commands and arguments.

## Structure

```text
skills/epismo/
  SKILL.md
  references/
    execute.md
    reuse.md
    capture.md
    evolve.md
    share.md
```

The main [Epismo skill](./skills/epismo/SKILL.md) routes each request to one of three durable state types:

- **Track** — work being planned or executed now.
- **Workflow pack** — a procedure worth reusing.
- **Context pack** — knowledge worth carrying across sessions, tools, or people.

Detailed guidance is organized by user action rather than by storage type:

- execute current work;
- reuse an existing procedure;
- capture learning;
- evolve durable knowledge;
- share or publish.

## Quick Start

Tell your agent:

```
Set up Epismo access and load the skill from github.com/epismoai/skills.
```

The agent will read this page and complete the steps. Or connect manually:

### 1. Connect

CLI and MCP connect to the same Epismo service. Use the surface available in the current environment.

**CLI**:

```bash
npm install -g epismo
epismo login --email you@example.com
epismo whoami
```

**MCP**: add `https://mcp.epismo.ai` as an MCP server in your client. Authentication is handled automatically via OAuth.

### 2. Load the skill

Clone, download, or copy `skills/epismo` into your agent's skills directory. Load `SKILL.md`; its references are read only when relevant.
