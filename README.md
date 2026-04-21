# Biggest Bro

**AI chat agent for YouTubers and musicians. Strategy, content ideas, and audience growth — powered by Claude.**

> An intelligent chat agent purpose-built for independent creators. Biggest Bro understands the YouTube and music industry ecosystem and gives specific, actionable advice — not generic tips. Ask about your next video concept, your release strategy, how to grow on a specific platform, or why your numbers are plateauing.

---

## Architecture

```
Browser
  │
  ▼
Next.js 15  (App Router)
  │
  ├── Chat UI  (React — optimistic updates, streaming)
  │
  └── /api/chat  (Edge API route)
        │
        └── Anthropic SDK  (claude-opus-4-7, extended thinking)
              │
              ├── System prompt  (creator-domain expert persona)
              ├── Conversation history  (managed client-side + server validation)
              ├── Tool use  (content calendar, trend lookup, analytics summary)
              └── Prompt cache  (system prompt + reference data, 5-min TTL)
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 15, App Router, React |
| AI | Anthropic Claude API (Opus 4.7) |
| Streaming | Server-Sent Events (SSE) via Anthropic SDK |
| Edge Runtime | Cloudflare Workers / Next.js edge |
| Caching | Anthropic prompt caching (system prompt + tool schemas) |
| Validation | Zod |

---

## Features

### Creator-Domain Intelligence
Biggest Bro has deep context about:
- YouTube algorithm signals (CTR, AVD, re-watch rate, click-through patterns)
- Music release strategy (timing, platform sequencing, pre-save campaigns)
- Audience growth levers specific to each platform
- Content formats that perform in niche communities
- Common mistakes indie creators make with their content strategy

### Tools (Function Calling)
Biggest Bro can use tools mid-conversation:

| Tool | Purpose |
|---|---|
| `generate_content_calendar` | Creates a 30-day content plan based on your niche and goals |
| `analyze_title` | Scores a YouTube title for CTR potential with suggestions |
| `compare_release_windows` | Evaluates release timing for music drops by day/season |
| `audience_growth_plan` | Generates a platform-specific growth roadmap |

### Conversation Management
- Full conversation history maintained client-side
- Server-side validation prevents history tampering
- System prompt cached at Anthropic — consistent persona, lower latency on every turn
- Extended thinking enabled for complex strategy questions

---

## Key Engineering Decisions

### Claude Opus 4.7 with extended thinking
Creator strategy questions often require multi-step reasoning — "why is my channel plateauing?" involves understanding algorithm signals, content patterns, upload frequency, and audience behavior simultaneously. Extended thinking produces more coherent, well-reasoned strategic advice than a fast-path response.

### Prompt caching for the system prompt
The system prompt (domain expert context, creator taxonomy, platform behavior patterns) is ~3,000 tokens. Caching it saves ~$0.03 per conversation turn at scale and reduces first-token latency from ~800ms to ~200ms.

### SSE streaming for conversational feel
Tokens stream progressively via Server-Sent Events. This is critical for long-form strategy responses — a 1,200-token response that streams feels fast and readable; the same response appearing all at once feels slow and overwhelming.

### Optimistic UI updates
Chat messages are added to the UI immediately on submit (before the API call returns). If the call fails, the message is rolled back with an error state. This eliminates the perceptible delay between send and display.

### Tool results injected as context, not shown raw
Tool outputs (e.g., content calendar JSON) are rendered as structured UI components in the chat, not as raw JSON. The AI response references the tool output narratively. This gives the user actionable artifacts without exposing implementation details.

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

## Security

- API route rate-limited per user session — prevents abuse
- Conversation history validated server-side (role sequence, length limits) — prevents prompt injection via history manipulation
- No user-generated content embedded directly into system prompt
- Tool inputs validated with Zod before execution
- No external API calls in tools without user-visible confirmation

---

## License

MIT — see [LICENSE](LICENSE)
