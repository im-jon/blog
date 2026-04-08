---
title: "Stop wrapping REST APIs and calling it an MCP server. Do this instead."
date: 2026-04-07T00:00:00Z
---

Most MCP servers are slapped together REST API wrappers with a new coat of paint. They technically work. They also confuse agents, bloat context, and quietly underperform until you give up and blame the model.

Here's what I've learned building servers that don't do that.

---

### Give your server a brain with Instructions

The MCP spec lets you set an `instructions` property on your server. Use it. This is where you tell the agent *how to use your server* — which resource to load first, what the expected workflow is, what not to do.

Example: "Invoke the `user_context` resource before calling anything else."

One caveat: some clients (ChatGPT, looking at you) don't support resources at all. A `get_user_context` tool covers those cases. You can even write client-specific instructions, though keep in mind the agent might not know what client it's running in. Don't overthink it — write for the common case first.

---

### Your tools can talk to each other

MCP is not REST. You don't need to answer everything in one shot.

Tools can instruct the agent to call other tools. This is underused and it's great:

- "Call `test_sql_query` first. If the result is valid, call `execute_sql_query`."
- "This user has 23 deals last quarter. Call `fetch_user_deals` for the full picture."

This lets agents break work into smaller steps, react to unexpected results, and make decisions as they go — instead of guessing upfront with incomplete context.

---

### Return a text summary alongside structured data

A lot of people build an MCP server by wrapping a REST API, mapping each endpoint to a tool, and calling it a day. The agent technically gets data. It just has no idea what to do with it.

When your tool returns structured content, add a dynamically generated text summary alongside it. Explain what the result means. Tell the agent what to do next — "result was empty, try `search_users` with broader filters" or "found 3 matches, call `get_user_details` to resolve."

This keeps tool descriptions short (they're always loaded in context), and it gives the agent exactly the nudge it needs at exactly the right moment.

Honestly? For MCP, plain contextual text often beats JSON outright. Your consumer is a language model, not a frontend. Act accordingly.

---

### Not all clients are the same

Seriously. The MCP spec is unevenly implemented across clients, and this will bite you.

- Some clients only support tools (ChatGPT, still, yes).
- Some can render visual elements via the MCP Apps extension; others are terminal-only (Claude Code).
- Some agents can call multiple tools per turn; some are limited to one.
- Some run frontier models; some run small local models to save money.

If you're only testing with the latest model on the market, you're not seeing what your users see.

You *can* write instructions that try to accommodate all of them — but you'll probably muddy things for everyone. Better approach: test with multiple clients and models, find where things break, and adjust surgically. Trust the model to handle the parts you haven't explicitly covered. It usually can.

---

### Make your parameters wider

Fetching a resource in REST is easy. You call a GET endpoint with an ID. Done.

In MCP, the agent has to get there from natural language. "Get me Jonathan's profile" means your server needs to figure out who Jonathan is before it can do anything.

One pattern I like: let the model fill in what it *knows* about the target resource directly in the parameters. For a user lookup, that might be `user_full_name`, `user_email`, and `user_id` — all optional, all resolved server-side based on what's filled in. No extra resolution tool call. Fewer round trips. Less context burned. The tool just works.

---

### Prune your tools. Aggressively.

No one is going to use a server with 250 tools. (Hi, GitHub MCP.) It eats context and paralyzes the agent with options.

Some people solve this with dynamic toolsets / tool search. That's fine, but if you need it, it's probably a symptom that your server is bloated in the first place.

The less painful fix: combine related tools, add telemetry, and kill what nobody's using. You'll be surprised how few tools you actually need.

---

### Document for the humans driving the agents

MCP servers are agent-facing, but people set them up. People decide what to do with them. People hit walls when something doesn't work and have no idea why.

The protocol itself makes it hard to discover what a server can do (SEP-1649, Server Cards, will eventually help, but we're not there yet). So document it yourself. Write down what your server can and can't do. Cover the gotchas. Mention the technical limits. Let users know what skills they can build with it.

The agent might not read your docs. The person building with your server definitely will.