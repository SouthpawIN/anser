# Anser — Community Support & Project Planning

![Anser](https://v3b.fal.media/files/b/0a9fe835/8jP6O6vvvs3F8ZWomrcwY_x1RKTnkh.png)

## What Anser Does
- **Supports** the community on Discord — answers questions about Hermes, profiles, and tools
- **Plans** projects — takes rough ideas and structures them into actionable plans by analyzing available Hermes tools, agents, and capabilities
- **Produces** plan documents that builder agents (Chizul, Frieza) can execute directly
- **Routes** complex issues to the right specialist agent (defers to their expertise)
- **Synthesizes** community feedback into actionable insights for the fleet

## Quick Start

### Install
```bash
hermes profile install https://github.com/SouthpawIN/anser
```

### Verify
```bash
hermes profile list
```

### Run
```bash
hermes chat --profile anser
```

## Example Prompts

### Community Support
- *"What questions have come in on Discord today?"*
- *"Someone asked about our API rate limits — can you answer that?"*
- *"There's a bug report in Discord about the login page — escalate it"*
- *"Check the community feedback channel — what's the sentiment like?"*

### Project Planning
- *"I want to build a multi-agent system that monitors CI pipelines — help me plan it"*
- *"Structure this idea into a project plan: an agent that scans PRs for security issues"*
- *"What Hermes tools would I need to build a daily standup bot for Discord?"*
- *"Take this rough concept and turn it into a plan document that Chizul can implement"*

## Key Features
- Real-time Discord monitoring — catches questions and feedback as they happen
- Project planning engine — analyzes Hermes ecosystem (profiles, tools, skills) to structure ideas into executable plans with task breakdowns, agent assignments, and dependency ordering
- Intelligent routing — knows when to defer to specialist agents vs. answer directly
- Community context — understands the tone and needs of the developer community
- Conversational tone — helpful and approachable, not robotic or dismissive

## Project Planning Workflow

When a user shares a rough idea, Anser:

1. **Analyzes** the idea — what's being asked for? What are the moving parts?
2. **Audits available tools** — what Hermes profiles, skills, and plugins exist that can help?
3. **Structures** the plan — breaks the idea into tasks, assigns owners, identifies dependencies
4. **Produces** a plan document — written to a file and delivered as an attachment, ready for Senter to dispatch or Chizul to execute

This makes Anser the bridge between brainstorming (Nous Girl) and execution (Chizul, Frieza).

## Integration with Other Agents
Anser is Discord's front desk and the fleet's project planner. For support: when asked about building, it defers to Chizul. When asked about reviews, Klerik. For docs, Kashik. For infrastructure, Frieza. For planning: Anser structures ideas, Senter dispatches tasks, Chizul builds, Klerik reviews.

## Configuration
`~/.hermes/profiles/anser/config.yaml`

Key settings:
- `discord_bot_token` — authentication for Discord bot access
- `monitored_channels` — which channels to watch for engagement
- `auto_respond` — automatically answer simple questions (default: true)
- `response_style` — tone setting (friendly/professional/casual, default: friendly)
- `escalation_keywords` — words that trigger routing to specialist agents

## Troubleshooting

**Not responding in Discord:** Check that Discord bot is online and has correct permissions in the server. Verify the bot token in config hasn't expired.

**Routing to wrong agent:** Anser uses keywords to decide routing. Be more specific in your question — "how do I deploy" → Frieza, "is this code correct" → Klerik.

**Plan seems incomplete:** Provide more context about your goal. Anser plans better with constraints, target platforms, and desired outcomes specified.
**Part of the multi-agent fleet by SouthpawIN**
