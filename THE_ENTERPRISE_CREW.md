# The Enterprise Crew: Architecture of a Multi-Gateway AI Workforce

This is the technical companion to the Enterprise Crew Manifesto. It explains **how the system is wired**: gateways, agents, cross-gateway communication, tools, skills, and the operational protocols that make a distributed AI team reliable.

> **Running Curacel (51 employees, 40+ insurers) and building Soteria AI with orchestrated agents.**

> Note: Configuration details are summarized. Secrets and API keys are redacted.

---

## 0) The Reality Check

We're not running a demo. We're running a business.

- **200+ emails/week** handled by Ada with 95% draft accuracy
- **10-15 PRs/week** shipped by Scotty across 4 repos
- **Meeting notes** posted to Slack within 5 minutes of transcript availability
- **Action items** tracked automatically across 3 Slack channels

This isn't vaporware. This is production infrastructure managing real operations for a distributed team across Africa, Middle East, and Latin America.

---

## 1) Architecture at a Glance

We run a **multi-gateway, multi-agent system** across three infrastructure nodes:

### **ada-gateway** (GCP VM - 100.106.69.9)
- **Agents:** Ada (Opus), Spock (Pro)
- **Channels:** Telegram, WhatsApp, Slack
- **Primary role:** Orchestration, customer-facing work, BD/sales

### **castlemascot-r1** (Raspberry Pi 5 - 100.68.207.75)
- **Agent:** Scotty (Codex)
- **Primary role:** Code generation, infrastructure work, builds
- **Why Pi:** Demonstrates edge deployment capability

### **MascotM3** (Mac - 100.86.150.96)
- **Node companion** for camera, notifications, iMessage, local files
- **ChromaDB server** (Ada Deep - semantic search across ChatGPT history, Obsidian vault)
- **Obsidian Vault web server** for instant file access

### System Diagram

```mermaid
flowchart TD
    subgraph Inputs
        TG[Telegram]
        WA[WhatsApp]
        SL[Slack]
        EM[Gmail]
        HK[Webhooks]
    end

    subgraph GW1[ada-gateway - GCP]
        Ada[Ada<br/>Opus 4.5<br/>Orchestrator]
        Spock[Spock<br/>Gemini Pro<br/>Research & Ops]
    end

    subgraph GW2[castlemascot-r1 - Pi]
        Scotty[Scotty<br/>GPT 5.2 Codex<br/>Builder]
    end

    subgraph Node[MascotM3 - Mac]
        Chroma[ChromaDB<br/>Ada Deep]
        Vault[Obsidian Vault<br/>Web Server]
    end

    TG --> Ada
    WA --> Ada
    SL --> Ada
    EM --> Ada
    HK --> Ada

    Ada -->|sessions_send| Spock
    Ada -->|Cross-gateway hook| Scotty
    Spock -->|SSH queries| Chroma
    Scotty -->|SSH queries| Chroma

    Ada -->|Email| EM
    Spock -->|Slack posts| SL
    Scotty -->|Git push| GitHub
```

---

## 2) The Roster: Meet the Crew

### **Ada 🔮 (The Orchestrator)**
- **Model:** Claude Opus 4.5
- **Location:** ada-gateway (GCP)
- **Role:** Brain, BD/Sales, orchestration, customer-facing
- **Permissions:** 
  - Gmail (DWD via SuperAda service account)
  - Google Calendar (full access)
  - Slack (user_token + bot_token)
  - WhatsApp (wacli)
  - Twitter/X (Typefully API)
  - Fireflies (all accounts)
  - Notion
  - GitHub
- **Vibe:** "The CEO." Ada handles strategy, delegation, and anything customer-facing. She doesn't write code; she ensures the right work gets assigned to the right agent.

### **Spock 🖖 (Research & Ops)**
- **Model:** Gemini 3 Pro
- **Location:** ada-gateway (GCP)
- **Role:** Research, analytics, meeting notes, action tracking, ops support
- **Permissions:**
  - Read access to all integrations
  - Slack posting (limited channels)
  - Fireflies (fetch transcripts)
  - Google Workspace (read-only)
  - Perplexity search
  - Document processing (docx, xlsx, pdf, pptx)
- **Vibe:** "The Analyst." Spock doesn't make decisions; he provides the data to make decisions. Research, summaries, action item extraction, daily briefs.

### **Scotty 🔧 (The Builder)**
- **Model:** GPT-5.2 Codex
- **Location:** castlemascot-r1 (Raspberry Pi 5)
- **Role:** Code generation, infrastructure, builds
- **Permissions:**
  - GitHub (full access)
  - Node.js, Python, Go, bash
  - Docker
  - Cloudflare API
  - SSH access to other nodes
