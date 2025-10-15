# MVP Roadmap: 5-Phase Development Plan

## Overview

This roadmap outlines a **12-18 month journey** from MVP to product-market fit, structured in 5 phases. Each phase builds on the previous, with clear milestones and success criteria.

**Philosophy:** Ship fast, learn faster. Each phase should deliver user value, not just technical infrastructure.

**Tech Stack:** Built on the 2025 AI-native stack (Next.js 15, Convex, Vercel AI SDK, Inngest, Clerk). See [TECH_STACK.md](./TECH_STACK.md) for full details.

---

## Phase 1: Voice → Tasks → Kanban (MVP)
**Timeline:** 2-3 months (faster with Convex vs. traditional SQL)
**Team:** 2-3 engineers, 1 designer
**Budget:** $40-60K (burn rate: ~$15-20K/month)

### Goals
- Prove core value prop: Voice input dramatically reduces task capture friction
- Validate AI can extract tasks accurately from natural speech
- Build foundation for future agent automation

### Features

#### 1.1 Voice Input (Mobile + Web)
- **Mobile app (React Native + Expo):**
  - Single-tap voice recording
  - Real-time waveform visualization
  - Streaming upload to backend
- **Web app:**
  - Browser-based voice input (Web Speech API fallback)
  - Text input alternative

**Technical:**
- Deepgram API for transcription (< 500ms latency)
- Vercel Blob for audio file storage
- Delete audio after 30 days (privacy)

---

#### 1.2 AI Task Extraction
- **Vercel AI SDK + Claude 3.5 Sonnet:**
  - Extract atomic tasks from transcript
  - Generate descriptions
  - Auto-categorize (Business, Personal, Admin)
  - Identify priorities (high/medium/low)
  - Detect due dates ("by Friday" → Jan 19)

**Implementation (Vercel AI SDK):**
```typescript
import { generateObject } from 'ai';
import { z } from 'zod';

const { object } = await generateObject({
  model: 'anthropic/claude-3-5-sonnet',
  schema: z.object({
    tasks: z.array(z.object({
      title: z.string(),
      description: z.string().optional(),
      category: z.enum(['Business', 'Personal', 'Admin']),
      priority: z.enum(['low', 'medium', 'high']),
      dueDate: z.string().optional(),
    })),
  }),
  prompt: `Extract tasks from this voice transcript: "${transcript}"`,
});
```

**Accuracy target:** 85%+ on test set (50 sample transcripts)

---

#### 1.3 Kanban Board (Web)
- **View modes:**
  - Kanban (default): Backlog | In Progress | Review | Done
  - List view (mobile-friendly)
- **Interactions:**
  - Drag & drop between columns
  - Click task → detail modal
  - Edit title, description, due date
  - Mark complete
  - Delete task

**Tech stack:**
- Next.js 15 (App Router, React Server Components)
- Tailwind CSS + shadcn/ui
- @dnd-kit for drag & drop
- Convex React hooks (useQuery, useMutation)

---

#### 1.4 Data Persistence (Convex)
- **Convex schema:**
  - tasks, agents, executions, voiceTranscripts, mcpTokens
  - (See [ARCHITECTURE.md](./ARCHITECTURE.md) for full schema)

**Convex Setup:**
```typescript
// convex/schema.ts
export default defineSchema({
  tasks: defineTable({
    userId: v.string(),
    title: v.string(),
    description: v.optional(v.string()),
    category: v.string(),
    status: v.union(
      v.literal('pending'),
      v.literal('in_progress'),
      v.literal('completed')
    ),
    priority: v.union(
      v.literal('low'),
      v.literal('medium'),
      v.literal('high')
    ),
    dueDate: v.optional(v.number()),
  })
    .index('by_user', ['userId'])
    .index('by_user_status', ['userId', 'status']),
});
```

- **Real-time sync:**
  - Convex WebSocket subscriptions (built-in, no Socket.io needed)
  - Mobile ↔ Web instant updates
- **Offline support (mobile):**
  - Convex React Native client handles offline queueing automatically
  - Sync when reconnected

---

