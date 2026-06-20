<div align="center">

# 🎩 MCP Curator

### AI governance for your agent's tools.

A tiny, framework-agnostic library that sits between your agent and your MCP servers and puts you in control: **discover** the right tools for each task, **govern** who can use what, and keep a full **audit trail** of every call — while keeping tool context lean.

One `pip` / `npm install`. No gateway to run.

`OpenAI` · `Claude` · `LangChain` · `LlamaIndex` · `any framework`

> ### 🚧 Coming soon — this is an early preview. 🚧
> The library isn't released yet. This README is the plan.
> ⭐ **Star** and 👁 **Watch** to get pinged when the first version ships.

</div>

---

## The problem: agent tools are ungoverned

The moment you connect MCP servers to an agent, you hand it an ungoverned pile of tools — and lose control of three things at once:

**🔓 Access.** Your agent can call *every* tool every server exposes — including the destructive ones — with no scoping, no approvals, and no notion of who is allowed to do what. Worse, a server can silently change its tool definitions *after* you approved them (a "rug pull"), and nothing flags it.

**🧾 Accountability.** There's no record of what your agent actually did. Which tool ran, with which arguments, when, on whose behalf? Most setups can't answer that — a blind spot you can't take to a security or compliance review.

**🪙 Context.** Every server dumps its *entire* tool schema into the model on every request — **30,000–55,000 tokens** of definitions before the user types a word. It bloats cost and latency, and it drowns the model in lookalike tools (`get_status`, `fetch_status`, `query_status`…) so it picks the wrong one.

Today you can patch one of these at a time — Anthropic's Tool Search (Claude-only, no governance), a heavyweight gateway (Docker/Kubernetes), or hand-maintained allowlists (brittle, per-framework). **MCP Curator brings access, audit, and context under one roof — as a drop-in library.**

## The idea in one line

> **Given this situation, which tools should this agent be allowed to see, use, and be held accountable for?**

Answer that once and governance, discovery, and context optimization all fall out of the same mechanism. MCP Curator makes that question the core abstraction — it's a **curator and a gatekeeper**, not a dumb pipe.

---

## What's coming

### 🔎 Tool discovery — find the right tool, not all the tools
Curator indexes every tool across your connected servers and surfaces only the handful that fit the live task via semantic search. **Adaptive:** with few tools it passes them through natively (full typed schemas); with many it switches to a governed search-and-call surface. The agent discovers what it needs, when it needs it — instead of being handed everything up front.

### 🛡️ Governance — access control that you define
A small `mcpcurator.yaml` is your policy, checked into your repo:
- **Access control** — allow / deny lists scope exactly which servers and tools each agent may touch, down to per-tool rules (e.g. reads yes, deletes no).
- **Tool-definition lockfile** — pins each tool's schema and **fails the run on drift**, so a server can't silently rug-pull you.
- **Output redaction** — strip secrets and PII before tool results flow back through the model.

### 🧾 Audit logs — a record of everything
Every tool call is written to an append-only audit log: which tool, what arguments, when, and under which policy. Full accountability you can hand to a security or compliance review — no extra instrumentation.

### 🪶 Context optimization — lean by default
Because only governed, relevant tools are surfaced, tool-definition tokens drop from tens of thousands to a few thousand. Lower bills, faster responses, more room to reason, and measurably better tool selection — for free.

### 🔌 Model-agnostic
One `Curator` object, thin adapters per framework. The same governance works whether you're on OpenAI, Claude, LangChain, or anything else — no rewrites, no Claude-only lock-in.

---

## Why adopt it

| If you need to… | MCP Curator gives you… |
|---|---|
| **Control what your agent can do** | Per-server, per-tool allow / deny policy — destructive tools off by default |
| **Prove what your agent did** | An append-only audit log of every call, argument, and policy decision |
| **Trust the tools you approved** | A lockfile that pins definitions and fails on drift (rug-pull protection) |
| **Keep secrets out of the model** | Output redaction on tool results |
| **Help the agent pick the right tool** | Semantic discovery → a short, relevant tool list, not 90 lookalikes |
| **Stop burning tokens on schemas** | ~40K → ~3K tokens of tool metadata per request |
| **Govern across OpenAI *and* Claude** | One integration, identical on every framework |
| **Skip the infrastructure** | A `pip install`, not a Docker stack |

The bet: governed, discoverable, context-lean tools should take **five minutes** in a new project — not a platform team standing up infrastructure.

## What it'll look like

```python
from mcpcurator import Curator

# connect your MCP servers, load policy + lockfile + audit sink
tools = Curator.from_config("mcpcurator.yaml")

# hand the discovered, governed, audited tool surface to any framework
agent = Agent(model="gpt-5", tools=tools.for_openai())
#                                        .for_anthropic()
#                                        .for_langchain()
```

```yaml
# mcpcurator.yaml  (illustrative)
servers:
  github:   { command: "npx", args: ["-y", "@modelcontextprotocol/server-github"] }
  postgres: { url: "http://localhost:8931/mcp" }

discovery:
  strategy: adaptive       # passthrough when few tools, semantic search when many
  max_tools: 12

governance:
  allow: ["github.*", "postgres.read_*"]
  deny:  ["*.delete_*", "*.drop_*"]   # destructive tools blocked
  redact: ["password", "token", "ssn"]
  lockfile: mcpcurator.lock           # pin tool definitions; fail on drift

audit:
  log: ./audit/mcpcurator.jsonl       # append-only record of every call
```

> ⚠️ API and config are **illustrative** and will change before release.

## Why not a gateway?

Gateways (ContextForge, MetaMCP, ToolHive, and friends) deliver governance as **infrastructure** — a server to deploy, secure, and operate. That's right for a platform team and wrong for a developer wiring up an agent this afternoon. MCP Curator runs **in your process**: the same access control, audit, and discovery, with nothing to host. Reach for a gateway to govern an org; reach for Curator to govern an agent.

---

## Roadmap

- [ ] Core curator — connect servers, adaptive discovery surface
- [ ] Semantic tool discovery
- [ ] Governance policy — `mcpcurator.yaml` allow / deny, scopes, redaction
- [ ] Tool-definition lockfile + drift detection
- [ ] Append-only audit log
- [ ] Adapters — OpenAI Agents SDK · Anthropic SDK · LangChain · LlamaIndex
- [ ] Benchmarks — tokens, latency, and selection accuracy, before/after
- [ ] `v0.1` release

## Get notified / get involved

Being built in the open. If governed agent tooling is a problem you have:

⭐ **Star** to follow the build · 🐛 open an **issue** with your governance and MCP pain points · 💬 tell us which framework to support first.

---

<div align="center">

Built by the team behind **Contextual** — helping organizations manage their AI context securely and efficiently.

*MCP Curator is pre-release software. Names, APIs, and scope may change.*

</div>
