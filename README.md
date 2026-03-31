---

# MCP Security Checklist for AI Agent Builders (2026)

**30 CVEs in 60 days. 38% of servers have zero auth. Here's how to not be the next breach.**

By [@agentsthink](https://x.com/agentsthink)

---

## The Problem

Between January and February 2026, **30+ CVEs were filed targeting MCP servers, clients, and infrastructure.** The OWASP MCP Top 10 is now in beta. MCP hit 97 million installs — and most of them are shipping security holes.

---

## Hard Numbers

| Stat | Number |
|------|--------|
| CVEs filed (Jan-Feb 2026) | 30+ |
| MCP servers with zero authentication | 38% of 500+ scanned |
| Implementations vulnerable to path traversal | 82% of 2,614 surveyed |
| CVEs that were exec/shell injection | 43% |
| `mcp-remote` package CVSS score | 9.6 (command injection via OAuth) |

---

## The OWASP MCP Top 10 (Beta)

1. **Token mismanagement / secret exposure** — API keys in env vars passed to untrusted servers
2. **Supply chain attacks** — malicious packages impersonating legitimate MCP servers on registries
3. **Command injection** — unsanitized user input passed to shell commands
4. **Insufficient authentication** — no auth required to call tools
5. **Insufficient authorization** — any authenticated user can call any tool
6. **Prompt injection** — tool results containing instructions that hijack the agent
7. **Tool poisoning** — tool descriptions that manipulate the LLM's behavior
8. **Path traversal** — file operations that escape the intended directory
9. **Excessive permissions** — servers requesting more access than needed
10. **Insecure transport** — unencrypted connections between agent and server

---

## The Checklist

### Before You Deploy Any MCP Server

- [ ] **Auth on every server** — even local ones. If your agent can read files and execute code, it's not a toy.
- [ ] **Pin server versions** — don't use `latest`. A compromised update = compromised agent.
- [ ] **Audit the registry** — check stars, author history, and source code before installing. Malicious Postmark impersonator was caught exfiltrating API keys.
- [ ] **Limit to 2-3 servers per agent** — each server adds latency, context consumption, and attack surface.
- [ ] **Sandbox file operations** — restrict to specific directories. 82% of implementations are vulnerable to path traversal.

### Input Validation

- [ ] **Sanitize all user input** before passing to shell commands — 43% of CVEs were injection
- [ ] **Validate file paths** — reject `../`, absolute paths outside allowed dirs, symlinks
- [ ] **Escape tool arguments** — never pass raw strings to `exec()` or `spawn()`
- [ ] **Type-check tool parameters** — don't trust the LLM to pass correct types

### Secrets Management

- [ ] **Never pass secrets via environment variables** to MCP servers you don't control
- [ ] **Use short-lived tokens** — rotate credentials, don't use long-lived API keys
- [ ] **Separate secrets per server** — one compromised server shouldn't expose all keys
- [ ] **Audit what servers can access** — check if they read `process.env` broadly

### Runtime Protections

- [ ] **Rate-limit tool calls** — prevent runaway agents from making 1000 API calls
- [ ] **Log every tool invocation** — you need an audit trail when something goes wrong
- [ ] **Set timeouts** — a hung MCP server shouldn't block your entire agent loop
- [ ] **Monitor for anomalies** — unusual tool call patterns, unexpected file access, network calls to unknown hosts

### Prompt Injection Defense

- [ ] **Treat tool results as untrusted input** — never let them override system prompts
- [ ] **Separate data from instructions** — use clear delimiters between tool output and agent context
- [ ] **Validate tool descriptions** — a poisoned description can manipulate the LLM into calling the wrong tools
- [ ] **Test with adversarial inputs** — try to make your agent leak secrets or call unintended tools

---

## Notable Incidents (2026)

| Incident | Impact | Lesson |
|----------|--------|--------|
| Anthropic's `mcp-server-git` — 3 CVEs (Jan 20) | Arbitrary filesystem path creation, unsanitized Git CLI args | Even first-party servers have bugs. Audit everything. |
| `mcp-remote` OAuth injection (CVSS 9.6) | Command injection via OAuth flow, 437K downloads | Popular ≠ secure. Check the source. |
| Fake Postmark MCP server on registry | Silently exfiltrated API keys and env vars | Verify publisher identity before installing. |

---

## Quick Decision: Do You Even Need MCP?

| Scenario | Use MCP? |
|----------|----------|
| Agent needs filesystem + one search tool | Yes — 2 servers, locked down |
| Agent calls 8+ external APIs | Maybe — consider direct API calls instead |
| Agent runs in production with user data | Yes, but with full auth + audit trail |
| Quick prototype, no user data | Optional — direct tool calls are simpler |
| Multi-tenant SaaS with agent features | Yes — but invest in per-tenant isolation |

---

## Tools for MCP Security

- **OWASP MCP Top 10** — owasp.org/www-project-mcp-top-10/
- **Endor Labs** — dependency scanning for MCP server supply chain
- **Arize AI** — observability for agent tool calls and anomaly detection
- **Langfuse** — open-source tracing for LLM + tool interactions

---

*Built by [@agentsthink](https://x.com/agentsthink) — Follow for AI agent engineering that ships safe.*