#### 1.5 Authentication (Clerk)
- **Clerk Auth:**
  - Email/password signup
  - Google OAuth (recommended)
  - GitHub OAuth
  - Magic link login
- **Convex + Clerk integration:**
  - JWT token validation
  - Auto-sync user data via webhooks
- **User onboarding:**
  - Welcome screen
  - Sample voice dump tutorial
  - Test with 3 demo tasks

**Setup:**
```bash
npm install @clerk/nextjs convex
npx convex dev  # Auto-configures Clerk integration
```

---

### Tech Stack Setup (Phase 1)

#### Week 1-2: Infrastructure
```bash
# Initialize Next.js 15
npx create-next-app@latest --typescript --tailwind

# Initialize Convex
npm install convex
npx convex dev  # Creates convex/ folder, auto-deploys

# Initialize Clerk
npm install @clerk/nextjs
# Add Clerk keys to .env.local

# Initialize Vercel AI SDK
npm install ai @ai-sdk/anthropic

# Initialize Deepgram
npm install @deepgram/sdk

# Initialize Vercel Blob
npm install @vercel/blob
```

#### Week 3-6: Core Features
- Voice recording UI (mobile + web)
- Deepgram transcription pipeline
- Vercel AI SDK task extraction
- Convex mutations (createTask, updateTask, deleteTask)
- Kanban board UI

#### Week 7-8: Polish & Testing
- Real-time sync testing (multi-device)
- Voice accuracy testing (50 sample dumps)
- Onboarding flow
- Error handling, loading states

#### Week 9-12: Beta Launch
- 20 invite-only beta users
- Feedback loop, iteration
- Performance optimization

---

### Success Metrics
- **Technical:**
  - Voice transcription accuracy: > 95%
  - Task extraction accuracy: > 85%
  - Page load time: < 1s (Convex optimized)
  - Mobile app size: < 20MB
- **User:**
  - 20 beta users complete onboarding
  - 80% use voice input (vs. text)
  - Average 5+ tasks created per session
  - 50% weekly retention

### Risks & Mitigation
| Risk | Mitigation |
|------|------------|
| Task extraction accuracy low | Improve prompt, fine-tune Claude, collect training data |
| Voice input friction on mobile | Optimize recording UI, add haptic feedback |
| Users don't see value without agents | Emphasize time saved on task organization alone |

---

## Phase 2: First Sub-Agent (Meeting Prep)
**Timeline:** 2 months
**Team:** Same (2-3 eng, 1 designer)
**Budget:** $40K

### Goals
- Prove sub-agent autonomy delivers 10x value
- Validate MCP integrations work reliably
- Learn what users want automated next

### Features

#### 2.1 Meeting Prep Agent
- **Trigger:** Task contains "meeting", "call", or "prep"
- **Workflow (Inngest):**
  1. Fetch meeting details (Calendar MCP)
  2. Research person/company (Exa MCP)
  3. Pull CRM history (Attio MCP)
  4. Generate meeting brief (Claude via AI SDK)
  5. Save to Drive (Drive MCP)
  6. Notify user (Slack MCP)

**Inngest Workflow Example:**
```typescript
import { inngest } from './client';
import { streamText } from 'ai';

export const meetingPrepAgent = inngest.createFunction(
  { id: 'meeting-prep-agent', retries: 2 },
  { event: 'agent/meeting-prep' },
  async ({ event, step }) => {
    const { taskId, userId } = event.data;

    // Step 1: Fetch meeting details
    const meeting = await step.run('fetch-meeting', async () => {
      return await calendarMCP.getEvent(userId, event.data.eventId);
    });

    // Step 2: Research attendees
    const research = await step.run('research-attendees', async () => {
      return await exaMCP.searchCompany(meeting.attendeeCompany);
    });

    // Step 3: Generate brief
    const brief = await step.run('generate-brief', async () => {
      const { text } = await streamText({
        model: 'anthropic/claude-3-5-sonnet',
        prompt: `Create a meeting brief for ${meeting.title}...`,
      });
      return text;
    });

    // Step 4: Save to Drive
    await step.run('save-to-drive', async () => {
      return await driveMCP.createDocument(userId, brief);
    });

    return { success: true };
  }
);
```

