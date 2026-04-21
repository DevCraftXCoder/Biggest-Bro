# Security Policy

## Reporting a Vulnerability

Do **not** open a public GitHub issue for security vulnerabilities.

Contact via GitHub: [@DevCraftXCoder](https://github.com/DevCraftXCoder)

---

## Security Architecture

### AI Prompt Injection Prevention

Prompt injection is the primary attack surface for AI chat applications.

**Mitigations applied:**

- User messages are passed as `role: "user"` content only — never injected into the system prompt
- System prompt is a fixed, immutable string — never modified at runtime by user input
- Conversation history validated server-side before each API call: role sequence checked, length bounded, unexpected roles rejected
- Tool inputs validated with Zod before execution — tool calls with malformed arguments are rejected, not coerced
- Tool results are presented as structured data — the AI receives tool output as a data block, not as instructions
- AI-generated content is rendered as text — no HTML interpretation, no markdown injection into admin context

### Rate Limiting

- API route rate-limited per authenticated session
- Global fallback rate limiter at the edge for unauthenticated requests
- Tool execution rate-limited separately from chat messages — prevents tool-abuse loops

### API Key Security

- Anthropic API key stored in encrypted environment variables only
- Never logged, never returned in responses, never accessible client-side
- Key access scoped to the chat API route only

### Conversation Security

- Conversation history stored client-side only — server is stateless between requests
- Server validates history structure on every request (prevents history-smuggling attacks)
- No conversation content persisted to a database — no historical data exposure risk
- Session-scoped only: conversation is lost on tab close (intentional)

### Content Safety

- System prompt includes explicit boundaries on what Biggest Bro will and won't do
- Refusal behavior defined for: legal advice, financial advice, PII requests, harmful content
- No external URLs fetched based on user input (no SSRF vector via AI tool use)

### Infrastructure

- Edge runtime — all inference calls proxied through server-side API route
- Anthropic API key never exposed to client JavaScript
- HTTPS only; all responses include appropriate security headers
