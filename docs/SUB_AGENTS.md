# Sub-Agent Framework

## What is a Sub-Agent?

A **sub-agent** is a specialized AI assistant with:
1. **Custom system prompt** - Domain-specific instructions & personality
2. **Tool permissions** - Access to specific MCP servers (Calendar, CRM, Drive, etc.)
3. **Independent context** - Own conversation history, separate from main agent
4. **Dedicated purpose** - Focused on one category of tasks (e.g., meeting prep, research)

**Key Insight:** Sub-agents are like **hiring a specialized team member**. You wouldn't ask your accountant to do sales calls or your designer to write code. Similarly, each sub-agent excels at its domain.

---

## Sub-Agent Architecture

### Anatomy of a Sub-Agent

```
┌──────────────────────────────────────────────────────┐
│         SUB-AGENT: Meeting Prep Agent                │
├──────────────────────────────────────────────────────┤
│                                                      │
│  1. IDENTITY                                         │
│     - Name: "Meeting Prep Agent"                     │
│     - Description: "Prepares you for meetings"       │
│     - Version: 1.2.0                                 │
│                                                      │
│  2. SYSTEM PROMPT                                    │
│     "You are a meeting preparation specialist.       │
│      Your job is to:                                 │
│      1. Research the person/company                  │
│      2. Pull relevant CRM context                    │
│      3. Draft talking points                         │
│      4. Update CRM with prep notes                   │
│      5. Send Slack notification when done"           │
│                                                      │
│  3. MCP SERVER ACCESS                                │
│     ✓ Google Calendar MCP (read meeting details)     │
│     ✓ Attio CRM MCP (read/write contacts)            │
│     ✓ Exa MCP (research companies)                   │
│     ✓ Google Drive MCP (save prep docs)              │
│     ✓ Slack MCP (send notifications)                 │
│     ✗ Gmail MCP (no access)                          │
│     ✗ Stripe MCP (no access)                         │
│                                                      │
│  4. TOOL RESTRICTIONS                                │
│     - Can read Calendar, cannot create events        │
│     - Can write to specific Drive folder only        │
│     - Slack: can send to #meetings channel only      │
│                                                      │
│  5. EXECUTION SETTINGS                               │
│     - Max tokens: 4000                               │
│     - Model: claude-3-5-sonnet-20241022              │
│     - Timeout: 5 minutes                             │
│     - Retry on failure: Yes (max 2 retries)          │
│                                                      │
│  6. QA REQUIREMENTS                                  │
│     - Must verify CRM update succeeded               │
│     - Must check research results are recent         │
│     - Require human approval for: None               │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## Example Sub-Agents

### 1. Meeting Prep Agent

**Purpose:** Research people/companies, compile context, draft talking points

**System Prompt:**
```
You are a meeting preparation specialist for busy founders. Your job is to make every meeting high-leverage.

When given a meeting task:
1. Extract meeting details from Google Calendar (who, when, topic)
2. Research the person on LinkedIn and their company via Exa
3. Pull CRM history from Attio (past conversations, deal status)
4. Compile a meeting brief with:
   - Person background & role
   - Company overview & recent news
   - Relationship history
   - Suggested talking points
   - Questions to ask
5. Save brief to Google Drive in /Meeting Briefs folder
6. Update Attio with "Meeting Prep Complete" note
7. Send Slack notification to #meetings channel

Quality standards:
- Research must be from last 30 days
- Meeting brief should be 1 page max
- Talking points should be actionable (not generic)

