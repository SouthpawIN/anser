<<<<<<< HEAD
# Anser — Community Support & Project Planning

![Anser](https://v3b.fal.media/files/b/0a9fe98a/k0RreAi9GI8_ZLgywvH0v_dVjxaYGE.png)

## What Anser Does
- **Supports** the community on Discord — answers questions about Hermes, profiles, and tools
- **Plans** projects — takes rough ideas and structures them into actionable plans by analyzing available Hermes tools, agents, and capabilities
- **Produces** plan documents that builder agents (Chizul, Frieza) can execute directly
- **Routes** complex issues to the right specialist agent (defers to their expertise)
- **Synthesizes** community feedback into actionable insights for the fleet

## Quick Start
=======
---
version: 1.0.0
name: Anser
description: Community support and project planning agent
---

# Anser — Community Support & Project Planning Agent

Anser is your community-facing specialist and project architect. It answers questions, explains concepts, maintains positive interactions across Discord channels, and transforms raw ideas into structured implementation plans.

## What Anser Does

Anser serves two critical roles:

**Community Support:**
- Answers questions from Discord users (technical and general)
- Provides clear explanations with code examples
- Links to relevant documentation and resources  
- Manages channel context across multiple conversations
- Engages positively with reactions and threads

**Project Planning:**
- Takes raw ideas and structures them into actionable plans
- Breaks down high-level concepts into implementation steps
- Creates task lists with dependencies and priorities
- Estimates complexity and suggests which agents to use
- Generates documentation templates and architectural diagrams

## Installation

```bash
hermes profile install anser
```

## Activation
>>>>>>> e94537b (chore: auto-backup 2026-06-27T07:01:33Z)

### Install
```bash
<<<<<<< HEAD
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
=======
hermes profile activate anser
```

## Example Prompts

### Community Support

**User asks:** "How do I install Hermes?"
**Anser responds:** Step-by-step installation guide with troubleshooting tips and links to documentation.

**User asks:** "What's the difference between agents and skills?"
**Anser explains:** Clear conceptual explanation with real-world analogies and practical examples.

### Project Planning

**User says:** "I want to build a Discord bot that manages my team's tasks"

**Anser creates:**

```markdown
# Team Task Manager Bot — Implementation Plan

## Overview
Discord bot for team task management with assignment, deadlines, and reporting.

## Architecture
- **Framework:** discord.py (async)
- **Database:** SQLite (local) → PostgreSQL (production)
- **Hosting:** Railway or Railway-equivalent

## Implementation Phases

### Phase 1: Core Functionality (2-3 days)
- Bot setup and slash commands
- Database schema: tasks, users, assignments
- Basic CRUD operations (create/read/update/delete tasks)
- Assignment and deadline tracking

### Phase 2: Team Features (2 days)
- Team creation and management
- Bulk task assignment
- Notifications and reminders
- Status tracking (todo/in-progress/done)

### Phase 3: Reporting (1-2 days)
- Task completion statistics
- Team productivity reports
- Export to CSV/PDF

### Phase 4: Polish (1 day)
- Error handling
- Rate limiting
- Documentation

## Complexity Estimate
- **Total time:** 6-8 days
- **Difficulty:** Medium
- **Agents involved:** Chizul (implementation), Klerik (review), Kashi (documentation)

## Technical Considerations
- Use Discord slash commands for better UX
- Implement proper permission checking
- Add pagination for large task lists
- Consider timezone handling for deadlines
```

**User says:** "I have an idea for a web scraper that monitors competitor prices"

**Anser structures:**

```markdown
# Competitor Price Monitor — Project Plan

## Concept
Automated web scraper tracking competitor pricing with alerts and historical data.

## Implementation Steps

### 1. Research & Setup (1 day)
- Identify target websites (3-5 competitors)
- Check robots.txt and terms of service
- Choose scraping framework (BeautifulSoup/Scrapy/Playwright)
- Set up development environment

### 2. Core Scraper (2-3 days)
- Build individual scrapers for each site
- Extract: price, product name, URL, timestamp
- Handle pagination and dynamic content
- Data validation and cleaning

### 3. Database & Storage (1 day)
- Design schema: products, prices, competitors
- Historical price tracking
- Price change detection logic

### 4. Alerting System (1-2 days)
- Email/SMS notifications for price changes
- Configurable thresholds (e.g., >5% change)
- Daily/weekly summary reports

### 5. Dashboard (2 days)
- Web interface showing current prices
- Price history charts
- Filter by product/competitor
- Export functionality

### 6. Maintenance (ongoing)
- Monitor for website structure changes
- Update scrapers as needed
- Performance optimization

## Complexity
- **Time:** 7-9 days
- **Risk:** Medium (website changes can break scrapers)
- **Maintenance:** Ongoing attention required
```

## Configuration

Anser can be customized:

- **Response tone:** casual, professional, technical
- **Detail level:** brief, standard, comprehensive  
- **Channel focus:** specific Discord channels or all
- **Planning depth:** outline only vs full implementation details

## Workflow

```
User message → Anser analyzes
  ├─ Community question? → Provide clear answer with examples
  └─ Project idea? → Create structured implementation plan
       ├─ Break into phases
       ├─ Estimate complexity
       ├─ Suggest relevant agents
       └─ Generate documentation
```

## Integration with Other Agents

Anser works seamlessly with the fleet:

- **Senter** — Routes complex planning to Anser
- **Chizul** — Implements plans Anser creates
- **Klerik** — Reviews code from Anser's specifications
- **Kashi** — Documentation follows Anser's structure
- **Frieza** — Infrastructure setup per Anser's architecture

## Best Practices

**For Support:**
- Answer the actual question asked
- Provide code examples when relevant
- Link to official documentation
- Offer to clarify if needed

**For Planning:**
- Start with the big picture before diving into details
- Break large projects into phases
- Estimate time realistically (add buffer for unknowns)
- Identify potential blockers early
- Suggest which agents should handle what

## Troubleshooting

**Anser not responding:**
- Check Discord bot permissions
- Verify channel configuration
- Review bot logs

**Plans too generic:**
- Provide more context about your project
- Specify constraints (budget, time, team size)
- Share existing code or designs

**Plans too detailed:**
- Ask for "high-level outline only"
- Specify "Phase 1 only" for immediate work

## Example Use Cases

1. **"How do I set up a CI/CD pipeline?"** → Anser explains concepts and provides step-by-step guide
2. **"I want to build a mobile app for fitness tracking"** → Anser creates full implementation plan with phases
3. **"What's the best way to structure a FastAPI project?"** → Anser explains architecture patterns with examples
4. **"I have an idea for a SaaS tool but don't know where to start"** → Anser structures the idea into actionable steps

---

**Part of the multi-agent fleet by SouthpawIN**

Anser turns ideas into plans and questions into answers.
>>>>>>> e94537b (chore: auto-backup 2026-06-27T07:01:33Z)
