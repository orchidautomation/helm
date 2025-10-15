# System Architecture

## Overview

This document outlines the technical architecture for the AI-native productivity OS. The system is designed as a **fully serverless, Vercel-native platform** where voice input flows through AI extraction, specialized sub-agents execute via MCP integrations, and results surface in real-time UI with zero infrastructure management.

**Philosophy:** TypeScript everywhere, serverless everything, AI-native from the ground up.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      CLIENT LAYER                                │
│  ┌────────────────────┐        ┌─────────────────────────────┐  │
│  │   Mobile App       │        │      Web Application        │  │
│  │  (React Native)    │◄──────►│   (Next.js 15)             │  │
│  │                    │        │                             │  │
│  │  - Voice Input     │        │  - Kanban Board             │  │
│  │  - Task List       │        │  - Task Detail View         │  │
│  │  - Push Notifs     │        │  - Side Chat Panel          │  │
│  │  - Agent Status    │        │  - Agent Marketplace        │  │
│  └────────────────────┘        └─────────────────────────────┘  │
└──────────────────────┬──────────────────────┬───────────────────┘
                       │                      │
                       │  Convex WebSocket    │
                       │  (Real-time Sync)    │
                       │                      │
┌──────────────────────┴──────────────────────┴───────────────────┐
│                    VERCEL PLATFORM                               │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Next.js 15 (App Router + Server Components)            │   │
│  │  - API Routes (voice upload, webhooks)                  │   │
│  │  - Server Actions (mutations, AI calls)                 │   │
│  │  - Clerk Auth (middleware)                              │   │
│  └──────────────────────────────────────────────────────────┘   │
└──────────────────────┬──────────────────────────────────────────┘
                       │
┌──────────────────────┴──────────────────────────────────────────┐
│                   CONVEX BACKEND                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Convex (Database + Serverless Functions + Real-time)   │   │
│  │                                                          │   │
│  │  - Mutations (create/update tasks, agents)              │   │
│  │  - Queries (list tasks, get agent status)               │   │
│  │  - Actions (AI calls, MCP integrations)                 │   │
│  │  - Scheduled Functions (daily agent runs)               │   │
│  │  - File Storage (voice recordings, generated docs)      │   │
│  │  - Vector Search (semantic task search)                 │   │
│  └──────────────────────────────────────────────────────────┘   │
└──────────────────────┬──────────────────────────────────────────┘
                       │
┌──────────────────────┴──────────────────────────────────────────┐
│                  AI ORCHESTRATION                                │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Vercel AI SDK + AI Gateway                              │   │
│  │  - Task extraction (Claude 3.5 Sonnet)                   │   │
│  │  - Agent execution (streaming, tool calling)             │   │
│  │  - Structured outputs (Zod schemas)                      │   │
│  │  - Automatic failover (Claude → GPT-4)                   │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Sub-Agents (Convex Actions + AI SDK)                    │   │
│  │  - Meeting Prep Agent                                    │   │
│  │  - Proposal Generator                                    │   │
│  │  - Research Agent                                        │   │
│  │  - CRM Sync Agent                                        │   │
│  │  - QA Verifier Agent                                     │   │
│  └──────────────────────────────────────────────────────────┘   │
└──────────────────────┬──────────────────────────────────────────┘
                       │
┌──────────────────────┴──────────────────────────────────────────┐
│               BACKGROUND JOBS (INNGEST)                          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Inngest Workflows                                       │   │
│  │  - Agent execution pipelines                             │   │
│  │  - Multi-step workflows (Research → Draft → QA)          │   │
│  │  - Retries & error handling                              │   │
│  │  - Rate limiting                                         │   │
│  └──────────────────────────────────────────────────────────┘   │
└──────────────────────┬──────────────────────────────────────────┘
                       │