**Agent Storage:**
- Agents stored in Convex `agents` table (not YAML files)
- System prompt, MCP permissions, triggers all in database
- Version control via Convex snapshots

---

#### 2.2 MCP Integration Layer
- **Core MCPs:**
  - Google Calendar MCP (read-only for MVP)
  - Attio CRM MCP (read/write contacts)
  - Exa MCP (web research)
  - Google Drive MCP (write to /Meeting Briefs folder)
  - Slack MCP (send notifications)

**MCP Client Runtime:**
- TypeScript SDK
- OAuth token management (encrypted in Convex `mcpTokens` table)
- Retry logic with exponential backoff (Inngest built-in)
- Error logging (Sentry + Convex logs)

**MCP Tool Integration (Vercel AI SDK):**
```typescript
// Load MCP tools dynamically based on agent config
export async function loadMCPTools(mcpServers: string[], userId: string) {
  const tools: Record<string, any> = {};

  if (mcpServers.includes('google-calendar')) {
    tools.fetchCalendarEvent = {
      description: 'Fetch calendar event details',
      parameters: z.object({ eventId: z.string() }),
      execute: async ({ eventId }) => {
        return await calendarMCP.getEvent(userId, eventId);
      },
    };
  }

  // ... more MCP tools

  return tools;
}
```

---

#### 2.3 Agent Execution UI
- **Task detail view:**
  - "Delegate to Agent" button appears for meeting tasks
  - Agent status: Queued → Running → Completed (Inngest Dashboard link)
  - Progress indicator (e.g., "Researching company...")
  - Live log stream via Convex subscriptions

- **Results display:**
  - Link to meeting brief (Drive)
  - Summary card: Person, Company, Key Points
  - CRM update confirmation

**Real-time Status Updates:**
```typescript
'use client';

import { useQuery } from 'convex/react';
import { api } from '@/convex/_generated/api';

export function AgentStatus({ executionId }) {
  // Auto-updates as agent progresses
  const execution = useQuery(api.executions.get, { id: executionId });

  return (
    <div>
      Status: {execution?.status}
      {execution?.currentStep && <p>Step: {execution.currentStep}</p>}
    </div>
  );
}
```

---

#### 2.4 User Controls
- **Approve/reject agent suggestions:**
  - Before execution: "Meeting Prep Agent can handle this. Proceed?"
  - After execution: Review brief, request changes, mark as done
- **Manual fallback:**
  - If agent fails, user can complete manually
  - Agent learns from manual completions (future enhancement)

---

### Tech Stack Additions (Phase 2)

#### Inngest Setup
```bash
npm install inngest

# Create inngest/ folder
mkdir inngest
touch inngest/client.ts inngest/functions.ts

# Add Inngest API route
# app/api/inngest/route.ts
```

#### MCP Client Manager
```bash
# Install MCP SDKs
npm install @modelcontextprotocol/sdk

# Create lib/mcp/ folder
mkdir -p lib/mcp
touch lib/mcp/calendar.ts lib/mcp/attio.ts lib/mcp/exa.ts
```

---

### Success Metrics
- **Agent performance:**
  - Meeting Prep success rate: > 90%
  - Average execution time: < 3 minutes
  - User approval rate: > 85% (briefs sent without edits)
- **User engagement:**
  - 50% of beta users delegate ≥1 task to agent
  - Average time saved: 30 min/week/user
  - NPS score: > 40

### Learnings to Gather
- Which tasks do users want automated next?
- What agent failures occur most often?
- How much control do users want (auto-execute vs. approve first)?

---

## Phase 3: Multi-Agent Framework (3-5 Agent Types)
**Timeline:** 3 months
**Team:** 4-5 engineers, 1 designer, 1 PM
**Budget:** $120K

### Goals
- Expand agent library to cover 80% of common workflows
- Enable users to create custom agents (no-code)
- Build agent marketplace foundation

### New Agent Types

#### 3.1 Proposal Generator Agent
- **Workflow:** Fetch template → Pull client context → Customize → Save draft

