# MVP v0.2 Implementation Guide
## Streamlined Stack: Voice → Tasks → Kanban

**Last Updated:** October 2025
**Estimated Time:** 3-4 weeks full-time
**Target:** 20-30 beta users

---

## Table of Contents

1. [Stack Overview](#stack-overview)
2. [Prerequisites](#prerequisites)
3. [Project Initialization](#project-initialization)
4. [Convex Database Setup](#convex-database-setup)
5. [Clerk Authentication](#clerk-authentication)
6. [Vercel AI SDK Integration](#vercel-ai-sdk-integration)
7. [Voice Recording Component](#voice-recording-component)
8. [API Routes](#api-routes)
9. [Kanban Board UI](#kanban-board-ui)
10. [Deployment](#deployment)
11. [Testing & Launch](#testing--launch)

---

## Stack Overview

### **Final Streamlined Stack**

```
┌─────────────────────────────────────────┐
│      Vercel AI SDK (Unified)            │
│  • Transcription: OpenAI Whisper        │
│  • Task Extract: Claude 3.5 Sonnet      │
│  • AI Gateway: Auto-failover            │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│      Next.js 15 (App Router)            │
│  • Server Actions                       │
│  • API Routes                           │
│  • React Server Components              │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│      Convex (Database + Real-time)      │
│  • TypeScript-native                    │
│  • WebSocket subscriptions              │
│  • Serverless functions                 │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│      Clerk (Authentication)             │
│  • Google OAuth                         │
│  • User management                      │
│  • Session handling                     │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│      Vercel (Deployment)                │
│  • Edge functions                       │
│  • Auto-scaling                         │
│  • Analytics                            │
└─────────────────────────────────────────┘
```

### **Why This Stack?**

| Decision | Rationale | Alternative Rejected |
|----------|-----------|---------------------|
| **OpenAI Whisper** | Same SDK as Claude, $2/mo cheaper at scale, good enough latency (1-2s) | Deepgram (adds complexity) |
| **Convex** | Database + real-time + serverless in one, no DevOps | PostgreSQL + Redis + Socket.io |
| **Clerk** | Auth in 20 minutes, handles OAuth complexity | NextAuth (more setup) |
| **No Inngest yet** | Next.js Server Actions sufficient for MVP | Inngest (Phase 2 when agents added) |
| **No React Native yet** | Mobile web is enough for validation | Expo (Phase 2) |

---

## Prerequisites

### **Required Accounts**

1. **GitHub** (free) - Version control
2. **Vercel** (free tier) - Deployment + AI Gateway
3. **Convex** (free tier) - Database (1GB free)
4. **Clerk** (free tier) - Auth (up to 5,000 MAU)
5. **OpenAI** (pay-as-you-go) - Whisper API ($0.006/min)
6. **Anthropic** (pay-as-you-go) - Claude API ($3/MTok)

### **Local Development Tools**

```bash
# Node.js 18+ (with npm)
node --version  # Should be v18+ or v20+

# Git
git --version

# VS Code (recommended) or your preferred editor
code --version

# pnpm (faster than npm)
npm install -g pnpm
```

### **Cost Estimate (First Month)**

| Service | Cost | Notes |
|---------|------|-------|
| Vercel Pro | $20 | 100K function executions |
| Convex | $0 | Free tier (1GB) |
| Clerk | $0 | Free tier (<5K users) |
| OpenAI Whisper | $5-10 | 15-20 hours transcription |
| Anthropic Claude | $10-15 | 3-5M tokens |
| **Total** | **$35-45** | For 20-30 beta users |

---

## Project Initialization

### **Step 1: Create Next.js Project**

```bash
# Navigate to your workspace
cd ~/Documents/Orchid\ Automation/Orchid\ Labs/helm

# Create Next.js 15 app
pnpx create-next-app@latest . --typescript --tailwind --app --no-src-dir --import-alias "@/*"

# When prompted:
# ✓ Use App Router? Yes
# ✓ Use Turbopack? Yes (faster dev server)
# ✓ Customize default import alias? No
```

### **Step 2: Install Dependencies**

```bash
# AI SDK packages
pnpm add ai @ai-sdk/openai @ai-sdk/anthropic zod

# Convex (database + real-time)
pnpm add convex

# Clerk (auth)
pnpm add @clerk/nextjs

# UI components (shadcn/ui)
pnpm add @radix-ui/react-slot class-variance-authority clsx tailwind-merge lucide-react

# Dev dependencies
pnpm add -D @types/node @types/react typescript
```

### **Step 3: Project Structure**

```
helm/
├── app/
│   ├── (auth)/
│   │   ├── sign-in/[[...sign-in]]/page.tsx
│   │   └── sign-up/[[...sign-up]]/page.tsx
│   ├── (dashboard)/
│   │   ├── layout.tsx         # Protected layout
│   │   └── page.tsx            # Kanban board
│   ├── api/
│   │   └── transcribe/
│   │       └── route.ts        # Voice → Tasks API
│   ├── layout.tsx              # Root layout (Clerk + Convex)
│   └── page.tsx                # Landing page
├── components/
│   ├── ui/                     # shadcn/ui components
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   └── dialog.tsx
│   ├── voice-recorder.tsx      # Voice input component
│   ├── task-card.tsx           # Individual task
│   ├── kanban-board.tsx        # 4-column board
│   └── kanban-column.tsx       # Single column
├── convex/
│   ├── schema.ts               # Database schema
│   ├── tasks.ts                # Task CRUD functions
│   ├── auth.config.ts          # Clerk + Convex integration
│   └── _generated/             # Auto-generated types
├── lib/
│   ├── utils.ts                # Helper functions
│   └── ai.ts                   # AI SDK clients
├── public/
├── .env.local                  # Environment variables (DO NOT COMMIT)
├── .gitignore
├── convex.json                 # Convex config
├── middleware.ts               # Clerk middleware
├── next.config.js
├── package.json
├── tailwind.config.ts
└── tsconfig.json
```

---

## Convex Database Setup

### **Step 1: Initialize Convex**

```bash
# Initialize Convex in your project
pnpm convex dev

# This will:
# 1. Create convex/ directory
# 2. Open browser to create/link Convex project
# 3. Generate convex.json config
# 4. Start dev server with hot reload
```

### **Step 2: Database Schema**

Create `convex/schema.ts`:

```typescript
import { defineSchema, defineTable } from 'convex/server';
import { v } from 'convex/values';

export default defineSchema({
  // Tasks table
  tasks: defineTable({
    // Core fields
    title: v.string(),
    description: v.optional(v.string()),
    status: v.union(
      v.literal('Backlog'),
      v.literal('In Progress'),
      v.literal('Review'),
      v.literal('Done')
    ),

    // Categorization
    category: v.union(
      v.literal('Business'),
      v.literal('Personal'),
      v.literal('Admin')
    ),
    priority: v.union(
      v.literal('High'),
      v.literal('Medium'),
      v.literal('Low')
    ),

    // Dates
    dueDate: v.optional(v.string()), // ISO string
    createdAt: v.number(), // Unix timestamp
    updatedAt: v.number(),

    // User association
    userId: v.string(), // Clerk user ID

    // Source tracking
    source: v.union(
      v.literal('voice'),
      v.literal('text'),
      v.literal('manual')
    ),
    originalTranscript: v.optional(v.string()),
  })
  .index('by_user', ['userId'])
  .index('by_status', ['userId', 'status'])
  .index('by_category', ['userId', 'category']),

  // Analytics table (optional for MVP)
  analytics: defineTable({
    userId: v.string(),
    event: v.string(), // 'task_created', 'voice_used', etc.
    timestamp: v.number(),
    metadata: v.optional(v.any()),
  })
  .index('by_user', ['userId'])
  .index('by_event', ['event', 'timestamp']),
});
```

### **Step 3: Task Functions**

Create `convex/tasks.ts`:

```typescript
import { mutation, query } from './_generated/server';
import { v } from 'convex/values';

// Create multiple tasks (from voice input)
export const createTasks = mutation({
  args: {
    tasks: v.array(v.object({
      title: v.string(),
      description: v.optional(v.string()),
      category: v.string(),
      priority: v.string(),
      dueDate: v.optional(v.string()),
    })),
    source: v.string(),
    originalTranscript: v.optional(v.string()),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error('Not authenticated');

    const taskIds = [];
    const now = Date.now();

    for (const task of args.tasks) {
      const taskId = await ctx.db.insert('tasks', {
        ...task,
        userId: identity.subject,
        status: 'Backlog',
        source: args.source as any,
        originalTranscript: args.originalTranscript,
        createdAt: now,
        updatedAt: now,
      });
      taskIds.push(taskId);
    }

    return taskIds;
  },
});

// Get all tasks for current user
export const getTasks = query({
  args: {},
  handler: async (ctx) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) return [];

    const tasks = await ctx.db
      .query('tasks')
      .withIndex('by_user', (q) => q.eq('userId', identity.subject))
      .collect();

    return tasks.sort((a, b) => b.createdAt - a.createdAt);
  },
});

// Update task status (for Kanban drag-drop)
export const updateTaskStatus = mutation({
  args: {
    taskId: v.id('tasks'),
    status: v.union(
      v.literal('Backlog'),
      v.literal('In Progress'),
      v.literal('Review'),
      v.literal('Done')
    ),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error('Not authenticated');

    const task = await ctx.db.get(args.taskId);
    if (!task || task.userId !== identity.subject) {
      throw new Error('Task not found or unauthorized');
    }

    await ctx.db.patch(args.taskId, {
      status: args.status,
      updatedAt: Date.now(),
    });
  },
});

// Update task details
export const updateTask = mutation({
  args: {
    taskId: v.id('tasks'),
    title: v.optional(v.string()),
    description: v.optional(v.string()),
    category: v.optional(v.string()),
    priority: v.optional(v.string()),
    dueDate: v.optional(v.string()),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error('Not authenticated');

    const { taskId, ...updates } = args;
    const task = await ctx.db.get(taskId);

    if (!task || task.userId !== identity.subject) {
      throw new Error('Task not found or unauthorized');
    }

    await ctx.db.patch(taskId, {
      ...updates,
      updatedAt: Date.now(),
    });
  },
});

// Delete task
export const deleteTask = mutation({
  args: { taskId: v.id('tasks') },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error('Not authenticated');

    const task = await ctx.db.get(args.taskId);
    if (!task || task.userId !== identity.subject) {
      throw new Error('Task not found or unauthorized');
    }

    await ctx.db.delete(args.taskId);
  },
});
```

---

## Clerk Authentication

### **Step 1: Create Clerk Application**

1. Go to https://clerk.com
2. Sign up / Log in
3. Create new application: "Helm MVP"
4. Enable **Google OAuth** provider
5. Copy API keys

### **Step 2: Configure Clerk in Next.js**

Create `.env.local`:

```bash
# Clerk
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...

# Clerk URLs (localhost for dev)
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/dashboard

# Convex (auto-generated by `convex dev`)
NEXT_PUBLIC_CONVEX_URL=https://your-project.convex.cloud

# OpenAI
OPENAI_API_KEY=sk-...

# Anthropic
ANTHROPIC_API_KEY=sk-ant-...
```

Create `middleware.ts`:

```typescript
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server';

const isPublicRoute = createRouteMatcher([
  '/',
  '/sign-in(.*)',
  '/sign-up(.*)',
  '/api/webhooks(.*)',
]);

export default clerkMiddleware((auth, request) => {
  if (!isPublicRoute(request)) {
    auth().protect();
  }
});

export const config = {
  matcher: [
    '/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)',
    '/(api|trpc)(.*)',
  ],
};
```

### **Step 3: Integrate Clerk + Convex**

Create `convex/auth.config.ts`:

```typescript
export default {
  providers: [
    {
      domain: 'https://your-clerk-domain.clerk.accounts.dev',
      applicationID: 'convex',
    },
  ],
};
```

Update `app/layout.tsx`:

```typescript
import { ClerkProvider } from '@clerk/nextjs';
import { ConvexClientProvider } from '@/components/convex-client-provider';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <ClerkProvider>
          <ConvexClientProvider>
            {children}
          </ConvexClientProvider>
        </ClerkProvider>
      </body>
    </html>
  );
}
```

Create `components/convex-client-provider.tsx`:

```typescript
'use client';

import { ConvexProviderWithClerk } from 'convex/react-clerk';
import { ClerkProvider, useAuth } from '@clerk/nextjs';
import { ConvexReactClient } from 'convex/react';
import { ReactNode } from 'react';

const convex = new ConvexReactClient(process.env.NEXT_PUBLIC_CONVEX_URL!);

export function ConvexClientProvider({ children }: { children: ReactNode }) {
  return (
    <ConvexProviderWithClerk client={convex} useAuth={useAuth}>
      {children}
    </ConvexProviderWithClerk>
  );
}
```

---

## Vercel AI SDK Integration

### **Step 1: Create AI Client**

Create `lib/ai.ts`:

```typescript
import { createOpenAI } from '@ai-sdk/openai';
import { createAnthropic } from '@ai-sdk/anthropic';

// OpenAI client (for Whisper transcription)
export const openai = createOpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

// Anthropic client (for Claude task extraction)
export const anthropic = createAnthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// Model references
export const WHISPER_MODEL = openai.transcription('whisper-1');
export const CLAUDE_MODEL = anthropic('claude-3-5-sonnet-20241022');
```

### **Step 2: Task Extraction Schema**

Create `lib/schemas.ts`:

```typescript
import { z } from 'zod';

export const TaskSchema = z.object({
  title: z.string().describe('Clear, actionable task title'),
  description: z.string().optional().describe('Additional context or details'),
  category: z.enum(['Business', 'Personal', 'Admin']).describe('Task category'),
  priority: z.enum(['High', 'Medium', 'Low']).describe('Task priority'),
  dueDate: z.string().optional().describe('ISO date string if mentioned'),
});

export const TaskExtractionSchema = z.object({
  tasks: z.array(TaskSchema),
});

export type Task = z.infer<typeof TaskSchema>;
export type TaskExtraction = z.infer<typeof TaskExtractionSchema>;
```

---

## Voice Recording Component

Create `components/voice-recorder.tsx`:

```typescript
'use client';

import { useState, useRef } from 'react';
import { Mic, Square } from 'lucide-react';
import { Button } from './ui/button';

interface VoiceRecorderProps {
  onTranscriptComplete: (tasks: any[]) => void;
}

export function VoiceRecorder({ onTranscriptComplete }: VoiceRecorderProps) {
  const [isRecording, setIsRecording] = useState(false);
  const [isProcessing, setIsProcessing] = useState(false);
  const mediaRecorderRef = useRef<MediaRecorder | null>(null);
  const chunksRef = useRef<Blob[]>([]);

  async function startRecording() {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
      const mediaRecorder = new MediaRecorder(stream);
      mediaRecorderRef.current = mediaRecorder;
      chunksRef.current = [];

      mediaRecorder.ondataavailable = (e) => {
        if (e.data.size > 0) {
          chunksRef.current.push(e.data);
        }
      };

      mediaRecorder.onstop = async () => {
        const audioBlob = new Blob(chunksRef.current, { type: 'audio/webm' });
        stream.getTracks().forEach(track => track.stop());
        await processAudio(audioBlob);
      };

      mediaRecorder.start();
      setIsRecording(true);
    } catch (error) {
      console.error('Error starting recording:', error);
      alert('Could not access microphone. Please check permissions.');
    }
  }

  function stopRecording() {
    if (mediaRecorderRef.current && isRecording) {
      mediaRecorderRef.current.stop();
      setIsRecording(false);
      setIsProcessing(true);
    }
  }

  async function processAudio(audioBlob: Blob) {
    try {
      const formData = new FormData();
      formData.append('audio', audioBlob);

      const response = await fetch('/api/transcribe', {
        method: 'POST',
        body: formData,
      });

      if (!response.ok) {
        throw new Error('Transcription failed');
      }

      const data = await response.json();
      onTranscriptComplete(data.tasks);
    } catch (error) {
      console.error('Error processing audio:', error);
      alert('Failed to process audio. Please try again.');
    } finally {
      setIsProcessing(false);
    }
  }

  return (
    <div className="flex flex-col items-center gap-4">
      <Button
        size="lg"
        onClick={isRecording ? stopRecording : startRecording}
        disabled={isProcessing}
        className={isRecording ? 'bg-red-500 hover:bg-red-600' : ''}
      >
        {isProcessing ? (
          <>Processing...</>
        ) : isRecording ? (
          <>
            <Square className="mr-2 h-4 w-4" />
            Stop Recording
          </>
        ) : (
          <>
            <Mic className="mr-2 h-4 w-4" />
            Record Voice
          </>
        )}
      </Button>

      {isRecording && (
        <p className="text-sm text-muted-foreground animate-pulse">
          Recording... Speak your tasks clearly
        </p>
      )}
    </div>
  );
}
```

---

## API Routes

Create `app/api/transcribe/route.ts`:

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { experimental_transcribe as transcribe, generateObject } from 'ai';
import { WHISPER_MODEL, CLAUDE_MODEL } from '@/lib/ai';
import { TaskExtractionSchema } from '@/lib/schemas';
import { auth } from '@clerk/nextjs/server';

export const maxDuration = 60; // 60 seconds timeout

export async function POST(req: NextRequest) {
  try {
    // Check authentication
    const { userId } = await auth();
    if (!userId) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    // Get audio from form data
    const formData = await req.formData();
    const audioFile = formData.get('audio') as Blob;

    if (!audioFile) {
      return NextResponse.json({ error: 'No audio file provided' }, { status: 400 });
    }

    // Convert to Buffer
    const buffer = Buffer.from(await audioFile.arrayBuffer());

    // Step 1: Transcribe with OpenAI Whisper
    console.log('Transcribing audio...');
    const { text: transcript } = await transcribe({
      model: WHISPER_MODEL,
      audio: buffer,
    });

    console.log('Transcript:', transcript);

    // Step 2: Extract tasks with Claude
    console.log('Extracting tasks...');
    const { object: extraction } = await generateObject({
      model: CLAUDE_MODEL,
      schema: TaskExtractionSchema,
      prompt: `Extract actionable tasks from this voice transcript.

For each task:
- Create a clear, concise title (action-oriented)
- Add description if there's extra context
- Categorize as Business, Personal, or Admin
- Set priority based on urgency keywords (urgent/asap = High, important = Medium, else = Low)
- Extract due date if mentioned (convert to ISO string)

Transcript: "${transcript}"

If no tasks are mentioned, return an empty array.`,
    });

    console.log('Extracted tasks:', extraction.tasks);

    return NextResponse.json({
      transcript,
      tasks: extraction.tasks,
      taskCount: extraction.tasks.length,
    });

  } catch (error) {
    console.error('Transcription error:', error);
    return NextResponse.json(
      { error: 'Failed to process audio', details: error.message },
      { status: 500 }
    );
  }
}
```

---

## Kanban Board UI

### **Kanban Board Component**

Create `components/kanban-board.tsx`:

```typescript
'use client';

import { useQuery, useMutation } from 'convex/react';
import { api } from '@/convex/_generated/api';
import { KanbanColumn } from './kanban-column';
import { VoiceRecorder } from './voice-recorder';

const COLUMNS = ['Backlog', 'In Progress', 'Review', 'Done'] as const;

export function KanbanBoard() {
  const tasks = useQuery(api.tasks.getTasks);
  const createTasks = useMutation(api.tasks.createTasks);
  const updateTaskStatus = useMutation(api.tasks.updateTaskStatus);

  async function handleTranscriptComplete(extractedTasks: any[]) {
    if (extractedTasks.length === 0) {
      alert('No tasks found in your recording. Try speaking more clearly.');
      return;
    }

    await createTasks({
      tasks: extractedTasks,
      source: 'voice',
    });
  }

  if (tasks === undefined) {
    return <div>Loading...</div>;
  }

  return (
    <div className="flex flex-col gap-8">
      {/* Voice Recorder */}
      <div className="flex justify-center">
        <VoiceRecorder onTranscriptComplete={handleTranscriptComplete} />
      </div>

      {/* Kanban Columns */}
      <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
        {COLUMNS.map((column) => (
          <KanbanColumn
            key={column}
            title={column}
            tasks={tasks.filter((task) => task.status === column)}
            onTaskMove={updateTaskStatus}
          />
        ))}
      </div>
    </div>
  );
}
```

### **Kanban Column Component**

Create `components/kanban-column.tsx`:

```typescript
'use client';

import { TaskCard } from './task-card';
import { Card } from './ui/card';

interface Task {
  _id: string;
  title: string;
  description?: string;
  category: string;
  priority: string;
  dueDate?: string;
  status: string;
}

interface KanbanColumnProps {
  title: string;
  tasks: Task[];
  onTaskMove: (args: { taskId: string; status: string }) => void;
}

export function KanbanColumn({ title, tasks, onTaskMove }: KanbanColumnProps) {
  return (
    <div className="flex flex-col gap-4">
      <div className="flex items-center justify-between">
        <h2 className="font-semibold text-lg">{title}</h2>
        <span className="text-sm text-muted-foreground">{tasks.length}</span>
      </div>

      <div className="flex flex-col gap-2 min-h-[200px]">
        {tasks.map((task) => (
          <TaskCard
            key={task._id}
            task={task}
            onMove={(status) => onTaskMove({ taskId: task._id, status })}
          />
        ))}

        {tasks.length === 0 && (
          <Card className="p-4 text-center text-muted-foreground">
            No tasks yet
          </Card>
        )}
      </div>
    </div>
  );
}
```

### **Task Card Component**

Create `components/task-card.tsx`:

```typescript
'use client';

import { Card } from './ui/card';
import { Badge } from './ui/badge';
import { useMutation } from 'convex/react';
import { api } from '@/convex/_generated/api';

interface Task {
  _id: string;
  title: string;
  description?: string;
  category: string;
  priority: string;
  dueDate?: string;
  status: string;
}

interface TaskCardProps {
  task: Task;
  onMove: (status: string) => void;
}

export function TaskCard({ task, onMove }: TaskCardProps) {
  const deleteTask = useMutation(api.tasks.deleteTask);

  const priorityColors = {
    High: 'bg-red-100 text-red-800',
    Medium: 'bg-yellow-100 text-yellow-800',
    Low: 'bg-green-100 text-green-800',
  };

  return (
    <Card className="p-4 cursor-pointer hover:shadow-md transition-shadow">
      <div className="flex flex-col gap-2">
        <div className="flex items-start justify-between">
          <h3 className="font-medium">{task.title}</h3>
          <Badge className={priorityColors[task.priority as keyof typeof priorityColors]}>
            {task.priority}
          </Badge>
        </div>

        {task.description && (
          <p className="text-sm text-muted-foreground">{task.description}</p>
        )}

        <div className="flex items-center gap-2">
          <Badge variant="outline">{task.category}</Badge>
          {task.dueDate && (
            <span className="text-xs text-muted-foreground">
              Due: {new Date(task.dueDate).toLocaleDateString()}
            </span>
          )}
        </div>

        {/* Simple move buttons for MVP (drag-drop in v1.0) */}
        <div className="flex gap-2 mt-2">
          {task.status !== 'Backlog' && (
            <button
              onClick={() => onMove('Backlog')}
              className="text-xs text-blue-600 hover:underline"
            >
              ← Backlog
            </button>
          )}
          {task.status !== 'Done' && (
            <button
              onClick={() => onMove(
                task.status === 'Backlog' ? 'In Progress' :
                task.status === 'In Progress' ? 'Review' : 'Done'
              )}
              className="text-xs text-blue-600 hover:underline"
            >
              → Next
            </button>
          )}
          <button
            onClick={() => deleteTask({ taskId: task._id })}
            className="text-xs text-red-600 hover:underline ml-auto"
          >
            Delete
          </button>
        </div>
      </div>
    </Card>
  );
}
```

---

## Deployment

### **Step 1: Deploy to Vercel**

```bash
# Install Vercel CLI
pnpm add -g vercel

# Login to Vercel
vercel login

# Deploy
vercel

# Follow prompts:
# Link to existing project? No
# Project name? helm-mvp
# Which directory? ./
# Override settings? No
```

### **Step 2: Set Environment Variables in Vercel**

```bash
# Via CLI
vercel env add OPENAI_API_KEY
vercel env add ANTHROPIC_API_KEY
vercel env add CLERK_SECRET_KEY
vercel env add NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY
vercel env add NEXT_PUBLIC_CONVEX_URL

# Or via dashboard: https://vercel.com/your-project/settings/environment-variables
```

### **Step 3: Deploy Convex**

```bash
# Deploy Convex functions
pnpm convex deploy

# This pushes your schema and functions to production
```

### **Step 4: Update Clerk Redirect URLs**

In Clerk dashboard:
1. Go to Paths
2. Update redirect URLs to your Vercel domain:
   - `https://your-app.vercel.app/dashboard`

---

## Testing & Launch

### **Pre-Launch Checklist**

- [ ] Voice recording works on desktop Chrome/Safari
- [ ] Voice recording works on mobile browsers
- [ ] Transcription accuracy > 80% (test with 10 recordings)
- [ ] Task extraction creates sensible tasks
- [ ] Tasks appear in Kanban immediately (real-time)
- [ ] Can move tasks between columns
- [ ] Can delete tasks
- [ ] Google OAuth login works
- [ ] User can only see their own tasks
- [ ] App works on mobile browsers (responsive)
- [ ] Error handling works (bad audio, API failures)

### **Beta User Onboarding**

1. **Invite 5 friends first**
   - Send direct link: `https://your-app.vercel.app`
   - Ask them to record 3 tasks
   - Collect feedback via 15-min call

2. **Expand to 20 users**
   - Post on Twitter/LinkedIn
   - Share in relevant Slack/Discord communities
   - Send personal invites

3. **Track Metrics**
   - Week 1: Are users coming back?
   - Week 2: 70%+ retention = success
   - Week 3: Are they using voice or text?
   - Week 4: Survey: "Would you pay $10/month?"

### **Success Metrics (Week 4)**

| Metric | Target | Actual |
|--------|--------|--------|
| Weekly retention | 70%+ | ___ |
| Voice usage % | 80%+ | ___ |
| Tasks created/user/week | 5+ | ___ |
| Task extraction accuracy | 80%+ | ___ |
| Would pay (survey) | 50%+ | ___ |

---

## Next Steps After MVP

If metrics hit targets:

1. **v1.0 Polish (Weeks 5-8)**
   - Add drag & drop (@dnd-kit)
   - Add keyboard shortcuts
   - Dark mode
   - Animations

2. **Phase 2: First Agent (Weeks 9-16)**
   - Meeting Prep Agent
   - MCP integrations (Calendar, Drive, Exa)
   - Inngest workflows

3. **Funding Round**
   - With 100+ users and strong retention, raise pre-seed ($500K)

---

## Troubleshooting

### **Common Issues**

| Issue | Solution |
|-------|----------|
| "Microphone not working" | Check browser permissions, use HTTPS in production |
| "Convex not syncing" | Restart `pnpm convex dev`, check auth integration |
| "Clerk 401 errors" | Verify middleware.ts is configured correctly |
| "AI API rate limits" | Implement request throttling, upgrade API tier |
| "High latency (>5s)" | Check Vercel function region, optimize API calls |

---

## Cost Projections

### **At 100 Users (Month 2)**

| Service | Cost | Usage |
|---------|------|-------|
| Vercel Pro | $20 | 100K functions/month |
| Convex Free | $0 | 1GB data (under limit) |
| Clerk Free | $0 | <5K MAU |
| OpenAI Whisper | $36 | 100 hrs transcription |
| Anthropic Claude | $60 | 20M tokens |
| **Total** | **$116** | |

**Revenue at $20/user:** $2,000
**Gross margin:** 94%

### **At 1,000 Users (Month 6)**

| Service | Cost | Usage |
|---------|------|-------|
| Vercel Pro | $20 | Still under limits |
| Convex Launch | $25 | 10GB data |
| Clerk Free | $0 | Still <5K MAU |
| OpenAI Whisper | $360 | 1,000 hrs |
| Anthropic Claude | $600 | 200M tokens |
| **Total** | **$1,005** | |

**Revenue at $20/user:** $20,000
**Gross margin:** 95%

---

## Support & Resources

- **Vercel AI SDK Docs:** https://ai-sdk.dev
- **Convex Docs:** https://docs.convex.dev
- **Clerk Docs:** https://clerk.com/docs
- **OpenAI Whisper API:** https://platform.openai.com/docs/guides/speech-to-text
- **Anthropic Claude API:** https://docs.anthropic.com

---

**Ready to build?** Start with Step 1: Project Initialization and work through sequentially. Each step builds on the previous one.
