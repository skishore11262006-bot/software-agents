A local-first AI coding agent for VS Code. It can read, search, debug and propose fixes for your codebase, but **never edits files or runs shell commands without your explicit approval**. 

## Why This Exists

Most AI coding agents want to act autonomously. This project takes the opposite approach. Instead of "ask it to fix and hope", it forces human-in-the-loop approval for every potentially destructive action. 

Power with a cage around it. 

## Features

### Agent Capabilities
- **Local-First** - Runs with [Ollama](https://ollama.com/) by default. Your code never leaves your machine.
- **Workspace Aware** - Can list files, read files, search across your codebase and pull errors directly from the VS Code Problems panel.
- **Tool Calling Loop** - Uses a reasoning loop to plan its next action based off of real tool outputs instead of guessing.
- **MCP Support** - Connect any MCP server through the config and all of its tools are automatically exposed to the agent. 

### Safety First
- **Diff Approval** - The agent can *never* directly write to a file. `propose_edit` will open a native VS Code diff for you to `Apply` or `Reject`.
- **Command Risk Engine** - Every shell command is assessed before execution. Commands are categorized as `low`, `high`, or `blocked` with reasoning provided.
- **Hard Blocks** - Destructive commands such as `rm -rf /`, `mkfs`, `curl | sh`, fork bombs and raw disk writes are automatically blocked with no way to bypass.
- **Explicit Confirmation** - High risk commands (`sudo`, `git push --force`, `git reset --hard`, recursive `rm`, etc) require a modal confirmation before running.
- **Audit Logging** - Every allowed, denied, blocked, or failed command is logged to the `AI Agent Audit` output channel.
- **Workspace Sandboxing** - All file operations are restricted to the currently opened workspace. Paths outside of it are rejected.
- **Secure Secrets** - API keys are stored in VS Code's `SecretStorage`, never in `settings.json`.

## Tech Stack

- **Language:** TypeScript
- **Runtime:** VS Code Extension API
- **LLM:** [Ollama](https://ollama.com/) (OpenAI and Anthropic adapters are currently stubbed)
- **MCP:** [@modelcontextprotocol/sdk](https://github.com/modelcontextprotocol/typescript-sdk)
- **Safety:** Custom regex based risk engine


## Getting Started

### Prerequisites

- [Visual Studio Code](https://code.visualstudio.com/) v1.85.0 or higher
- [Node.js](https://nodejs.org/) v18+
- [Ollama](https://ollama.com/) installed and running
- A coding model pulled (Recommended: `qwen2.5-coder:7b`)

```bash
ollama pull qwen2.5-coder:7b
Installation (Development)
1. Clone the repository
git clone <your-repo-url>
cd "agen in vs"
2. Install dependencies
npm install
3. Compile the extension
npm run compile
4. Launch the extension
- Open this folder in VS Code
- Press F5 to open a new "Extension Development Host" window
- In that window, run the command AI Software Engineer: Open Chat from the Command Palette (Ctrl+Shift+P / Cmd+Shift+P)
Usage
1. Open a workspace in the Extension Development Host
2. Run AI Software Engineer: Open Chat to open the agent panel
3. Ask it to analyze your codebase, find bugs, explain errors, or propose fixes
4. When it wants to edit a file, a diff will pop up. Review and hit Apply or Reject
5. When it wants to run a command, you'll be prompted with its risk level and reasoning. Approve or deny at your own discretion
Configuration
These can be configured under Settings > Extensions > AI Software Engineer
Setting	Default	Description
aiSoftwareEngineer.provider	ollama	LLM provider (ollama, openai, anthropic)
aiSoftwareEngineer.model	qwen2.5-coder:7b	Model to use with your provider
aiSoftwareEngineer.ollamaUrl	http://localhost:11434	URL for your Ollama instance
aiSoftwareEngineer.maxIterations	15	Maximum amount of tool calls before the agent is forced to stop
aiSoftwareEngineer.mcpServers	[]	Array of MCP servers to connect on startup e.g [{"name": "filesystem", "command": "npx", "args": ["-y", "@modelcontextprotocol/server-filesystem"]}]
To set an API key for remote providers, run AI Software Engineer: Set API Key from the command palette. This will be stored securely in VS Code SecretStorage.
Architecture Overview
src/
├── extension.ts      # Extension entrypoint, commands & lifecycle
├── config.ts         # Settings + SecretStorage
├── agent/            # Orchestrator, system prompt
├── llm/              # Adapters (Ollama implemented, others stubbed)
├── tools/            # Built-in tools (read/search/errors/patch/run)
├── mcp/              # MCP client, dynamically loads remote tools
└── webview/          # Chat UI
- Orchestrator manages the conversation, tool schema injection, execution, and iteration limiting.
- proposeEditTool creates a virtual diff via a custom aiproposed: scheme before applying any changes.