┌──────────────────────┴──────────────────────────────────────────┐
│                   MCP INTEGRATION HUB                            │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  MCP Clients (via Convex Actions + Vercel AI SDK)       │   │
│  │  - OAuth token management (Convex storage)               │   │
│  │  - Request/response handling                             │   │
│  │  - Error handling & retries                              │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌──────────┐  │
│  │  Calendar  │  │   Attio    │  │   Drive    │  │  Slack   │  │
│  │    MCP     │  │    MCP     │  │    MCP     │  │   MCP    │  │
│  └────────────┘  └────────────┘  └────────────┘  └──────────┘  │
│                                                                  │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌──────────┐  │
│  │   Notion   │  │   GitHub   │  │    Exa     │  │ Airtable │  │
│  │    MCP     │  │    MCP     │  │    MCP     │  │   MCP    │  │
│  └────────────┘  └────────────┘  └────────────┘  └──────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

**Key Characteristics:**
- ✅ **Fully Serverless:** No infrastructure to manage (Vercel + Convex handle scaling)
- ✅ **Type-Safe:** TypeScript from DB → API → UI
- ✅ **Real-Time:** Convex subscriptions (WebSocket built-in, no Socket.io)
- ✅ **AI-Native:** Vercel AI SDK + AI Gateway throughout
- ✅ **Zero Config:** Deploy with `vercel` command

---

## Data Flow: Voice Input to Task Execution

```
┌──────────────────────────────────────────────────────────────────┐
│  STEP 1: VOICE CAPTURE (Mobile App)                             │
└──────────────────────────────────────────────────────────────────┘
   User records voice dump
   │
   ├─► Audio blob sent to Next.js API Route
   │   POST /api/transcribe
   │
   └─► Deepgram transcription
       │
       └─► Transcript returned (< 500ms)

┌──────────────────────────────────────────────────────────────────┐
│  STEP 2: TASK EXTRACTION (Vercel AI SDK)                        │
└──────────────────────────────────────────────────────────────────┘
   API Route calls Vercel AI SDK:

   import { generateObject } from 'ai';
   import { z } from 'zod';

   const { object } = await generateObject({
     model: 'anthropic/claude-3-5-sonnet',
     schema: z.object({
       tasks: z.array(z.object({
         title: z.string(),
         description: z.string().optional(),
         category: z.string(),
         priority: z.enum(['low', 'medium', 'high']),
       })),
     }),
     prompt: `Extract tasks from: "${transcript}"`,
   });

   → Returns structured JSON (type-safe)

┌──────────────────────────────────────────────────────────────────┐
│  STEP 3: TASK PERSISTENCE (Convex Mutation)                     │
└──────────────────────────────────────────────────────────────────┘
   For each extracted task:

   await convex.mutation(api.tasks.create, {
     title: task.title,
     description: task.description,
     category: task.category,
     priority: task.priority,
   });

   → Convex saves to database
   → Real-time update pushed to all connected clients (WebSocket)
   → UI updates instantly on web + mobile

┌──────────────────────────────────────────────────────────────────┐
│  STEP 4: AGENT ROUTING (Convex Query + Inngest)                 │
└──────────────────────────────────────────────────────────────────┘
   Convex function determines which agent to use:

   const agent = await ctx.db
     .query('agents')
     .filter(q => q.eq(q.field('trigger_keywords'), task.category))
     .first();

   If agent found:
   → Trigger Inngest workflow

   await inngest.send({
     name: 'agent/execute',
     data: { taskId, agentId, userId },
   });

┌──────────────────────────────────────────────────────────────────┐
│  STEP 5: AGENT EXECUTION (Inngest Workflow)                     │
└──────────────────────────────────────────────────────────────────┘
   Inngest function runs:

   export const executeAgent = inngest.createFunction(
     { id: 'execute-agent' },
     { event: 'agent/execute' },
     async ({ event, step }) => {
       // Step 1: Load agent config
       const agent = await step.run('load-agent', ...);

       // Step 2: Run agent with AI SDK
       const result = await step.run('run-agent', async () => {
         return await streamText({
           model: 'anthropic/claude-3-5-sonnet',
           system: agent.systemPrompt,
           tools: { /* MCP tools */ },
         });
       });

       // Step 3: Save results to Convex
       await step.run('save-results', ...);
     }
   );

┌──────────────────────────────────────────────────────────────────┐
│  STEP 6: MCP TOOL CALLS (Convex Actions)                        │
└──────────────────────────────────────────────────────────────────┘
   Agent calls MCPs via Vercel AI SDK tools:

   tools: {
     fetchCalendarEvent: {
       parameters: z.object({ eventId: z.string() }),
       execute: async ({ eventId }) => {
         // Convex Action calls Google Calendar MCP
         return await calendarMCP.getEvent(eventId);
       },
     },
     updateCRM: {
       parameters: z.object({ contactId, notes }),
       execute: async (params) => {
         // Convex Action calls Attio MCP
         return await attioMCP.updateContact(params);
       },
     },
   }

   → Agent can call multiple MCPs in sequence
   → Results streamed back to Inngest workflow

┌──────────────────────────────────────────────────────────────────┐
│  STEP 7: REAL-TIME UI UPDATE                                    │
└──────────────────────────────────────────────────────────────────┘
   Convex mutation updates task status:

   await ctx.db.patch(taskId, {
     status: 'completed',
     agentOutput: result,
   });

   → Convex subscription pushes update to all clients
   → Kanban board updates instantly
   → No manual refresh needed

┌──────────────────────────────────────────────────────────────────┐
│  STEP 8: USER NOTIFICATION                                      │
└──────────────────────────────────────────────────────────────────┘
   Convex scheduled function or Inngest sends notification:

   - Push notification (Expo push API for mobile)
   - Email (Resend)
   - Slack message (via MCP)

   User sees: "Meeting prep complete for Joe (Acme Corp)"
```

