# Scribiz MCP Server — Automated Installation Guide

This guide is written for AI coding agents (Cline, Cursor, Claude Code, Codex, Windsurf, etc.) to install and configure the Scribiz MCP server automatically. Follow the steps in order.

## What this server does

Scribiz (`https://scribiz.com/mcp`) is a remote Streamable HTTP MCP server that gives an AI agent the context of any public video link (YouTube, TikTok, Instagram, X, Vimeo): spoken transcript with timestamps, scene-deduped on-screen OCR notes, summary, chapters, and cited moments.

## Prerequisites

None for the hosted server. It uses Streamable HTTP (`https://scribiz.com/mcp`), supports OAuth 2.1, and includes a keyless free daily allowance (no account or API key required to start).

## Installation

Add the remote HTTP server to the client's MCP configuration:

### Cline / Cursor / Claude Desktop (`mcpServers` JSON)

```json
{
  "mcpServers": {
    "scribiz": {
      "url": "https://scribiz.com/mcp"
    }
  }
}
```

### Claude Code CLI

```bash
claude mcp add --transport http scribiz https://scribiz.com/mcp
```

### VS Code (`.vscode/mcp.json`)

```json
{
  "servers": {
    "scribiz": {
      "type": "http",
      "url": "https://scribiz.com/mcp"
    }
  }
}
```

### Optional: Local CLI MCP Server (for local video/audio files on disk)

If the user wants to transcribe local files on macOS:

```bash
npm install -g scribiz
scribiz login
```

```json
{
  "mcpServers": {
    "scribiz-local": {
      "command": "scribiz",
      "args": ["mcp"]
    }
  }
}
```

## Verify the installation

Call `get_video_context` with `url: "https://www.youtube.com/watch?v=jNQXAC9IVRw"`. A working install returns the title, duration, summary, and timestamped moments for "Me at the zoo".
