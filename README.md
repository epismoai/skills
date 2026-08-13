# Epismo Skill

One portable skill that teaches AI agents how to discover reusable Playbooks and coordinate real work in Epismo.

## Operating model

Epismo keeps two layers separate:

- **Playbook** — reusable, versioned guidance. Its Steps describe a recommended way of working.
- **Case** — one real matter worth sharing or resuming. Tasks and Records exist only when coordination needs them.

The agent runtime still owns planning, tool selection, credentials, retries, and local intermediate state. Epismo stores only durable guidance and shared work state.

## Structure

```text
skills/epismo/
  SKILL.md
  references/
    use-playbooks.md
    author-playbooks.md
    coordinate-cases.md
    improve-playbooks.md
    share-playbooks.md
```

The main [Epismo skill](./skills/epismo/SKILL.md) routes a request to the relevant action guide. CLI and MCP share the same Playbook, Case, Task, Record, Suggestion, Star, and Alias operation families; live MCP schemas and CLI help remain the source of truth for exact names, fields, enums, and limits.

## Connect

Use the surface available to the agent.

**CLI**

The CLI is a standalone native executable.

macOS and Linux:

```bash
curl -fsSL https://epismo.ai/install.sh | sh
```

Windows PowerShell:

```powershell
irm https://epismo.ai/install.ps1 | iex
```

npm:

```bash
npm install -g epismo
```

The npm package installs the same native executable. Direct downloads are also available from [GitHub Releases](https://github.com/epismoai/cli/releases).

After installation, see the available authentication and workspace commands:

```bash
epismo --help
```

**MCP**

Add **https://mcp.epismo.ai/** as a Streamable HTTP MCP server. The client authenticates through OAuth.

Copy **skills/epismo** into the agent's skills directory. Load **SKILL.md**; it directs the agent to detailed references only when needed.