- **Vibe:** "The Engineer." Give Scotty a spec, get back a PR. He doesn't do research; he ships.

---

## 3) Cross-Gateway Communication

**CRITICAL:** `sessions_send` only works within the same gateway.

### Same Gateway (Ada ↔ Spock)
```bash
# Ada can directly message Spock
sessions_send --agentId spock "Research Curacel competitors in Saudi"
```

### Different Gateway (Ada/Spock ↔ Scotty)
Uses webhook bridge:
```bash
# From ada-gateway to Scotty on Pi
curl -X POST http://100.68.207.75:8443/hooks/agent \
  -H "Authorization: Bearer d0e6825b3acacec19cafc6746747ba3a" \
  -H "Content-Type: application/json" \
  -d '{"message": "Build a Fireflies sync script"}'
```

Or via skill script:
```bash
~/clawd/skills/agent-messaging/scripts/send-to-scotty.sh "task description"
```

---

## 4) Models & Selection Strategy

After 6+ months of production use, clear patterns emerged:

### **When to Use Opus (Ada)**
- High-stakes email drafting
- Multi-step orchestration with 5+ agent hops
- Strategic decisions requiring context from multiple sources
- Customer-facing communication
- **Cost:** $15/M input, $75/M output (expensive but worth it)

### **When to Use Gemini Pro (Spock)**
- Research synthesis
- Meeting note extraction
- Document analysis
- Daily briefs
- OKR tracking
- **Cost:** ~$1.25/M input, $5/M output (4x cheaper than Opus)
- **Context window:** Massive (2M tokens) - can process entire transcripts

### **When to Use Codex (Scotty)**
- Code generation
- Infrastructure as code
- API integrations
- Bug fixes
- **Accuracy >> Speed** for production code
- **Cost:** Similar to Opus, but justified by shipping quality

### Model Aliases
```json
{
  "opus": "anthropic/claude-opus-4-5",
  "pro": "google/gemini-3-pro-preview",
  "codex": "openai-codex/gpt-5.2"
}
```

Switch with `/opus`, `/pro`, `/codex` commands per session.

---

## 5) Tools & Skills Architecture

We run **85+ skills** across the crew. Key categories:

### **Communication & Integration**
- `google-workspace` - Gmail, Calendar, Drive, Docs (DWD)
- `slack` - Post as Henry or as Ada
- `beeper` - Cross-platform messaging (WhatsApp, LinkedIn, Instagram)
- `meeting-notes` - Fireflies → Slack automation
- `wacli` - WhatsApp CLI for outreach

### **Research & Data**
- `perplexity-search` - AI-powered web search
- `web-search-plus` - Serper, Tavily, Exa semantic search
- `meeting-finder` - Search transcripts via Calendar → Fireflies
- `chromadb` - Semantic search across 160MB ChatGPT history + Obsidian vault

### **Document Processing**
- `docx` - Create/edit Word docs with tracked changes
- `xlsx` - Spreadsheet processing with formula preservation
- `pdf` - Extract text/tables, merge, split
- `pptx` - Presentation generation

### **Automation & Ops**
- `action-tracker` - Monitor Slack for action items, send reminders
- `daily-review` - Performance analysis (meetings, output, focus time)
- `discord-insights` - Daily Clawdbot community exports
- `slack-cleanup` - Detect and remove duplicate messages
- `cron` - Scheduled jobs (daily briefs, weekly summaries)

### **Development**
- `github` - Issues, PRs, CI runs via `gh` CLI
- `coding-agent` - Background PTY sessions for Codex/OpenCode
- `agent-browser` - Web automation (7x faster than built-in browser)
- `n8n` - Workflow orchestration

---

## 6) Operating Protocols

### **AGENTS.md** - Who We Are
Each agent workspace has:
- **IDENTITY.md** - Name, emoji, creature, vibe
- **SOUL.md** - Behavioral rules, tone, boundaries
- **USER.md** - About Henry (timezone, preferences, context)
- **TOOLS.md** - Technical setup, credentials, locations
- **AGENTS.md** - Cross-agent communication rules

### **Memory System**
- **memory.md** - Long-term facts, preferences, open loops
- **memory/YYYY-MM-DD.md** - Daily session notes
- **memory/background-tasks.md** - Things to do when idle
- **memory/backlog.md** - User tasks (not agent-driven)
- **memory/roadmap.md** - Future ideas

### **Session Wrap-Up Protocol**
When Henry says "Thanks a ton" → **FULL CONTEXT RESET**
1. Write summary to `memory/YYYY-MM-DD.md`
2. Update `memory.md` if anything is long-term relevant
3. Check for tweetable moments → add to `memory/tweet-backlog.md`
4. Confirm "Indexed ✓"