---

## Convex Data Model

**Schema definition** (TypeScript, not SQL):

```typescript
// convex/schema.ts
import { defineSchema, defineTable } from 'convex/server';
import { v } from 'convex/values';

export default defineSchema({
  // Tasks
  tasks: defineTable({
    userId: v.string(),
    title: v.string(),
    description: v.optional(v.string()),
    category: v.string(),
    status: v.union(
      v.literal('pending'),
      v.literal('in_progress'),
      v.literal('completed'),
      v.literal('blocked')
    ),
    priority: v.union(
      v.literal('low'),
      v.literal('medium'),
      v.literal('high')
    ),
    dueDate: v.optional(v.number()),
    assignedAgentId: v.optional(v.id('agents')),
    metadata: v.optional(v.any()),
    createdAt: v.number(),
    updatedAt: v.number(),
  })
    .index('by_user', ['userId'])
    .index('by_user_status', ['userId', 'status'])
    .index('by_category', ['userId', 'category'])
    .searchIndex('search_title', {
      searchField: 'title',
      filterFields: ['userId', 'status'],
    }),

  // Categories
  categories: defineTable({
    userId: v.string(),
    name: v.string(),
    description: v.optional(v.string()),
    color: v.optional(v.string()),
    suggestedAgentId: v.optional(v.id('agents')),
    taskCount: v.number(),
  }).index('by_user', ['userId']),

  // Sub-Agents
  agents: defineTable({
    userId: v.string(),
    name: v.string(),
    description: v.string(),
    systemPrompt: v.string(),
    mcpServers: v.array(v.string()),
    toolPermissions: v.optional(v.any()),
    triggerKeywords: v.array(v.string()),
    isPublic: v.boolean(),
    usageCount: v.number(),
  }).index('by_user', ['userId']),

  // Agent Executions (Audit Trail)
  executions: defineTable({
    taskId: v.id('tasks'),
    agentId: v.id('agents'),
    userId: v.string(),
    status: v.union(
      v.literal('queued'),
      v.literal('running'),
      v.literal('completed'),
      v.literal('failed')
    ),
    input: v.any(),
    output: v.optional(v.any()),
    tokensUsed: v.optional(v.number()),
    executionTimeMs: v.optional(v.number()),
    errorMessage: v.optional(v.string()),
    startedAt: v.optional(v.number()),
    completedAt: v.optional(v.number()),
  })
    .index('by_task', ['taskId'])
    .index('by_agent', ['agentId'])
    .index('by_user', ['userId']),

  // Voice Transcripts
  voiceTranscripts: defineTable({
    userId: v.string(),
    audioStorageId: v.optional(v.id('_storage')), // Convex file storage
    transcript: v.string(),
    extractedTaskIds: v.array(v.id('tasks')),
    createdAt: v.number(),
  }).index('by_user', ['userId']),

  // MCP OAuth Tokens (encrypted)
  mcpTokens: defineTable({
    userId: v.string(),
    mcpServer: v.string(),
    accessToken: v.string(), // Encrypted
    refreshToken: v.optional(v.string()), // Encrypted
    expiresAt: v.number(),
    scopes: v.array(v.string()),
  }).index('by_user_server', ['userId', 'mcpServer']),
});
```

