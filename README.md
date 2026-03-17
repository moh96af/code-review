# Code Review Skills for Claude Code

A set of skills that turn Claude Code into a code review co-pilot for Bitbucket (and GitHub) workflows.

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| [review](./review/) | `/code-review:review <branch>` | Review someone else's PR — analyzes diff, spots bugs/security/performance issues, drafts comments |
| [verify-fixes](./verify-fixes/) | `/code-review:verify-fixes <pr-url>` | Check if a PR author resolved your review comments |
| [address-feedback](./addressing-feedback/) | `/code-review:address-feedback <pr-url>` | Fix review comments left on your own PR |

## Installation

### Install all skills

```bash
npx skills@latest add moh96af/code-review -g -y
```

### Install individual skill

```bash
npx skills@latest add moh96af/code-review@review -g -y
npx skills@latest add moh96af/code-review@verify-fixes -g -y
npx skills@latest add moh96af/code-review@address-feedback -g -y
```

## Prerequisites

### Bitbucket MCP Server

The `verify-fixes` and `address-feedback` skills require a Bitbucket MCP server for fetching PR comments.

#### Setup

1. Generate a Bitbucket App Password at: `https://bitbucket.org/account/settings/app-passwords/`
   - Required permissions: **Repositories: Read**, **Pull requests: Read/Write**

2. Add to your Claude Code settings (project or user level):

```json
{
  "mcpServers": {
    "Bitbucket MCP Server": {
      "command": "npx",
      "args": ["-y", "@aashari/mcp-server-atlassian-bitbucket"],
      "env": {
        "ATLASSIAN_BITBUCKET_USERNAME": "your-username",
        "ATLASSIAN_BITBUCKET_APP_PASSWORD": "your-app-password"
      }
    }
  }
}
```

3. Restart Claude Code. Verify with `/mcp` — you should see the Bitbucket MCP Server listed and connected.

> **Note:** The `review` skill works without Bitbucket MCP — it only needs local git access. The MCP is only required for skills that interact with PR comments via the Bitbucket API.

## License

MIT