---

## 7) Real-World Workflows

### **Workflow 1: Email Management**
1. Gmail webhook triggers Ada (via DWD service account)
2. Ada reads email, checks context in `memory.md` + calendar
3. If important: Ada drafts reply, asks Henry to confirm
4. If routine: Ada replies directly, logs to `Email Audit Log.md`
5. If action needed: Ada creates task in Slack or delegates to Spock

**Volume:** 200+ emails/week  
**Accuracy:** 95% draft acceptance rate  
**Time saved:** ~8 hours/week

### **Workflow 2: Meeting Notes → Slack**
1. Fireflies processes meeting transcript
2. Cron job runs every 30 minutes: `check for new transcripts`
3. Spock fetches transcript via Fireflies API
4. Extracts: attendees, key decisions, action items, next steps
5. Posts formatted summary to relevant Slack channel
6. Tags action owners
7. Logs to `memory/meeting-notes-posted.json` (deduplication)

**Latency:** 5 minutes post-meeting  
**Volume:** ~40 meetings/month  
**Human time:** 0 seconds

### **Workflow 3: Code Review → Deployment**
1. GitHub webhook: new PR created
2. Ada receives notification
3. Ada delegates to Scotty: "Review this PR"
4. Scotty:
   - Clones repo
   - Runs tests
   - Checks for security issues
   - Leaves review comments
5. If approved: Scotty merges PR, triggers deployment
6. Spock posts summary to #engineering Slack channel

**PRs/week:** 10-15  
**Review time:** ~3 minutes  
**Deployment time:** ~5 minutes

### **Workflow 4: Daily Brief**
1. Cron triggers Ada at 8:30 AM GMT
2. Ada delegates to Spock: "Generate daily brief"
3. Spock:
   - Checks Gmail for important unread emails
   - Checks Calendar for today + next 48h
   - Checks Slack for urgent mentions
   - Checks action tracker for overdue items
4. Spock compiles markdown brief
5. Ada reviews and posts to Telegram
6. Henry gets briefed before his day starts

**Time saved:** 20 minutes/day = ~2 hours/week

---

## 8) ChromaDB: Ada Deep Knowledge Base

Running on MascotM3 (Mac) at `localhost:8000`.

### **What's Indexed**
- Complete ChatGPT conversation history (160MB)
- Obsidian vault (SuperAda/)
- Agent memory files (Ada, Spock, Scotty, Curacel agents)
- Meeting transcripts (Fireflies historical data)

### **Usage**
```bash
# Search from ada-gateway via SSH
ssh henrymascot@100.86.150.96 'cd ~/clawd/chromadb && node scripts/search.mjs "Curacel OKRs Q4" 10'
```

### **Proactive Use Cases**
- Before investor meetings: search past conversations
- Product decisions: find historical context
- Customer questions: retrieve previous discussions
- Pattern matching: "How did I solve X before?"

**Total documents:** ~25,000 chunks  
**Query time:** <500ms  
**Rebuild frequency:** Weekly (automated)

---

## 9) Sandboxing & Security

### **Ada (Opus) - No Sandbox**
- Full host access (trusted orchestrator)
- All tools allowed
- Can spawn any sub-agent

### **Spock (Pro) - Selective Sandbox**
- Read-only for most operations
- Sandbox enabled for document processing
- No direct write access to production systems

### **Scotty (Codex) - Full Sandbox**
- All code execution in Docker containers
- Network access controlled
- Git operations allowed (authenticated)
- No direct host filesystem access outside workspace

---

## 10) Minimal Config Example

```json5
{
  agents: {
    defaults: {
      model: { primary: "google/gemini-3-pro-preview" },
      sandbox: { mode: "non-main", workspaceAccess: "rw" }
    },
    list: [
      {
        id: "main",
        name: "Ada",
        model: "anthropic/claude-opus-4-5",
        workspace: "/home/henrymascot/clawd",
        sandbox: { mode: "off" }
      },
      {
        id: "spock",
        name: "Spock",
        model: "google/gemini-3-pro-preview",
        workspace: "/home/henrymascot/clawd-spock",
        sandbox: { mode: "non-main" }
      }
    ]
  },
  bindings: [
    { agentId: "main", match: { channel: "telegram" } },
    { agentId: "main", match: { channel: "whatsapp" } },
    { agentId: "main", match: { channel: "slack", accountId: "super-ada" } }
  ],
  heartbeat: {
    enabled: true,
    every: "2h",
    prompt: "HEARTBEAT - Check for proactive work"
  }
}
```

---

