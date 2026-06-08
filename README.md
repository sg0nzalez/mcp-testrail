# TestRail MCP Server

<a href="https://flatt.tech/oss/gmo/trampoline" target="_blank"><img src="https://flatt.tech/assets/images/badges/gmo-oss.svg" height="24px"/></a>

This Model Context Protocol (MCP) server provides tools for interacting with TestRail directly from Claude AI and other MCP-supported clients like Cursor. It allows you to manage test cases, projects, suites, runs, and more without leaving your conversation with the AI.

## Available Tools

The TestRail MCP server provides the following tools:

| Category | Tools |
|----------|-------|
| **Projects** | `getProjects`, `getProject` |
| **Suites** | `getSuites`, `getSuite`, `addSuite`, `updateSuite` |
| **Cases** | `getCase`, `getCases`, `addCase`, `updateCase`, `deleteCase`, `getCaseTypes`, `getCaseFields`, `copyToSection`, `moveToSection`, `getCaseHistory`, `updateCases`, `addBdd`, `getBdd` |
| **Sections** | `getSection`, `getSections`, `addSection`, `moveSection`, `updateSection`, `deleteSection` |
| **Runs** | `getRuns`, `getRun`, `addRun`, `updateRun` |
| **Tests** | `getTests`, `getTest` |
| **Results** | `getResults`, `getResultsForCase`, `getResultsForRun`, `addResultForCase`, `addResultsForCases` |
| **Plans** | `getPlans` |
| **Milestones** | `getMilestones` |
| **Shared Steps** | `getSharedSteps` |

## Usage

You can connect this MCP server by setting like the below. This method uses `npx` to automatically download and run the latest version of the package, eliminating the need for local installation.

```json
// Example configuration using npx
{
  "mcpServers": {
    "testrail": {
      "command": "npx",
      "args": ["@bun913/mcp-testrail@latest"],
      "env": {
        "TESTRAIL_URL": "https://your-instance.testrail.io", // Replace with your TestRail URL
        "TESTRAIL_USERNAME": "your-email@example.com", // Replace with your TestRail username
        "TESTRAIL_API_KEY": "YOUR_API_KEY" // Replace with your TestRail API key
      }
    }
  }
}
```

## Troubleshooting

- **`spawn npx ENOENT` / `spawn node ENOENT` (commonly on macOS)**: your MCP host (Cursor, Claude Code, Claude Desktop, …) cannot find `npx` or `node` at process-spawn time. The chat UI usually surfaces this as a generic "MCP server doesn't work" with no useful detail; the per-server log is the diagnostic source of truth.

  **Why it happens on macOS:** GUI apps launched from the Dock, Spotlight, or Finder inherit launchd's minimal `PATH` (`/usr/bin:/bin:/usr/sbin:/sbin`). If you installed Node via a version manager (`nvm`, `asdf`, `mise`, `fnm`, `Volta`) or Apple Silicon Homebrew (`/opt/homebrew/bin/`), `npx` lives outside that PATH — only your shell startup file (`~/.zshrc` / `~/.bashrc`) adds it. Your terminal works because the shell ran the startup file; the GUI-app process never did.

  **Diagnose** by checking the per-server log for `spawn npx ENOENT`:

  - **Cursor:** `~/Library/Application Support/Cursor/logs/<session>/window<N>/exthost/anysphere.cursor-mcp/MCP <server>.log`
  - **Claude Code / Claude Desktop:** `~/Library/Logs/Claude/`

  **Fix** by replacing `"npx"` in your MCP config with its absolute path. Run `which npx` in your normal terminal:

  ```text
  /Users/you/.nvm/versions/node/v24.15.0/bin/npx   # nvm
  /opt/homebrew/bin/npx                            # Apple Silicon Homebrew
  /usr/local/bin/npx                               # Intel Homebrew / system Node
  ```

  Then update your MCP config:

  ```json
  {
    "mcpServers": {
      "testrail": {
        "command": "/Users/you/.nvm/versions/node/v24.15.0/bin/npx",
        "args": ["@bun913/mcp-testrail@latest"],
        "env": {
          "TESTRAIL_URL": "https://your-instance.testrail.io",
          "TESTRAIL_USERNAME": "your-email@example.com",
          "TESTRAIL_API_KEY": "YOUR_API_KEY"
        }
      }
    }
  }
  ```

  Restart your MCP client after the change. The same fix applies to every `npx`-launched MCP server — if `mcp-testrail` is failing for this reason, your other `npx`-launched servers are likely failing too.

- **Authentication issues**: Check your TestRail API credentials.
- **Your conversation is too long**: Use `limit` and `offset` parameters for test cases and sections to paginate results.
- **HTTP 400 errors when creating/updating test cases**: TestRail projects have different templates, custom fields, and required fields. This MCP server passes your parameters directly to the TestRail API — it does not validate or transform them. If you encounter 400 errors, define your project's rules in `CLAUDE.md` or `AGENTS.md` so the LLM sends the correct parameters. For example:

  ```markdown
  # TestRail Rules for This Project
  - Project ID: 1
  - Always use template 2 (Separated Steps) when creating test cases
    - Use `customStepsSeparated` (array of step objects)
    - Do NOT send `customSteps` or `customExpected` with template 2
  - Required custom fields: custom_automation_type (default: 0)
  - Call `getCaseFields` at the start of a session to check available fields
  ```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Acknowledgements

- [TestRail API](https://docs.testrail.techmatrix.jp/testrail/docs/702/api/)
- [Model Context Protocol SDK](https://github.com/modelcontextprotocol/typescript-sdk)

