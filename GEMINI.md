# Gemini CLI - Codebase Review & Insights: `docker-nanobot`

This document summarizes the initial architectural review, security analysis, and proposed feature roadmap for the `docker-nanobot` project.

## 🏗️ Architectural Overview

`docker-nanobot` is a modular, asynchronous agent framework designed for extensibility and security. Its core strengths include:

*   **Asynchronous Message Bus:** A central `MessageBus` handles communication between inbound channels (CLI, Discord, Slack, etc.) and the core agent loop, ensuring high responsiveness and support for concurrent sessions.
*   **Modular Providers & Channels:** Clear abstraction layers for LLM providers (OpenAI, Anthropic, Azure, etc.) and communication channels make it easy to add new integrations.
*   **Progressive "Skills" System:** Instead of bloating the context window, the agent uses a discovery mechanism for "Skills" (defined in `SKILL.md` files). It can read these files on-demand using `read_file`, allowing for a virtually unlimited set of specialized capabilities.
*   **MCP Integration:** Native support for the Model Context Protocol (MCP) allows the agent to leverage a wide range of external tools and data sources.
*   **Subagent Workflows:** The `SubagentManager` enables delegation of long-running or complex tasks to background "worker" agents, preventing the main interaction loop from blocking.

## 🔒 Security Analysis

The framework incorporates several key security measures:

*   **Filesystem Guardrails:** Tools like `read_file` and `write_file` use `Path.resolve()` and strict workspace boundary checks to prevent path traversal attacks.
*   **Shell Execution Safety:** `ExecTool` includes a configurable list of `deny_patterns` for dangerous commands and can be restricted to the workspace directory.
*   **Network SSRF Protection:** Built-in utilities resolve hostnames and block access to private/internal IP ranges (e.g., `127.0.0.1`, `192.168.x.x`) for both web-fetching and shell-based network tools.
*   **Isolation of Subagents:** Subagents are restricted from spawning additional agents or sending outbound messages directly, mitigating risks of uncontrolled recursion or "message storms."

## 🧠 Memory & Context Management

The agent employs a two-layer memory strategy:

1.  **Long-term Memory (`MEMORY.md`):** A distilled set of facts and preferences maintained by a periodic consolidation process.
2.  **Event Log (`HISTORY.md`):** A searchable record of past interactions, useful for long-term recall.
3.  **Automatic Consolidation:** The framework monitors token usage and automatically triggers consolidation when the context window limit is approached.

## 🚀 Recommended Roadmap & New Features

To further enhance the agentic workflows and security of `docker-nanobot`, the following features are proposed:

### 1. Human-in-the-Loop (HITL) for "Sensitive Tools"
Add a mechanism to mark certain tools (e.g., `exec`, `write_file`, `mcp_...`) as "sensitive." When triggered through non-interactive or high-risk channels, the agent should pause and wait for explicit user approval via a confirmation message.

### 2. History Search & RAG Integration
As `HISTORY.md` and `MEMORY.md` grow, provide the agent with a `search_history` tool (using `grep` or a vector-based search) to allow it to efficiently recall specific details from past sessions without loading the entire history into context.

### 3. Ephemeral Docker Sandboxing for `ExecTool`
Provide an option to execute shell commands within an ephemeral, sidecar Docker container. This would provide absolute isolation from the host system, even if the agent is compromised by a malicious payload.

### 4. Knowledge Base "Skill"
Implement a specialized skill that allows the agent to index and search a local directory of documents (PDFs, Markdown, etc.) using Retrieval-Augmented Generation (RAG).

### 5. Web-Based Monitoring Dashboard
A lightweight, read-only web UI to monitor active sessions, track subagent progress, and provide a visual interface for managing the agent's long-term memory.

### 6. Enhanced Multi-Modal Support
Improve the handling of images and other non-textual data across all channels, ensuring that subagents can also "see" and process visual information retrieved during their tasks.
