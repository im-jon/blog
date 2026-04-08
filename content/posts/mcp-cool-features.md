---
title: "MCP Has More Than Tools. Nobody's Using It."
date: 2026-04-06T00:00:00Z
---

Everyone talks about MCP tools. The spec has features almost nobody uses, and a few of them are genuinely good.

MCP launched in late 2024. In AI terms, that's ancient history. Agents were barely a concept. We were still figuring out single-turn task completion.

The underused features from that era are arguably *more* useful now that agents are doing real work.

Here's what you're probably skipping.

---

## Elicitation: ask the user mid-tool

Most tools either work or error. They can't ask for clarification.

Elicitation lets a server pause a tool call, prompt the user, and resume once it has an answer. Simple mode: the server defines a schema, the user fills it out, the tool continues. URL mode: more interesting. A tool that needs Slack access can kick off an OAuth flow right there instead of failing because you forgot to connect it beforehand. No pre-configuration.

ChatGPT doesn't support this.

**Spec:** https://modelcontextprotocol.io/specification/draft/client/elicitation

---

## Sampling: the server talks to the model

Normally the client drives everything.

Sampling flips it. The server sends a request directly to the model, bypassing your system prompt and conversation history. The obvious use is structured output — need a well-formed object without context bleed? Here's your mechanism.

The problem: it's limited to the LLM layer, not the agent. So if your server needs to fetch all the Salesforce deals during a performance review, sampling can't help you there. But this could be a nice improvement to the MCP protocol.

Bonus: sampling responses include the client's model name. If you want your server to behave differently for GPT-5 vs Claude, you can use this to tell them apart.

ChatGPT doesn't support this either.

**Spec:** https://modelcontextprotocol.io/specification/draft/client/sampling

---

## Prompts: reusable workflows the user can invoke

MCP prompts aren't instructions for the agent. They're user-invoked templates — typically surfaced as slash commands.

Think of them as server-shipped skills. `/summarize`, `/review`, `/generate-test-cases` — the server defines them, the client surfaces them, the user triggers them. The agent plays no role.

That's also the main limitation. No standard way for an agent to invoke these autonomously. There's no opt-in mechanism in the spec yet. Client developers can add it themselves — some probably have — but it's a real gap. You probably want a human in that loop anyway. It just shouldn't be mandatory.

Still not supported by ChatGPT.

**Spec:** https://modelcontextprotocol.io/specification/draft/server/prompts

---

## Logging: see what the server is actually doing

Tool calls are a black box. Input goes in, output comes out. Everything in between is invisible.

Logging lets the server emit structured messages during execution. For the agent, this is live feedback: a failing tool call doesn't have to wait for a final error to know something is wrong. It can read the logs and retry with different parameters. For humans, it's basic observability over a system that's otherwise opaque.

You know — a debugger. For your AI stack.

ChatGPT doesn't support this.

**Spec:** https://modelcontextprotocol.io/specification/draft/server/utilities/logging

---

## The takeaway

None of this is exotic. Elicitation solves real auth friction. Sampling enables server-side orchestration that otherwise isn't possible. Prompts give server developers a way to ship reusable workflows. Logging makes the whole stack debuggable.

The spec thought about more than tools.

Most clients just haven't caught up yet.