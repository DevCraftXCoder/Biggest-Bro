# Biggest Bro

![Next.js](https://img.shields.io/badge/Next.js_15-000000?style=flat&logo=nextdotjs&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white)
![LLM Powered](https://img.shields.io/badge/LLM_Powered-D97706?style=flat&logo=anthropic&logoColor=white)

**AI co-pilot for independent creators — LLM extended thinking, tool use, domain expertise.**

> Domain-expert AI assistant for YouTubers, musicians, and independent creators. Uses LLM extended thinking with an 8,000-token budget and tool use to reason through creator-specific strategy — content calendars, trend analysis, monetization paths, audience growth.

## Architecture

```
React Chat UI
  └── /api/chat edge route
        └── LLM API (extended thinking)
              ├── System prompt (creator domain expertise, 5-min cache TTL)
              ├── Tool use: content calendar, trend analysis, analytics summary
              └── Conversation history (client-managed, server-validated per turn)
```

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 15 (App Router, edge runtime) |
| AI | LLM (Opus tier) with extended thinking (8,000-token budget) |
| Prompt caching | 5-min TTL on system prompt + creator expertise persona |
| Tool use | Content calendar generation, trend analysis, analytics summary |
| Language | TypeScript |

## How the AI Works

- **Model: LLM (Opus tier)** with extended thinking — reasons through multi-step creator decisions before responding
- **Thinking budget: 8,000 tokens** — allows deep analysis of content strategy, monetization, and growth
- **System prompt cached** at 5-min TTL — creator domain expertise persona loaded once per session
- **Tool use** — 3 registered tools: content calendar generator, trend analyzer, analytics summarizer
- **Conversation history** managed client-side + validated server-side on each turn
- **Fallback** — drops extended thinking on budget-exceeded to maintain responsiveness

## Domain Expertise

- YouTube algorithm signals and upload timing
- Music release strategy (DSP, pre-save, editorial pitching)
- Audience growth and retention analysis
- Creator monetization paths (subscriptions, merch, brand deals, sync licensing)
- Content calendar planning and cross-platform scheduling

