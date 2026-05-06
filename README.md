# Biggest Bro

![Next.js](https://img.shields.io/badge/Next.js_15-000000?style=flat&logo=nextdotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white)
![Anthropic](https://img.shields.io/badge/Anthropic_Claude-D97706?style=flat&logo=anthropic&logoColor=white)
![Cloudflare](https://img.shields.io/badge/Cloudflare-F38020?style=flat&logo=cloudflare&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)

**Domain-expert AI agent for independent creators. Content strategy, audience growth, and platform optimization — powered by Claude Opus 4.7 with extended thinking and function calling.**

> A production AI agent that goes beyond generic chat. Built with deep context about platform algorithms, content ecosystems, and creator growth patterns — backed by Claude extended thinking for multi-step strategy questions that need real reasoning, not quick answers.

---

## Table of Contents

- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Features](#features)
- [Security](#security)
- [Key Engineering Decisions](#key-engineering-decisions)
- [API](#api)
- [Running This](#running-this)

---

## Architecture

```
Browser
  │
  ▼
Next.js 15  (App Router)
  │
  ├── Chat UI  (React — optimistic updates · streaming)
  │
  └── /api/chat  (Edge API route)
        │
        └── Anthropic SDK  (claude-opus-4-7 · extended thinking)
              │
              ├── System prompt  (creator-domain expert persona · ~3,000 tokens)
              ├── Conversation history  (client-managed · server-validated per turn)
              ├── Tool use  (content calendar · title analyzer · release timing · growth plan)
              └── Prompt cache  (system prompt + tool schemas · 5-min TTL)
```

---

## Tech Stack

| Layer | Technology | Notes |
|---|---|---|
| Frontend | Next.js 15, App Router, React | Optimistic UI updates |
| AI | Anthropic Claude API (`claude-opus-4-7`) | Extended thinking, 8k token budget |
| Streaming | Server-Sent Events via Anthropic SDK | Progressive token delivery |
| Runtime | Cloudflare Workers / Next.js edge | Zero cold starts |
| Caching | Anthropic prompt caching | System prompt + tool schemas, 5-min TTL |
| Validation | Zod | Conversation history + tool input validation |

---

## Features

### Creator-Domain Intelligence

Biggest Bro has deep context about:
- YouTube algorithm signals (CTR, AVD, re-watch rate, click-through patterns)
- Platform release strategy (timing, sequencing, pre-release campaigns)
- Audience growth levers specific to each platform
- Content formats that perform in niche communities
- Common mistakes independent creators make with content strategy

### Tools (Function Calling)

| Tool | Purpose |
|---|---|
| `generate_content_calendar` | 30-day content plan based on niche and goals |
| `analyze_title` | Scores a YouTube title for CTR potential with suggestions |
| `compare_release_windows` | Evaluates release timing by day/season |
| `audience_growth_plan` | Platform-specific growth roadmap |

Tool outputs are rendered as structured UI components in the chat — not raw JSON. The AI narrative references the tool output naturally.

### Conversation Management
- Full conversation history maintained client-side
- Server-side validation prevents history tampering on every turn (role sequence, length limits, content bounds)
- System prompt cached at Anthropic — consistent persona, lower latency on every turn
- Extended thinking enabled for complex multi-step strategy questions

---

## Security

### API Credential Protection
- Anthropic API credentials stored server-side as environment variables — never included in client bundles or accessible from the browser.
- Edge runtime API route — credentials exist only in the Cloudflare Workers execution context.

### Conversation History Validation
- Conversation history is validated server-side on every request — role sequence checked, length capped, content bounds enforced.
- A client cannot inject system-role messages or manipulate prior AI responses in the history.
- History tampering returns a `400` before any Claude API call is made.

### Prompt Injection Prevention
- User messages are passed in the `user` role only — never interpolated into the system prompt.
- Tool inputs validated with Zod before execution.
- No external API calls from tools without explicit user-facing confirmation.

### Rate Limiting
- API route rate-limited per user session — prevents abuse and runaway token consumption.
- Token budget enforcement: extended thinking budget capped at 8,000 tokens per turn.

### No Persistent Data
- No conversation history stored server-side — each session is ephemeral.
- No user data transmitted beyond the explicit conversation context.

---

## Key Engineering Decisions

### Claude Opus 4.7 with extended thinking
Creator strategy questions require multi-step reasoning — "why is my channel plateauing?" involves understanding algorithm signals, content patterns, upload frequency, and audience behavior simultaneously. Extended thinking produces more coherent, well-reasoned strategic advice than a fast-path response.

### Prompt caching for the system prompt
The system prompt (domain expert context, creator taxonomy, platform behavior patterns) is ~3,000 tokens. Caching it saves approximately $0.03 per conversation turn at scale and reduces first-token latency from ~800ms to ~200ms on cache hits.

### SSE streaming for conversational feel
Tokens stream progressively via Server-Sent Events. Long-form strategy responses (1,200+ tokens) that stream feel fast and readable; the same content appearing all at once feels slow.

### Optimistic UI updates
Chat messages appear in the UI immediately on submit — before the API call returns. If the call fails, the message rolls back with an error state. This eliminates perceptible delay between send and display.

### Tool results as structured UI components
Tool outputs (content calendar JSON, title score, etc.) are rendered as structured React components in the chat, not raw JSON blocks. The AI response references the tool output narratively — users get actionable artifacts without implementation details cluttering the conversation.

---

## API

```http
POST /api/chat
Content-Type: application/json

{
  "messages": [
    { "role": "user", "content": "My YouTube channel has 8k subs but views dropped 40% last month. What should I look at first?" }
  ],
  "tools": ["analyze_channel", "generate_content_calendar"]
}
```

Response: `text/event-stream` — streamed token deltas.

---

## Running This

```bash
npm install

npm run dev        # dev server
npm run build      # production build
npm run typecheck  # tsc --noEmit
```

See `.env.example` for required environment variables (`ANTHROPIC_API_KEY`).

---

## License

MIT — see [LICENSE](LICENSE)
