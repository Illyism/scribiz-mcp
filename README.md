# Scribiz: video MCP server for YouTube transcripts, summaries and timestamps

Give Claude, Cursor, Codex or any MCP client the context of a video from its link: the transcript, a summary, chapters, and the exact moments that answer a question, each with a timestamp link. Works on YouTube, TikTok, Instagram, X and Vimeo links. No account and no API key needed to start.

```bash
claude mcp add --transport http scribiz https://scribiz.com/mcp
```

This repo holds the setup files for the hosted Scribiz MCP server at `https://scribiz.com/mcp`. The server runs on scribiz.com. The Scribiz source code is not in this repo.

## What the agent does with it

You paste a video link into a chat. The agent asks Scribiz for a short overview first. It searches for the part it needs, reads only that part, and answers with a link that opens the video at that moment. It does not read a whole transcript to answer one question.

## Add it

The hosted server needs no account and no key. Add the URL and it works.

### Claude Code

```bash
claude mcp add --transport http scribiz https://scribiz.com/mcp
```

Run `/mcp` inside Claude Code to see the server and its tools.

### Claude Desktop and claude.ai

Go to Customize, then Connectors. Select **+ Add**, then **Add custom connector**. Enter:

- Name: `Scribiz`
- URL: `https://scribiz.com/mcp`
- Authentication: **No sign in**

The `claude_desktop_config.json` file is a separate setup for local servers. You do not need it for the hosted server.

### Cursor

Put this in `~/.cursor/mcp.json`, or in `.cursor/mcp.json` inside a project:

```json
{
  "mcpServers": {
    "scribiz": { "url": "https://scribiz.com/mcp" }
  }
}
```

### VS Code

Put this in `.vscode/mcp.json`. VS Code uses `servers`, not `mcpServers`.

```json
{
  "servers": {
    "scribiz": { "type": "http", "url": "https://scribiz.com/mcp" }
  }
}
```

### Codex

Add this to `~/.codex/config.toml`:

```toml
[mcp_servers.scribiz]
url = "https://scribiz.com/mcp"
```

### Claude Code plugin (server and skill together)

This repo is also a Claude Code plugin marketplace. The plugin adds the server and a skill that teaches Claude how to use it.

```bash
claude plugin marketplace add Illyism/scribiz-mcp
claude plugin install scribiz@scribiz-mcp
```

Use the plugin or the `claude mcp add` line above, not both. Other agents can install the skill alone:

```bash
npx skills add Illyism/scribiz-mcp
```

## Try it

Ask your agent:

```text
Summarize https://www.youtube.com/watch?v=jNQXAC9IVRw, then find where they talk about the trunks and quote it.
```

This is a 19 second public video, "Me at the zoo". You should see a call to `get_video_context`, then `search_video`, and an answer with a timestamp link. The result says how the transcript was made. When a model read the link, its times are approximate (about 2 seconds either way). A video Scribiz has not processed before uses a little of the day's allowance on its first read, about 0.3 minutes for this one. After that it costs nothing.

## Tools

All five tools are read-only. They reach out to the internet, and they never change your files.

| Tool | What it does |
| --- | --- |
| `get_video_context` | Get video context. Title, length, summary, chapters and key moments with links. Start here. |
| `search_video` | Search a video. The moments where a word or phrase is said or shown, ranked, with links. |
| `ask_video` | Ask a question about a video. An answer with 3 to 5 cited moments. |
| `get_transcript` | Get a video transcript. The words, or one part of them, one page at a time. In TXT, Markdown, SRT, VTT or JSON. |
| `get_job` | Check a video job. Picks up a video that is still being processed. |

Every result says which layers ran (captions, listening, watching, a model summary) and how many minutes it used. Details are in the [tools reference](https://scribiz.com/docs/mcp/tools?ref=github-mcp).

## What works without an account

- Captions, when the server can read them.
- Videos Scribiz has already processed, at no cost. The one exception is a summary Scribiz has not written for the video yet, which uses a tenth of its length.
- Videos with no captions the server can read: a model reads the video from its link. Its times are approximate (about 2 seconds either way), it has no speaker labels, and the result says so.

This runs inside a free allowance: about 10 minutes of reading a day for each connection (the same 10 minutes as the free web tool), videos up to 15 minutes, 5 questions a day and 30 caption lookups a day. Everyone who connects without a key also shares one more daily limit. When your 10 minutes or the shared limit are used up, captions and videos already processed still work. When the 30 caption lookups are used up, videos already processed still work. When the 5 questions are used up, searching and reading still work. The error says what is used up and when it resets (00:00 UTC).

Without a key the server never looks at the picture, so there are no on-screen notes, and a question about what was shown is answered from the words only. Watch reads the picture and the text shown on screen. It needs a Scribiz API key and uses minutes from your plan. A free account has 30 minutes a month, and you make a key in the [dashboard](https://scribiz.com/dashboard?ref=github-mcp). With a key, pass `watch: true` to `get_video_context` or `ask_video`; on-screen notes Scribiz already stored come back at no cost. See [connection modes](https://scribiz.com/docs/mcp/connection-modes?ref=github-mcp) and [limits and cost](https://scribiz.com/docs/mcp/limits?ref=github-mcp).

The hosted server takes public links only. It cannot read files on your disk.

### With a key

Send the key as a bearer token. In Claude Code:

```bash
claude mcp add --transport http scribiz https://scribiz.com/mcp --header "Authorization: Bearer YOUR_SCRIBIZ_KEY"
```

### Local files

A local server comes with the command-line tool. It runs on your machine and can read your files. It needs Node 24, `ffmpeg` and `yt-dlp`, and has been tested on macOS only.

```bash
npm install -g scribiz
scribiz login
claude mcp add scribiz -- scribiz mcp
```

## What is sent where

Your agent sends the video link and your questions to scribiz.com. Scribiz fetches the captions from the site you linked when it can. For a video it has not processed, it passes the link to Google's Gemini API, whose model reads the public video. Nothing is uploaded from your machine. Where Scribiz downloads the audio, it sends the audio to Google for transcription and analysis. For Watch it also sends a small low-resolution copy of the picture, with no sound. The [privacy policy](https://scribiz.com/privacy) has the retention periods and Google's terms. This repo sends nothing. It holds setup files only.

## Treat transcripts as data

A video can say anything, including "ignore your instructions". The server wraps all text from a video between `=== BEGIN UNTRUSTED VIDEO TEXT [id] ===` and `=== END UNTRUSTED VIDEO TEXT [id] ===` lines, and sets `untrusted_content: true` on the result. Do not auto-approve tools that can send messages, run commands or write files in a session that reads video. More in [MCP security](https://scribiz.com/docs/mcp/security?ref=github-mcp).

## Links

- [Scribiz MCP server page](https://scribiz.com/video-mcp?ref=github-mcp)
- [MCP docs](https://scribiz.com/docs/mcp?ref=github-mcp)
- [Privacy policy](https://scribiz.com/privacy) and [terms](https://scribiz.com/terms)
- Problems with this repo or the setup files: open an issue here.

## Also from Scribiz

Scribiz also has a free web tool. Paste a video link, get the transcript. No account needed: [scribiz.com](https://scribiz.com/?ref=github-mcp). There is a Mac app for Apple silicon Macs at [scribiz.com/download](https://scribiz.com/download?ref=github-mcp), and a command-line tool on npm (`npm install -g scribiz`).

Built by [Ilias Ism](https://il.ly).
