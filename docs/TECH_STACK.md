# The 2025 AI-Native Tech Stack

**Philosophy:** Bleeding-edge, Vercel-native, TypeScript everywhere, built for AI agents.

---

## The Stack

```
┌──────────────────────────────────────────────────────────────┐
│                    VERCEL ECOSYSTEM                           │
├──────────────────────────────────────────────────────────────┤
│  Frontend         Next.js 15 (App Router, React Server       │
│                   Components)                                 │
│                                                               │
│  AI Layer         Vercel AI SDK 5 (streaming, tools,         │
│                   structured output)                          │
│                   Vercel AI Gateway (100+ models, failover)  │
│                                                               │
│  Backend          Vercel Serverless Functions                │
│                   Inngest (background jobs, agent workflows)  │
│                                                               │
│  Database         Convex (TypeScript-native, real-time,      │
│                   vector search, serverless functions)        │
│                                                               │
│  Auth             Clerk (social OAuth, JWT, webhooks)        │
│                                                               │
│  Storage          Vercel Blob (voice files, generated docs)  │
│                                                               │
│  Monitoring       Vercel Analytics + Speed Insights          │
│                   Sentry (error tracking)                     │
│                                                               │
│  Deployment       Vercel (edge network, instant rollback)    │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                  SPECIALIZED SERVICES                         │
├──────────────────────────────────────────────────────────────┤
│  Voice AI         Deepgram (real-time transcription)         │
│  LLM              Anthropic Claude 3.5 Sonnet                │
│                   (via Vercel AI Gateway)                     │
│  Emails           Resend (transactional emails)              │
│  Mobile           React Native with Expo                     │
└──────────────────────────────────────────────────────────────┘
```

---

## Why This Stack?

### Vercel AI SDK 5 + AI Gateway
**The industry standard for AI in web apps (2M+ weekly downloads)**

- Access 100+ models (OpenAI, Anthropic, xAI, Meta, Cohere) via single API
- Automatic failover (if Claude down → GPT-4 takes over)
- Built-in streaming, tool calling, structured outputs, attachments
- No token markup (pay provider rates directly)
- Production-ready (powers v0, Cursor, Perplexity)

**Code example:**
```typescript
import { streamText } from 'ai';

const result = streamText({
  model: 'anthropic/claude-3-5-sonnet', // via AI Gateway
  prompt: 'Extract tasks from this voice transcript...',
  tools: {
    createTask: {
      description: 'Create a new task',
      parameters: z.object({
        title: z.string(),
        category: z.string(),
        priority: z.enum(['low', 'medium', 'high']),
      }),
    },
  },
});
```

---

### Convex
**TypeScript-native, real-time database with built-in serverless functions**

**Why not traditional SQL/Postgres?**
- TypeScript schema (no SQL, full type safety from DB → UI)
- Real-time subscriptions (tasks update live on all devices, no WebSocket code)
- Vector search built-in (semantic task search, RAG for agent context)
- Serverless functions included (agent logic lives next to data)
- File storage (voice recordings, meeting briefs)
- Scheduled functions (daily agent runs)

**Convex schema example:**
```typescript
// convex/schema.ts
import { defineSchema, defineTable } from 'convex/server';
import { v } from 'convex/values';

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
    agentId: v.optional(v.id('agents')),
    metadata: v.optional(v.any()),
  })
    .index('by_user', ['userId'])
    .index('by_category', ['userId', 'category'])
    .searchIndex('search_tasks', {
      searchField: 'title',
      filterFields: ['userId', 'status'],
    }),

  agents: defineTable({
    userId: v.string(),
    name: v.string(),
    systemPrompt: v.string(),
    mcpServers: v.array(v.string()),
    triggers: v.optional(v.any()),
  }),

  executions: defineTable({
    taskId: v.id('tasks'),
    agentId: v.id('agents'),
    status: v.union(
      v.literal('queued'),
      v.literal('running'),
      v.literal('completed'),
      v.literal('failed')
    ),
    input: v.any(),
    output: v.optional(v.any()),
    tokensUsed: v.optional(v.number()),
    startedAt: v.optional(v.number()),
    completedAt: v.optional(v.number()),
  }).index('by_task', ['taskId']),
});
```

**Convex mutation example:**
```typescript
// convex/tasks.ts
import { mutation, query } from './_generated/server';
import { v } from 'convex/values';

export const create = mutation({
  args: {
    title: v.string(),
    description: v.optional(v.string()),
    category: v.string(),
    priority: v.string(),
  },
  handler: async (ctx, args) => {
    const userId = await ctx.auth.getUserIdentity();
    if (!userId) throw new Error('Unauthorized');

    const taskId = await ctx.db.insert('tasks', {
      userId: userId.subject,
      title: args.title,
      description: args.description,
      category: args.category,
      status: 'pending',
      priority: args.priority as 'low' | 'medium' | 'high',
    });

    return taskId;
  },
});

export const listByUser = query({
  args: {},
  handler: async (ctx) => {
    const userId = await ctx.auth.getUserIdentity();
    if (!userId) return [];

    const tasks = await ctx.db
      .query('tasks')
      .withIndex('by_user', (q) => q.eq('userId', userId.subject))
      .order('desc')
      .collect();

    return tasks;
  },
});
```