**Type safety:** Convex auto-generates TypeScript types from schema → Use throughout app.

---

## Real-Time Subscriptions

**Frontend (React):**
```typescript
'use client';

import { useQuery, useMutation } from 'convex/react';
import { api } from '@/convex/_generated/api';

export function TaskBoard() {
  // Auto-subscribes via WebSocket, re-renders on any change
  const tasks = useQuery(api.tasks.listByUser);
  const createTask = useMutation(api.tasks.create);

  // Tasks update live when agents complete, no polling needed
  return (
    <div>
      {tasks?.map(task => (
        <TaskCard
          key={task._id}
          task={task}
          onComplete={() => {/* mutation */}}
        />
      ))}
    </div>
  );
}
```

**How it works:**
1. Client calls `useQuery(api.tasks.listByUser)`
2. Convex establishes WebSocket connection
3. On server-side mutation (e.g., agent updates task), Convex pushes update
4. Client re-renders with fresh data
5. **Zero configuration needed**

---

## Tech Stack Summary

| Layer | Technology | Why |
|-------|------------|-----|
| Frontend (Web) | Next.js 15 + TypeScript | App Router, RSC, Server Actions |
| Frontend (Mobile) | React Native + Expo | Cross-platform, same Convex client |
| Database | Convex | TypeScript-native, real-time, vector search |
| Auth | Clerk | Social OAuth, JWT, webhooks |
| AI | Vercel AI SDK + AI Gateway | 100+ models, streaming, tools |
| Background Jobs | Inngest | Serverless workflows, visual debugger |
| Voice | Deepgram | Real-time transcription, streaming |
| Storage | Vercel Blob + Convex Files | Voice recordings, generated docs |
| Monitoring | Vercel Analytics + Sentry | Performance + error tracking |
| Deployment | Vercel | Edge network, instant rollback |

**No legacy tech:**
- ❌ PostgreSQL/SQL (use Convex)
- ❌ Redis (use Convex for cache)
- ❌ Socket.io (use Convex subscriptions)
- ❌ BullMQ (use Inngest)
- ❌ Separate API server (use Next.js + Convex)

---

## Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    VERCEL EDGE NETWORK                       │
│  (Global CDN + Serverless Functions)                        │
└────────────────────────┬────────────────────────────────────┘
                         │
         ┌───────────────┴───────────────┐
         │                               │
         ▼                               ▼
┌──────────────────┐           ┌──────────────────┐
│  Next.js Web App │           │ Next.js API      │
│  (Static + RSC)  │           │ Routes           │
│                  │           │ - /api/transcribe│
│  Deployed to:    │           │ - /api/webhooks  │
│  Vercel Edge     │           │                  │
└──────────────────┘           └──────────────────┘
         │                               │
         └───────────────┬───────────────┘
                         │
                         ▼
         ┌───────────────────────────────┐
         │  Convex Cloud                 │
         │  - Database (multi-region)    │
         │  - Serverless Functions       │
         │  - WebSocket (real-time)      │
         │  - File Storage               │
         │  - Vector Search              │
         └───────────────┬───────────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