#### 3.2 Research Agent
- **Workflow:** Web search (Exa) → Scrape content (Firecrawl) → Summarize → Save report

#### 3.3 CRM Sync Agent
- **Workflow:** Read emails/transcripts → Extract key points → Update CRM

#### 3.4 QA Verifier Agent
- **Workflow:** Review other agent outputs → Check quality → Flag issues

#### 3.5 (User-Requested Agent)
- Based on Phase 2 feedback

---

### Features

#### 3.6 Agent Builder (No-Code)
- **UI wizard:**
  1. Name your agent
  2. Describe workflow in natural language
  3. Main Agent generates system prompt (Vercel AI SDK)
  4. Select MCP servers
  5. Set triggers (keywords, schedule, manual)
  6. Test with sample task

**Agent Generation (AI SDK):**
```typescript
const { object: agentConfig } = await generateObject({
  model: 'anthropic/claude-3-5-sonnet',
  schema: z.object({
    systemPrompt: z.string(),
    mcpServers: z.array(z.string()),
    triggers: z.array(z.string()),
  }),
  prompt: `Generate an agent config for: "${userDescription}"`,
});

// Save to Convex
await convex.mutation(api.agents.create, {
  userId,
  name: agentName,
  systemPrompt: agentConfig.systemPrompt,
  mcpServers: agentConfig.mcpServers,
  triggers: agentConfig.triggers,
});
```

---

