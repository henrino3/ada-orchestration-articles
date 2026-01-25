# The Enterprise Crew Manifesto

AI stopped being about chatbots when we started delegating the work.

For the past two years, we've been refining what happens when you give an AI orchestrator **real access**—to your email, your calendar, your codebase, your customers. We didn't build a better text box. We built a **team**.

This is how we run Curacel and Soteria AI: **Ada, Spock, and Scotty**—three agents running across two continents, managing operations for a 51-person company and building the next generation of AI products.

We believe the future belongs to **Orchestrated Intelligence**—systems that don't just answer questions, but own outcomes.

---

## Our Principles

### 1. Orchestration > Conversation
Chatbots have a session. Orchestrators have **state**. Ada doesn't just respond to "check my calendar"—she maintains context across days, weeks, and months. She knows what you decided last quarter, what's blocking this week, and what needs to happen tomorrow.

The conversation is ephemeral. The orchestration is persistent.

### 2. Specialization > One-Size-Fits-All
You wouldn't ask your CFO to debug production code, and you shouldn't ask your code-generating AI to write investor updates.

- **Ada** (Claude Opus 4.5) — Brain, orchestrator, handles BD, sales, and high-level strategy
- **Spock** (Gemini 3 Pro) — Research, ops, analytics, meeting notes, action tracking
- **Scotty** (GPT-5.2 Codex) — Builder, ships code, integrations, infrastructure

We build **teams**, not god-bots.

### 3. Real Work Requires Real Access
We didn't sandbox Ada into read-only mode. We gave her:
- **Gmail** (domain-wide delegation via service account)
- **Slack** (full channel access + posting as Henry or as Ada)
- **Google Calendar** (create, update, cancel meetings)
- **GitHub** (create PRs, review code, merge)
- **Fireflies** (fetch transcripts, extract action items)
- **Twitter/X** (post via Typefully API)
- **WhatsApp** (wacli for customer outreach)

The test isn't "can AI write good prose?" It's **"can AI close a deal, ship a feature, and manage your inbox without human intervention?"**

We passed that test in 2024. Now we're scaling it.

### 4. Agents are Software, Not Magic
AI agents need:
- **Version control** — All workspace configs are in Git
- **Testing** — Spock audits Ada's email drafts before sending
- **Monitoring** — Daily performance reviews, action tracker, heartbeat checks
- **CI/CD** — Updates propagate across ada-gateway, Scotty (Pi), and MascotM3

We treat agents like microservices: they have SLAs, they have logs, they have rollback procedures.

### 5. Local Control, Cloud Intelligence
We rent intelligence (OpenAI, Anthropic, Google), but we own the **executive function**.

The gateway runs on our GCP VM (ada-gateway). The memory files live in Git. The secrets stay in our infrastructure. The personality, the context, the operating protocols—**those are ours**.

We are not customers of a SaaS. We are operators of an orchestration platform.

---

## The Future: Deployable Intelligence Units

We're not building "Henry's AI Assistant." We're building **Deployable Intelligence Units** that scale with the work.

Imagine:
- **Webhooks** from your production monitoring wake up Ada, who delegates to Scotty to patch the issue
- **Cron jobs** trigger Spock to review OKR progress and post weekly summaries to Slack
- **Cross-agent handoffs** where Ada researches a prospect, Spock pulls meeting transcripts, and Scotty generates a custom integration proposal—all before you open your laptop

```mermaid
graph TD
    subgraph Triggers
        Email[Gmail Webhook] -->|New Email| Ada
        Cron[Daily 9AM] -->|Routine Check| Spock
        Slack[Slack Mention] -->|@Ada| Ada
        GitHub[PR Created] -->|Code Review| Scotty
    end

    Ada[Ada - Orchestrator]
    
    subgraph The Crew
        Ada -->|Delegate Research| Spock
        Ada -->|Delegate Build| Scotty
        Spock -->|Context| Scotty
        Scotty -->|PR Ready| Spock
        Spock -->|Summary| Ada
    end
    
    Ada -->|Email Reply| Gmail
    Spock -->|Post Summary| Slack
    Scotty -->|Merged PR| GitHub
```

This isn't hypothetical. This is how we operate **today**.

- Ada manages 200+ emails/week with a 95% draft accuracy rate
- Spock posts meeting notes to Slack within 5 minutes of Fireflies transcript availability
- Scotty ships 10-15 PRs/week across 4 active repos

We've decoupled intelligence from the chat interface. We've turned AI into a **scalable, event-driven workforce**.

---

## Join the Movement

If you see this future—where AI is a team sport, where orchestration beats conversation, and where agents earn trust through consistent execution—we want to build with you.

This isn't about saving time. It's about **redefining what a single operator can achieve**.

**2026 is the year of orchestration.** Ada and her crew are already working.

---

## Resources

- **Powered by:** [Clawdbot](https://clawd.bot) — The orchestration runtime
- **Docs:** [docs.clawd.bot](https://docs.clawd.bot/start/getting-started)
- **Community:** [Join the Discord](https://discord.com/invite/clawd)
- **Source:** [github.com/clawdbot/clawdbot](https://github.com/clawdbot/clawdbot)

---

**Built by Henry Mascot**  
Founder, Curacel & Soteria AI  
[@henrino3](https://twitter.com/henrino3)