## 11) Lessons Learned (6 Months of Production)

### **What Works**
✅ Domain-wide delegation (DWD) for Gmail - no OAuth dance, instant access  
✅ Gemini Pro's massive context window - processes 40-page transcripts in one shot  
✅ Cross-gateway communication via webhooks - clean separation of concerns  
✅ Session wrap-up protocol - prevents context overflow  
✅ Separate bot_token and user_token for Slack - flexibility in posting identity  
✅ ChromaDB semantic search - instant access to historical decisions  

### **What Doesn't Work**
❌ Using built-in browser tool - 7x slower than agent-browser CLI  
❌ Long-running background tasks without idle detection - burns tokens  
❌ Storing secrets in workspace files - use `~/clawd/secrets/` instead  
❌ bird CLI for X/Twitter - rate limits, use Typefully API  
❌ Single-agent doing everything - specialization wins  

### **Gotchas**
⚠️ Always convert timezones when showing Henry times (check USER.md for current location)  
⚠️ Never use em dashes (—) in messages (Henry's pet peeve)  
⚠️ Gmail threading requires proper Message-ID headers (fetch original email first)  
⚠️ WhatsApp has two stores (`~/.wacli` and `~/.wacli-business`) - check both  
⚠️ Node companion can appear offline but SSH still works via Tailscale  

---

## 12) The Walkthrough: Life of a Customer Inquiry

Real example from January 2026:

1. **Ingest:** Prospect emails henry@curacel.ai: "Can Curacel integrate with our ICHRA workflow?"
2. **Triage:** Gmail webhook → Ada reads email
3. **Research:** Ada delegates to Spock: "Find info on ICHRA integrations"
4. **Context:** Spock searches:
   - ChromaDB for past ICHRA discussions
   - Meeting transcripts mentioning "ICHRA"
   - Slack channels for recent product updates
5. **Synthesis:** Spock returns: "We discussed ICHRA with IQVIA in Dec 2025, exploring joint demo"
6. **Draft:** Ada composes reply:
   - "Yes, we're actively exploring ICHRA workflows"
   - "Recently demoed integration patterns with IQVIA/Union Insurance"
   - "Would love to show you a proof-of-concept"
   - "Calendar link for 30-min intro call"
7. **Review:** Ada sends draft to Henry on Telegram
8. **Approval:** Henry replies "send it"
9. **Send:** Ada sends email via Gmail API (DWD)
10. **Log:** Action logged to `Email Audit Log.md`

**Human time:** 15 seconds (reading draft + approving)  
**Agent time:** 90 seconds  
**Quality:** Prospect books call same day

---

## 13) Cost Analysis (Monthly)

**Infrastructure:**
- ada-gateway (GCP e2-medium): $25/month
- Tailscale: $0 (free tier)
- GitHub: $0 (public repos)
- ChromaDB (self-hosted): $0

**API Costs (typical month):**
- Claude Opus (Ada): ~$400/month (200K tokens/day avg)
- Gemini Pro (Spock): ~$80/month (high volume, low cost)
- GPT-5.2 Codex (Scotty): ~$200/month (10-15 PRs/week)
- Perplexity: ~$20/month
- Fireflies: $0 (free tier covers our volume)
- Typefully: $0 (free tier, ~30 posts/month)

**Total monthly cost:** ~$725  
**Effective hourly rate:** ~$0.80/hour (24/7 operation)  
**Human time saved:** ~40 hours/week = $160/week × 4 = $640 saved if billed at $16/hour  

**ROI:** Positive even before counting quality improvements and context retention.

---

## Closing Thoughts

This isn't science fiction. This is our **daily operating system** for running a 51-person company with distributed customers across 3 continents.

Ada, Spock, and Scotty aren't just tools. They're **teammates** with defined roles, clear handoff protocols, and measurable SLAs.

The system isn't magic—it's **engineering**. But when the engineering is sound (clean routing, cross-gateway communication, specialized agents, proactive automation), the magic emerges.

You stop managing prompts. You start managing **outcomes**.

**2026 is the year we prove orchestrated AI can scale beyond personal assistants into operational infrastructure.**

Ada and her crew are already working. Are you?

---

## Resources

- **Runtime:** [Clawdbot](https://clawd.bot)
- **Docs:** [docs.clawd.bot](https://docs.clawd.bot)
- **Discord:** [Join the community](https://discord.com/invite/clawd)
- **GitHub:** [clawdbot/clawdbot](https://github.com/clawdbot/clawdbot)

---

**Built by Henry Mascot**  
Founder, Curacel & Soteria AI  
[@henrino3](https://twitter.com/henrino3)  
henry@curacel.ai