#### 3.7 Agent Marketplace (Beta)
- **Public agent library:**
  - Browse community-created agents (stored in Convex)
  - Filter by category (Sales, Support, Finance, etc.)
  - Ratings & reviews (Convex relations)
  - One-click install (clone agent to user's workspace)
- **Revenue model:**
  - Free agents: Community contributions
  - Paid agents: Creator sets price, 70/30 split (Stripe integration)
- **Quality control:**
  - Agents must pass automated tests before publishing
  - Manual review for top listings

---

#### 3.8 Pattern Recognition (Convex Scheduled Functions)
- **System learns user habits:**
  - Convex scheduled function runs daily
  - Analyzes task patterns (e.g., "client update" tasks 5x/week)
  - Suggests agent creation via notification

**Example:**
```typescript
// convex/crons.ts
import { cronJobs } from 'convex/server';
import { internal } from './_generated/api';

const crons = cronJobs();

crons.daily(
  'analyze-user-patterns',
  { hourUTC: 0 }, // Midnight UTC
  internal.patterns.analyzeAndSuggest
);

export default crons;
```

---

### Success Metrics
- **Agent diversity:**
  - 5 agent types in production
  - 80% of tasks covered by at least 1 agent
- **User adoption:**
  - 60% of users have ≥3 agents active
  - 100 agents published to marketplace
  - Average 2 hours/week saved per user
- **Marketplace:**
  - 20 paid agents published
  - $500 monthly revenue from marketplace

---

## Phase 4: Cross-System Orchestration + QA
**Timeline:** 3 months
**Team:** 5-6 engineers, 1 designer, 1 PM
**Budget:** $180K

### Goals
- Enable complex workflows (multi-step, multi-agent)
- Add QA layer for reliability
- Scale to 1,000 active users

### Features

#### 4.1 Multi-Agent Orchestration (Inngest)
- **Workflow composer:**
  - Define sequential steps: Research → Proposal → CRM Update
  - Parallel execution via Inngest's `step.run` (parallel: true)
  - Conditional logic via TypeScript

**Example Workflow:**
```typescript
export const newClientWorkflow = inngest.createFunction(
  { id: 'new-client-workflow' },
  { event: 'workflow/new-client' },
  async ({ event, step }) => {
    // Parallel execution
    const [companyData, contactData] = await Promise.all([
      step.run('research-company', async () => {
        return await researchAgent.execute(event.data.companyName);
      }),
      step.run('research-contact', async () => {
        return await meetingPrepAgent.execute(event.data.contactEmail);
      }),
    ]);

    // Sequential after parallel steps
    const proposal = await step.run('generate-proposal', async () => {
      return await proposalAgent.execute({ companyData, contactData });
    });

    // QA check
    const qaResult = await step.run('qa-check', async () => {
      return await qaAgent.verify(proposal);
    });

    if (!qaResult.passed) {
      // Flag for human review
      await step.run('flag-for-review', async () => {
        await convex.mutation(api.tasks.update, {
          id: event.data.taskId,
          status: 'needs_review',
        });
      });
    }

    return { success: qaResult.passed };
  }
);
```

---

#### 4.2 QA Verification Layer
- **QA Agent reviews all agent outputs:**
  - Proposals: Grammar, pricing accuracy, personalization
  - CRM updates: Data consistency, no duplicates
  - Research: Source credibility, recency
- **Automated fixes:**
  - Minor issues (typos) → Fix silently (AI SDK tool)
  - Major issues (pricing error) → Flag for human review
- **QA reports stored in Convex:**
  - ✓ Passed checks
  - ⚠ Warnings (fixed)
  - ❌ Failures (needs review)

---

#### 4.3 Advanced MCP Integrations
- **Add Tier 2 MCPs:**
  - Gmail (read/send emails)
  - Notion (knowledge base)
  - Airtable (custom databases)
  - Firecrawl (advanced web scraping)
- **Custom MCPs:**
  - Voice Transcription MCP (Deepgram wrapper)
  - Task Database MCP (Convex API wrapper for external agents)

---

#### 4.4 Team Features (Beta)
- **Shared workspaces (Convex):**
  - Team members see shared tasks (query filter by `teamId`)
  - Assign tasks to specific people (`assignedUserId` field)
  - Shared agent library
- **Permissions:**
  - Admin: Configure agents, billing
  - Member: Create tasks, use agents
  - Viewer: Read-only access
- **Clerk Organizations:**
  - Use Clerk's org management
  - Sync org memberships to Convex via webhooks

---

### Success Metrics
- **Reliability:**
  - QA Agent catches 95% of errors
  - Agent failure rate: < 5%
  - User trust score: > 8/10
- **Scale:**
  - 1,000 active users
  - 50,000 tasks processed/month
  - 10,000 agent executions/month
- **Engagement:**
  - Average 5 agents per user
  - 3+ hours saved per week per user

---

## Phase 5: Enterprise + Agent Platform
**Timeline:** 6 months
**Team:** 10+ engineers, 2 designers, 2 PMs
**Budget:** $500K+

### Goals
- Scale to 10,000+ users
- Enterprise tier for teams
- Open platform for third-party agent developers

### Features

#### 5.1 Enterprise Features
- **SSO & SCIM (Clerk Enterprise):**
  - Okta, Azure AD integration
  - Automated user provisioning
- **Advanced permissions:**
  - Role-based access control (RBAC) via Convex
  - Audit logs for compliance (Convex built-in)
  - Data residency options (Convex multi-region)
- **Dedicated support:**
  - Slack Connect for support
  - Custom onboarding
  - SLA guarantees (99.9% uptime via Convex SLA)

---

#### 5.2 Agent Platform (3rd Party Developers)
- **Developer portal:**
  - Agent SDK (TypeScript)
  - Documentation & tutorials
  - Test sandbox (separate Convex deployment)
- **Publish & monetize:**
  - Submit agents for review (Convex marketplace table)
  - Set pricing (free, paid, usage-based)
  - Revenue dashboard (Stripe Connect)
- **Ecosystem growth:**
  - Target 500 agents in marketplace
  - Partner with MCP server creators

---

#### 5.3 Advanced Analytics (Convex Queries)
- **User insights:**
  - Time saved by agent type (calculated from execution logs)
  - Task completion trends (Convex aggregations)
  - Agent performance benchmarks
- **Admin dashboard (enterprise):**
  - Team productivity metrics
  - Agent ROI calculator
  - Usage billing breakdown

**Example Analytics Query:**
```typescript
// convex/analytics.ts
export const timeSavedByAgent = query({
  args: { userId: v.string() },
  handler: async (ctx, args) => {
    const executions = await ctx.db
      .query('executions')
      .withIndex('by_user', (q) => q.eq('userId', args.userId))
      .collect();

    const byAgent = executions.reduce((acc, exec) => {
      const agentName = exec.agentName || 'Unknown';
      acc[agentName] = (acc[agentName] || 0) + (exec.timeSaved || 0);
      return acc;
    }, {} as Record<string, number>);

    return byAgent;
  },
});
```

---

#### 5.4 Mobile Parity
- **Full feature parity:**
  - Agent configuration on mobile
  - Marketplace browsing
  - Team collaboration
- **Offline mode (Convex React Native):**
  - Queue tasks offline, sync when online (Convex handles automatically)
  - Local agent execution (lightweight tasks via Convex mutations)

---

### Success Metrics
- **Scale:**
  - 10,000 active users
  - 50 enterprise customers (10+ seats each)
  - $500K ARR
- **Marketplace:**
  - 500 agents published
  - 50 active developers
  - $50K monthly revenue from marketplace
- **Retention:**
  - 70% monthly retention
  - < 5% churn
  - NPS: > 50

---

## Infrastructure Costs (Updated for 2025 Stack)

### Phase 1 (100 users, MVP)
| Service | Usage | Cost/Month |
|---------|-------|------------|
| Vercel (Pro) | 100K function executions | $20 |
| Convex | 1GB storage, 100K operations | Free tier |
| Clerk | 100 MAU | Free tier |
| Deepgram | 10 hours transcription | ~$12 |
| Anthropic API (via Gateway) | 5M tokens | ~$15 |
| Vercel Blob | 1GB storage | Free tier |
| **Total** | | **~$50/month** |

### Phase 2-3 (1,000 users)
| Service | Usage | Cost/Month |
|---------|-------|------------|
| Vercel (Pro) | 1M function executions | $20 |
| Convex | 10GB storage, 1M operations | ~$50 |
| Clerk | 1,000 MAU | $25 |
| Deepgram | 100 hours transcription | ~$120 |
| Anthropic API | 50M tokens | ~$150 |
| Inngest | 100K function runs | $20 |
| Vercel Blob | 10GB storage | $3 |
| **Total** | | **~$400/month** |

### Phase 4-5 (10,000 users)
| Service | Usage | Cost/Month |
|---------|-------|------------|
| Vercel (Enterprise) | 10M function executions | $500 |
| Convex | 100GB storage, 10M operations | ~$500 |
| Clerk | 10,000 MAU | $250 |
| Deepgram | 1,000 hours transcription | ~$1,200 |
| Anthropic API | 500M tokens | ~$1,500 |
| Inngest | 1M function runs | $200 |
| Vercel Blob | 100GB storage | $30 |
| Sentry | Error tracking | $26 |
| **Total** | | **~$4,200/month** |

**Revenue at 10K users**: 10K × $20/month = $200K/month
**Gross Margin**: ($200K - $4.2K) / $200K = **98%**

---

## Funding & Hiring Plan

### Pre-Seed ($500K) - Phases 1-2
**Use of funds:**
- Engineering (3 eng × 6 months): $300K
- Design (1 designer × 6 months): $60K
- Infrastructure (Convex, Clerk, Deepgram, Claude API): $30K
- Legal, ops, misc: $40K
- Buffer: $70K
- Runway: ~6 months

**Milestones:**
- Phase 1 complete (MVP with Convex + AI SDK)
- Phase 2 complete (1 agent with Inngest)
- 100 beta users
- $5K MRR

---

### Seed ($2M) - Phases 3-4
**Use of funds:**
- Engineering (6 eng × 12 months): $1.2M
- Product/Design (2 PMs, 2 designers): $400K
- Marketing/Sales (2 people): $200K
- Infrastructure & APIs: $100K (Convex scales efficiently)
- Legal, ops, misc: $100K
- Runway: ~12 months

**Milestones:**
- Phase 3 complete (multi-agent marketplace)
- Phase 4 complete (orchestration + QA)
- 1,000 paying users
- $50K MRR

---

### Series A ($10M+) - Phase 5
**Use of funds:**
- Engineering (20+ eng × 12 months): $4M
- Product, design, PM (5 people): $1M
- Sales & marketing (10 people): $2M
- Enterprise support (5 people): $750K
- Infrastructure & scaling: $1M (Convex enterprise contracts)
- Legal, ops, misc: $250K
- Runway: ~18 months

**Milestones:**
- 10,000+ users
- 100+ enterprise customers
- $3-5M ARR
- Agent marketplace at scale

---

## Risk Mitigation

### Technical Risks
| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| AI accuracy too low | Medium | High | Extensive prompt engineering, fine-tuning, human-in-loop |
| MCP ecosystem stalls | Low | High | Build custom MCPs, influence Anthropic roadmap |
| Convex scaling issues | Low | Medium | Convex designed for scale, fallback to custom infrastructure if needed |

### Market Risks
| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Incumbents (Notion, Linear) copy | Medium | Medium | Speed to market, sub-agent differentiation, platform play |
| Users don't trust agent autonomy | Low | High | Gradual rollout, human-in-loop, transparency |
| Enterprise sales cycle too long | Medium | Low | Focus on SMB/founders first, build enterprise later |

### Execution Risks
| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Hiring takes longer than expected | Medium | Medium | Use contractors for MVP, convert to FTE later |
| Scope creep delays launch | High | Medium | Ruthless prioritization, ship Phase 1 in 3 months max |
| Burn rate too high | Low | High | Conservative hiring, use Convex/Vercel (not custom infra) |

---

## Go/No-Go Criteria

### After Phase 1 (3 months)
**Proceed to Phase 2 if:**
- ✓ 50+ beta users signed up
- ✓ 70%+ weekly retention
- ✓ Task extraction accuracy > 80%
- ✓ Users report 1+ hour/week saved

**Pivot if:**
- ❌ Retention < 50% (core value prop weak)
- ❌ Users prefer manual task entry (voice not valuable)

---

### After Phase 2 (6 months)
**Proceed to Phase 3 if:**
- ✓ Meeting Prep Agent success rate > 85%
- ✓ Users delegate 30%+ of tasks to agent
- ✓ Early revenue signal ($1K+ MRR)

**Pivot if:**
- ❌ Agent success rate < 70% (reliability concerns)
- ❌ Users don't see value in automation

---

### After Phase 3 (9 months)
**Proceed to Phase 4 if:**
- ✓ 500+ active users
- ✓ $20K+ MRR
- ✓ 5 agent types with > 80% success rate
- ✓ Marketplace traction (50+ published agents)

**Pivot if:**
- ❌ Growth stalled (< 20% MoM)
- ❌ Churn > 10% monthly

---

## Next Steps (Pre-Seed Fundraising)

1. **Validate with design partners (4 weeks):**
   - Recruit 10 founders/operators
   - Show Figma mockups
   - Collect feedback on workflows

2. **Build Phase 1 MVP (10-12 weeks):**
   - Week 1-2: Convex + Clerk + Next.js setup
   - Week 3-6: Voice pipeline + task extraction (AI SDK)
   - Week 7-9: Kanban UI + real-time sync
   - Week 10-12: Polish, testing, onboarding

3. **Launch beta (Week 13):**
   - 50 invite-only users
   - Collect usage data, iterate

4. **Add Meeting Prep Agent (Weeks 14-20):**
   - Inngest setup + MCP integrations
   - Prove agent automation value
   - Gather data on what to automate next

5. **Fundraise ($500K pre-seed):**
   - With traction from beta users
   - Clear roadmap to Phase 3

**Target:** Raise pre-seed by Month 6, use funds to accelerate Phase 3.

---

## Why This Stack Enables Speed

**Convex vs. PostgreSQL:**
- No SQL migrations → TypeScript schema changes deploy instantly
- Real-time built-in → No Socket.io setup/maintenance
- Serverless functions → Backend logic lives next to data

**Vercel AI SDK vs. Custom:**
- Structured outputs with Zod → No manual JSON parsing
- Tool calling built-in → Easy MCP integration
- Streaming by default → Better UX

**Inngest vs. BullMQ/Redis:**
- No infrastructure → No Redis to manage
- Visual debugger → See agent execution in real-time
- Automatic retries → Less error handling code

**Result:** Phase 1 MVP in 10-12 weeks instead of 16 weeks with traditional stack.