**Real-time subscription (frontend):**
```typescript
'use client';

import { useQuery, useMutation } from 'convex/react';
import { api } from '@/convex/_generated/api';

export function TaskBoard() {
  // Auto-updates when data changes (WebSocket built-in)
  const tasks = useQuery(api.tasks.listByUser);
  const createTask = useMutation(api.tasks.create);

  return (
    <div>
      {tasks?.map(task => (
        <TaskCard key={task._id} task={task} />
      ))}
    </div>
  );
}
```

---

### Inngest
**Serverless background jobs for agent workflows**

**Why Inngest over traditional queues?**
- No infrastructure (no Redis, no workers to manage)
- Built-in retries, rate limiting, batching
- Visual workflow debugger (see agent execution in real-time)
- TypeScript-native (type-safe job definitions)

**Agent execution workflow:**
```typescript
// inngest/functions.ts
import { inngest } from './client';
import { streamText } from 'ai';

export const executeAgent = inngest.createFunction(
  { id: 'execute-agent' },
  { event: 'agent/execute' },
  async ({ event, step }) => {
    const { taskId, agentId, userId } = event.data;

    // Step 1: Load agent config
    const agent = await step.run('load-agent', async () => {
      return await db.agents.get(agentId);
    });

    // Step 2: Execute agent with AI SDK
    const result = await step.run('run-agent', async () => {
      return await streamText({
        model: agent.model,
        system: agent.systemPrompt,
        prompt: `Task: ${event.data.taskTitle}...`,
        tools: await loadAgentTools(agent.mcpServers),
      });
    });

    // Step 3: Save results
    await step.run('save-results', async () => {
      await db.executions.create({
        taskId,
        agentId,
        status: 'completed',
        output: result,
      });
    });

    return { success: true };
  }
);
```

---

### Clerk
**Modern auth with social OAuth, JWT, webhooks**

**Why Clerk?**
- Social OAuth out-of-box (Google, GitHub, Twitter)
- Embeddable UI components (sign-in modal, user button)
- Webhook integration with Convex (sync user data)
- Team/org management (for Team tier)

**Setup:**
```typescript
// app/layout.tsx
import { ClerkProvider } from '@clerk/nextjs';

export default function RootLayout({ children }) {
  return (
    <ClerkProvider>
      <html lang="en">
        <body>{children}</body>
      </html>
    </ClerkProvider>
  );
}
```

**Convex auth integration:**
```typescript
// convex/auth.config.ts
export default {
  providers: [
    {
      domain: process.env.CLERK_JWT_ISSUER_DOMAIN,
      applicationID: 'convex',
    },
  ],
};
```

---

### Deepgram
**Real-time voice transcription with streaming**

**Why Deepgram over Whisper?**
- Faster (< 500ms latency vs 2-3s for Whisper)
- Streaming support (transcribe as user speaks)
- Better punctuation, speaker diarization
- WebSocket API for real-time

**Voice pipeline:**
```typescript
// app/api/transcribe/route.ts
import { createClient } from '@deepgram/sdk';

export async function POST(req: Request) {
  const deepgram = createClient(process.env.DEEPGRAM_API_KEY);
  const audioBlob = await req.blob();

  const { result } = await deepgram.listen.prerecorded.transcribeFile(
    audioBlob,
    {
      model: 'nova-2',
      smart_format: true,
      punctuate: true,
    }
  );

  return Response.json({
    transcript: result.results.channels[0].alternatives[0].transcript,
  });
}
```

---

## Project Structure

```
your-app/
├── app/                      # Next.js App Router
│   ├── (auth)/
│   │   ├── sign-in/
│   │   └── sign-up/
│   ├── dashboard/
│   │   ├── page.tsx          # Kanban board
│   │   └── layout.tsx
│   ├── api/
│   │   └── transcribe/       # Voice API route
│   ├── layout.tsx            # Clerk wrapper
│   └── page.tsx
│
├── convex/                   # Convex backend
│   ├── schema.ts             # Database schema
│   ├── tasks.ts              # Task mutations/queries
│   ├── agents.ts             # Agent CRUD
│   ├── executions.ts         # Agent execution logs
│   └── auth.config.ts        # Clerk integration
│
├── inngest/                  # Background jobs
│   ├── client.ts
│   └── functions.ts          # Agent workflows
│
├── components/
│   ├── task-board.tsx
│   ├── voice-input.tsx
│   └── agent-status.tsx
│
├── lib/
│   ├── mcp/                  # MCP client wrappers
│   │   ├── calendar.ts
│   │   ├── attio.ts
│   │   └── drive.ts
│   └── utils.ts
│
└── package.json
```

---

## Development Setup

### 1. Initialize Convex
```bash
npm install convex
npx convex dev
```