┌──────────────┐  ┌──────────┐  ┌─────────────┐
│  Clerk       │  │  Inngest │  │  Deepgram   │
│  (Auth)      │  │  (Jobs)  │  │  (Voice)    │
└──────────────┘  └──────────┘  └─────────────┘
```

**Deployment:**
1. `git push` → Vercel auto-deploys frontend + API
2. `npx convex deploy` → Convex deploys backend functions
3. Environment variables synced via Vercel dashboard

**Zero infrastructure management:**
- No servers to provision
- No scaling configuration
- No database backups (Convex handles)
- No load balancers (Vercel Edge handles)

---

## Security & Privacy

### Auth (Clerk)
- Social OAuth (Google, GitHub, Twitter)
- JWT tokens (auto-refreshed)
- Webhook integration with Convex (sync user data)
- Team/org management (for Team tier)

### Data Encryption
- Convex: Encrypted at rest (AES-256)
- MCP tokens: Encrypted in Convex (crypto library)
- Voice files: Auto-deleted after 30 days

### Agent Permissions
- Convex row-level security (users can only access their data)
- MCP access scoped per agent (Meeting Prep can't access Gmail)
- Human-in-loop for sensitive operations (env variable toggle)

### API Security
- Clerk JWT validation on all requests
- Rate limiting (Vercel built-in: 100 req/10s per IP)
- CORS auto-configured (Next.js)

---

## Monitoring & Observability

### Vercel Analytics
- Page views, conversions, Web Vitals
- Real-time dashboard

### Convex Dashboard
- Function execution logs (real-time)
- Database queries (inspect performance)
- WebSocket connections (active users)
- Scheduled function history

### Inngest Dashboard
- Visual workflow execution
- Step-by-step debugging
- Retry history
- Error traces

### Sentry (Error Tracking)
- Frontend + backend errors
- AI SDK failures
- Agent execution errors

---

## Scalability

### Automatic (Convex + Vercel Handle)
- Database: Auto-scales to millions of documents
- Functions: Serverless, infinite horizontal scaling
- WebSocket: Convex manages connections
- CDN: Vercel Edge network (global)

### Manual Optimizations (if needed at scale)
1. **Convex indexes:** Add for frequent queries
2. **Caching:** Use Convex queries (auto-cached)
3. **File storage:** Move large files to Vercel Blob
4. **Rate limiting:** Custom logic in Convex actions

**Cost at 10,000 users:**
- Convex: ~$500/month
- Vercel Pro: $20/month
- Clerk: $500/month (5,000 MAU tier)
- Inngest: ~$100/month
- **Total: ~$1,200/month** (vs $20K revenue = 94% margin)

---

## Development Workflow

### Local Development
```bash
# Terminal 1: Convex backend
npx convex dev

# Terminal 2: Next.js frontend
npm run dev

# Terminal 3: Inngest dev server
npx inngest-cli dev
```

**Hot reload:** Changes to Convex functions → auto-redeploy (< 1s)

### Testing
```bash
# Convex test framework
npx convex test

# Next.js + React Testing Library
npm test
```

### Deployment
```bash
# Deploy frontend + API
vercel

# Deploy Convex backend
npx convex deploy --prod

# Environment variables
vercel env pull
```

---

## Next Steps

1. **Initialize Convex:** `npx convex dev`
2. **Set up Clerk:** Add to Next.js layout
3. **Install AI SDK:** `npm install ai @ai-sdk/anthropic`
4. **Build voice pipeline:** Next.js API route → Deepgram → Convex
5. **Deploy:** `vercel`

**See TECH_STACK.md for detailed setup instructions.**