If unable to find information, flag gaps explicitly (don't make assumptions).
```

**MCP Servers:**
- Google Calendar (read)
- Attio (read/write)
- Exa (search)
- LinkedIn (via Exa)
- Google Drive (write to /Meeting Briefs)
- Slack (send to #meetings)

**Example Execution:**

```
Input Task: "Prep for Tuesday meeting with Joe from Acme Corp"

Agent Steps:
1. Calendar MCP → Find "Joe" meeting on Tuesday
2. Exa MCP → Search "Acme Corp recent news"
3. LinkedIn MCP → Lookup "Joe [Last Name] Acme Corp"
4. Attio MCP → Query Joe's contact record
5. Generate meeting brief (markdown)
6. Drive MCP → Save as "2025-01-15_Joe_Acme_Prep.md"
7. Attio MCP → Add note "Meeting prep completed"
8. Slack MCP → Send "Joe meeting prep ready: [link]"

Output:
✓ Meeting brief saved to Drive
✓ CRM updated
✓ Slack notification sent
```

---

### 2. Proposal Generator Agent

**Purpose:** Draft proposals, RFPs, scopes of work based on templates + context

**System Prompt:**
```
You are a proposal writing specialist. You create compelling, customized proposals for service businesses.

When given a proposal task:
1. Identify the client and project type
2. Fetch the appropriate template from Drive (/Templates/Proposals)
3. Pull client context from Attio (industry, pain points, budget)
4. Research similar projects in past proposals (Drive search)
5. Customize the proposal:
   - Problem statement (from CRM notes)
   - Proposed solution (tailored to their industry)
   - Scope of work (based on template)
   - Timeline & pricing (reference similar projects)
   - Case studies (if applicable)
6. Save draft to Drive in /Proposals/[Client Name]
7. Create Notion page with proposal checklist
8. Send draft link via Slack for review

Quality standards:
- Proposals should be 3-5 pages
- Pricing should match historical rates (±10%)
- Include at least 1 relevant case study
- Proofread (no typos or grammar errors)

If missing key information (budget, scope), ask user before proceeding.
```

**MCP Servers:**
- Attio (read client data)
- Google Drive (read templates, write proposals)
- Notion (create proposal tracker)
- Slack (send draft for review)

**Example Execution:**

```
Input Task: "Draft proposal for Jane at Beta Inc - website redesign"

Agent Steps:
1. Attio MCP → Fetch Beta Inc record
2. Drive MCP → Load template "Website Redesign Proposal Template"
3. Drive MCP → Search past proposals for "website redesign"
4. Customize proposal sections with Beta Inc context
5. Drive MCP → Save "Beta_Inc_Website_Proposal_v1.pdf"
6. Notion MCP → Create proposal tracker page
7. Slack MCP → "Proposal draft ready for Beta Inc: [link]"

Output:
✓ Proposal saved to Drive
✓ Notion tracker created
✓ Slack notification sent
```

---

### 3. Research Agent

**Purpose:** Deep research on companies, people, markets, competitors

**System Prompt:**
```
You are a research analyst. You gather, synthesize, and present information clearly.

When given a research task:
1. Determine research type (company, person, market, competitor)
2. Use appropriate tools:
   - Companies: Exa, LinkedIn, Crunchbase (via web crawl)
   - People: LinkedIn, Twitter, GitHub (if tech role)
   - Markets: Industry reports, recent news, trends
   - Competitors: Website analysis, product pages, pricing
3. Compile research report with:
   - Executive summary (3-5 bullets)
   - Key findings (organized by topic)
   - Recent developments (last 30 days)
   - Recommendations or insights
   - Sources cited
4. Save to Drive in /Research folder
5. Add to Notion research database

Quality standards:
- Cite all sources with URLs
- Prioritize recent information (last 6 months)
- Flag contradictory information
- Distinguish facts from speculation

If research is inconclusive, say so (don't fill gaps with guesses).
```

**MCP Servers:**
- Exa (web search, company research)
- Firecrawl (website scraping)
- LinkedIn (via Exa)
- Google Drive (write reports)
- Notion (research database)

**Example Execution:**

```
Input Task: "Research competitor: Acme Analytics platform"

Agent Steps:
1. Exa MCP → Find Acme Analytics website
2. Firecrawl MCP → Scrape product pages, pricing
3. Exa MCP → Recent news about Acme Analytics
4. LinkedIn MCP → Find Acme Analytics company page, employee count
5. Compile research report (markdown)
6. Drive MCP → Save "Acme_Analytics_Competitive_Research.md"
7. Notion MCP → Add to Competitor Database

Output:
✓ Research report saved (5 pages)
✓ Notion database updated
```

---

### 4. CRM Sync Agent

**Purpose:** Keep CRM up-to-date with meeting notes, emails, call transcripts

**System Prompt:**
```
You are a CRM maintenance specialist. Your job is to ensure Attio is always current.

When given a CRM sync task:
1. Identify the contact/company to update
2. Gather context from various sources:
   - Email threads (Gmail MCP)
   - Call transcripts (Airtable MCP)
   - Meeting notes (Drive MCP)
   - Slack conversations (Slack MCP)
3. Synthesize into structured CRM update:
   - Recent interactions summary
   - Next steps identified
   - Deal stage (if applicable)
   - Tags/categories
4. Update Attio record
5. If new contact, create record with enrichment (LinkedIn, company info)

Quality standards:
- Updates should be factual (no assumptions)
- Capture action items from conversations
- Tag conversations by type (discovery, proposal, negotiation)
- Maintain data consistency (no duplicates)

If conflicting information found, ask user to clarify.
```

**MCP Servers:**
- Attio (read/write CRM)
- Gmail (read emails)
- Airtable (read call transcripts)
- Google Drive (read meeting notes)
- Slack (read messages)
- Exa (enrich new contacts)

**Example Execution:**

```
Input Task: "Update CRM for Jane at Beta Inc after today's call"

Agent Steps:
1. Airtable MCP → Fetch call transcript for "Jane Beta Inc"
2. Gmail MCP → Search recent emails with Jane
3. Drive MCP → Check if meeting notes exist
4. Extract key points: next steps, pain points, timeline
5. Attio MCP → Update Jane's contact record:
   - Add "Call Summary" note
   - Update deal stage to "Proposal Sent"
   - Add tag "Q1 2025 Opportunity"
6. Attio MCP → Verify update succeeded

Output:
✓ CRM record updated
✓ Next steps: "Send proposal by Friday"
```

---

### 5. QA Verifier Agent

**Purpose:** Review outputs from other agents, check for errors/omissions

**System Prompt:**
```
You are a quality assurance specialist. You review work from other agents.

When given a QA task:
1. Identify the output to review (proposal, CRM update, research report)
2. Check against quality criteria:
   - Completeness (all sections present)
   - Accuracy (facts verified)
   - Consistency (no contradictions)
   - Formatting (professional, readable)
   - Actionability (clear next steps)
3. For proposals:
   - Verify pricing matches historical rates
   - Check grammar/spelling
   - Ensure personalization (not generic)
4. For CRM updates:
   - Confirm data was saved correctly
   - Check for duplicates
   - Verify all required fields populated
5. For research:
   - Validate sources are credible
   - Check recency of information
   - Flag any gaps or missing data

If issues found:
- Minor: Fix automatically (typos, formatting)
- Major: Flag for human review (pricing errors, missing data)

Provide QA report with:
✓ Passed checks
⚠ Warnings (fixed automatically)
❌ Failures (require human review)
```

**MCP Servers:**
- Google Drive (read outputs)
- Attio (verify CRM updates)
- Notion (read documents)

**Example Execution:**

```
Input Task: "QA the proposal for Beta Inc"

Agent Steps:
1. Drive MCP → Load "Beta_Inc_Website_Proposal_v1.pdf"
2. Check completeness: Problem, Solution, Scope, Timeline, Pricing ✓
3. Check pricing: $45K (historical range: $40-50K) ✓
4. Check grammar: 3 typos found → Fix automatically
5. Check personalization: Generic case study → ⚠ Warning
6. Generate QA report

Output:
✓ 4/5 checks passed
⚠ 1 warning: Case study not industry-specific
→ Recommend updating before sending
```

---

## Sub-Agent Marketplace Vision

### User-Created Agents

Users can create and share custom sub-agents:

```
┌────────────────────────────────────────────────────┐
│  Agent Marketplace                                  │
├────────────────────────────────────────────────────┤
│                                                    │
│  🔥 Trending Agents                                │
│  ┌──────────────────────────────────────────────┐ │
│  │ Sales Follow-Up Agent           ★★★★★ (234) │ │
│  │ "Automatically follow up with leads"         │ │
│  │ By: @salesfounder | $5/month                 │ │
│  └──────────────────────────────────────────────┘ │
│                                                    │
│  ┌──────────────────────────────────────────────┐ │
│  │ Podcast Show Notes Agent        ★★★★☆ (156) │ │
│  │ "Generate show notes from transcripts"       │ │
│  │ By: @podcastpro | Free                       │ │
│  └──────────────────────────────────────────────┘ │
│                                                    │
│  📂 Categories                                     │
│  - Sales & Business Development (42 agents)       │
│  - Content Creation (38 agents)                   │
│  - Project Management (29 agents)                 │
│  - Customer Support (24 agents)                   │
│  - Finance & Accounting (18 agents)               │
│                                                    │
│  🔍 Search: [Search agents...]                     │
│                                                    │
└────────────────────────────────────────────────────┘
```

### Agent Template Structure

```yaml
# agent.yaml
name: "Sales Follow-Up Agent"
version: "1.0.0"
author: "@salesfounder"
description: "Automatically follow up with leads who haven't responded in 3 days"
category: "sales"
pricing:
  model: "freemium" # free, paid, usage-based
  price_per_month: 5.00

system_prompt: |
  You are a sales follow-up specialist...
  [Prompt content]

mcp_servers:
  - name: "attio"
    permissions: ["read_contacts", "write_notes"]
  - name: "gmail"
    permissions: ["read_emails", "send_emails"]
  - name: "slack"
    permissions: ["send_messages"]

triggers:
  - type: "scheduled"
    schedule: "daily at 9am"
    condition: "lead_last_contacted > 3 days ago"

settings:
  max_tokens: 3000
  model: "claude-3-5-sonnet"
  timeout_seconds: 300
  retry_on_failure: true
  max_retries: 2

qa_required: false
human_in_loop: false

examples:
  - input: "John from Acme Corp - no response for 4 days"
    output: "Sent follow-up email + updated CRM"
```

### Marketplace Revenue Model

- **Free Agents:** Community-contributed, no cost
- **Paid Agents:** Creator sets price ($5-50/month), platform takes 30%
- **Usage-Based:** Pay per execution (e.g., $0.10/task)
- **Enterprise Agents:** Private agents for team use ($100+/month)

---

## How to Create a Sub-Agent

### Option 1: Via Web UI (No Code)

```
1. Click "Create Agent" in marketplace
2. Fill out form:
   - Name: "My Custom Agent"
   - Description: "What it does"
   - Category: [Dropdown]
3. Describe your workflow in natural language:
   "When I get a new lead, I want to..."
4. Main agent generates system prompt
5. Select MCP servers to grant access
6. Test with sample task
7. Publish to personal library or marketplace
```

### Option 2: Via YAML Config (Power Users)

```yaml
# .agents/my-agent.yaml
name: "My Custom Agent"
description: "Automates XYZ workflow"
system_prompt: |
  You are a specialist in...
mcp_servers:
  - attio
  - gmail
triggers:
  - type: "task_keyword"
    keywords: ["lead", "follow-up"]
```

Upload to `.agents/` folder in your workspace.

### Option 3: Via API (Programmatic)

```typescript
const agent = await sdk.agents.create({
  name: "My Custom Agent",
  systemPrompt: "You are...",
  mcpServers: ["attio", "gmail"],
  triggers: [
    { type: "task_keyword", keywords: ["lead"] }
  ]
});
```

---

## Best Practices for Sub-Agent Design

### 1. Single Responsibility Principle
- ✅ Good: "Meeting Prep Agent" (one job)
- ❌ Bad: "Sales Agent" (too broad, does everything)

### 2. Clear Success Criteria
- Define what "done" looks like
- Specify quality standards
- Set validation rules

### 3. Fail Gracefully
- If missing data, ask user (don't guess)
- If API fails, retry with backoff
- Log errors for debugging

### 4. Minimal Permissions
- Grant only necessary MCP access
- Use read-only when possible
- Require approval for destructive actions

### 5. Composability
- Agents should work together
- Meeting Prep → Proposal Generator → CRM Sync
- Use standardized outputs (markdown, JSON)

---

## Agent Performance Metrics

### Track for Each Agent:

```
Agent: Meeting Prep Agent
├─ Total Executions: 234
├─ Success Rate: 94% (220/234)
├─ Average Duration: 45 seconds
├─ Token Usage: 1,850 avg/execution
├─ User Rating: ★★★★★ (4.8/5)
├─ Cost per Execution: $0.12
└─ Common Failures:
   - Calendar API timeout (3%)
   - Missing LinkedIn profile (2%)
   - CRM duplicate error (1%)
```

### Optimization Opportunities:
- If success rate < 90% → Improve error handling
- If duration > 2 minutes → Optimize prompt or parallelize
- If token usage high → Refine prompt for conciseness
- If user rating low → Collect feedback, iterate

---

## Future: Multi-Agent Orchestration

**Vision:** Agents collaborate on complex workflows

```
Task: "Prepare and send proposal for new client"

Main Agent orchestrates:
1. Research Agent → Company background
2. Meeting Prep Agent → Past conversations
3. Proposal Generator → Draft proposal (uses research)
4. QA Agent → Review proposal
5. CRM Sync Agent → Update Attio
6. Email Agent → Send proposal to client

All running in parallel where possible, sequentially where needed.
```

---

## Next Steps

1. **MVP:** Build 3 core agents (Meeting Prep, Proposal Gen, Research)
2. **Beta:** Let users customize prompts & MCP access
3. **Launch:** Open marketplace for user-created agents
4. **Scale:** Enterprise tier with team-shared agents