### 2. Initialize Clerk
```bash
npm install @clerk/nextjs
```
Add to `.env.local`:
```
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
CLERK_SECRET_KEY=sk_...
```

### 3. Initialize Vercel AI SDK
```bash
npm install ai @ai-sdk/anthropic
```

### 4. Initialize Inngest
```bash
npm install inngest
npx inngest-cli dev
```

### 5. Deploy to Vercel
```bash
vercel
```

**Environment variables:**
```env
# Clerk
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=

# Convex (auto-configured via npx convex dev)
NEXT_PUBLIC_CONVEX_URL=

# Deepgram
DEEPGRAM_API_KEY=

# Vercel AI Gateway (optional - uses default if not set)
ANTHROPIC_API_KEY=

# Inngest
INNGEST_EVENT_KEY=
INNGEST_SIGNING_KEY=
```

---

## Cost Breakdown (at scale)

### 1,000 Active Users
| Service | Usage | Cost/Month |
|---------|-------|------------|
| Vercel (Pro) | 1M function executions | $20 |
| Convex | 10GB storage, 1M operations | ~$50 |
| Clerk | 1,000 MAU | $25 |
| Deepgram | 30 mins/user × 1,000 = 30K mins | ~$130 |
| Anthropic (via Gateway) | 50M tokens | ~$150 |
| Inngest | 100K function runs | $20 |
| Vercel Blob | 10GB storage | $3 |
| **Total** | | **~$400/month** |

**Revenue**: 1,000 users × $20/month = $20K
**Gross Margin**: ($20K - $400) / $20K = **98%**

---

## Monitoring & Observability

### Vercel Analytics
```typescript
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react';
import { SpeedInsights } from '@vercel/speed-insights/next';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
        <SpeedInsights />
      </body>
    </html>
  );
}
```

### Sentry Error Tracking
```typescript
// sentry.client.config.ts
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  tracesSampleRate: 0.1,
});
```

### Convex Dashboard
- Real-time function execution logs
- Database query inspector
- Performance metrics

### Inngest Dashboard
- Visual workflow execution
- Retry history
- Error debugging

---

## Mobile (React Native + Expo)

```typescript
// Convex React Native setup
import { ConvexProvider, ConvexReactClient } from 'convex/react-native';

const convex = new ConvexReactClient(process.env.EXPO_PUBLIC_CONVEX_URL);

export default function App() {
  return (
    <ConvexProvider client={convex}>
      <VoiceInput />
      <TaskList />
    </ConvexProvider>
  );
}
```

**Voice recording:**
```typescript
import { Audio } from 'expo-av';

async function recordVoice() {
  const recording = new Audio.Recording();
  await recording.prepareToRecordAsync(
    Audio.RecordingOptionsPresets.HIGH_QUALITY
  );
  await recording.startAsync();

  // ... on stop
  const uri = recording.getURI();
  const blob = await fetch(uri).then(r => r.blob());

  // Upload to Vercel Function → Deepgram
  const response = await fetch('/api/transcribe', {
    method: 'POST',
    body: blob,
  });
}
```

---

## Key Advantages of This Stack

**1. Type Safety Everywhere**
- Convex schema → auto-generated TypeScript types
- AI SDK structured outputs → Zod schemas
- End-to-end type safety (DB → API → UI)

**2. Real-Time by Default**
- Convex subscriptions = live updates (no WebSocket code)
- Voice dump on mobile → tasks appear on web instantly
- Agent status changes stream to all devices

**3. Serverless Everything**
- No servers to manage (Vercel handles scaling)
- Pay only for what you use
- Zero-config deployment

**4. AI-Native**
- Vercel AI SDK built for LLM apps
- Convex vector search for semantic queries
- Inngest for multi-step agent workflows

**5. Developer Experience**
- TypeScript everywhere
- Hot reload in dev (Convex + Next.js)
- Visual debugging (Convex + Inngest dashboards)
- Deploy with `vercel` command

---

## Migration Path (If Starting Elsewhere)

**From Supabase/Postgres:**
1. Export data to JSON
2. Define Convex schema
3. Import via Convex mutations
4. Switch frontend queries to Convex
5. Deploy

**From Firebase:**
1. Convex schema matches Firestore (both NoSQL)
2. Migrate auth to Clerk
3. Real-time subscriptions work similarly

---

## Next Steps

1. **Clone starter:** `npx create-next-app@latest --example convex`
2. **Add AI SDK:** `npm install ai @ai-sdk/anthropic`
3. **Set up Clerk:** `npm install @clerk/nextjs`
4. **Deploy to Vercel:** `vercel`
5. **Start building:** Voice input → Task extraction → Agent execution

---

## Resources

- **Vercel AI SDK:** https://sdk.vercel.ai
- **Convex Docs:** https://docs.convex.dev
- **Clerk Docs:** https://clerk.com/docs
- **Inngest Docs:** https://www.inngest.com/docs
- **Deepgram Docs:** https://developers.deepgram.com

**This is the stack. Ship fast.**
