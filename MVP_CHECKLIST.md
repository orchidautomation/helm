# MVP v0.2 Implementation Checklist
## 3-4 Week Build Plan

**Goal:** Voice → Tasks → Kanban board with 20-30 beta users

---

## Week 1: Foundation (Days 1-7)

### Day 1: Project Setup
- [ ] Create Next.js 15 project with TypeScript + Tailwind
- [ ] Install dependencies (ai, convex, @clerk/nextjs, zod)
- [ ] Set up project structure (app/, components/, lib/, convex/)
- [ ] Initialize Git repo and first commit
- [ ] Create .env.local with placeholder keys

### Day 2: Convex Database
- [ ] Run `pnpm convex dev` and create Convex project
- [ ] Create `convex/schema.ts` with tasks table
- [ ] Create `convex/tasks.ts` with CRUD functions
- [ ] Test database locally with Convex dashboard
- [ ] Verify real-time subscriptions work

### Day 3: Clerk Authentication
- [ ] Create Clerk account and application
- [ ] Enable Google OAuth provider
- [ ] Add Clerk keys to .env.local
- [ ] Create middleware.ts for route protection
- [ ] Set up sign-in/sign-up pages
- [ ] Integrate Clerk with Convex (auth.config.ts)
- [ ] Test login flow end-to-end

### Day 4-5: AI SDK Setup
- [ ] Add OpenAI and Anthropic API keys to .env.local
- [ ] Create `lib/ai.ts` with client setup
- [ ] Create `lib/schemas.ts` with task extraction schema
- [ ] Test Whisper transcription with sample audio file
- [ ] Test Claude task extraction with sample transcript
- [ ] Verify structured output works correctly

---

## Week 2: Core Features (Days 8-14)

### Day 6-7: Voice Recording Component
- [ ] Create `components/voice-recorder.tsx`
- [ ] Implement browser MediaRecorder API
- [ ] Add recording UI (mic button, status indicator)
- [ ] Handle audio permissions
- [ ] Test recording on desktop browser
- [ ] Test recording on mobile browser
- [ ] Add error handling for unsupported browsers

### Day 8-9: Transcription API Route
- [ ] Create `app/api/transcribe/route.ts`
- [ ] Implement POST handler with Clerk auth check
- [ ] Add Whisper transcription logic
- [ ] Add Claude task extraction logic
- [ ] Return structured JSON response
- [ ] Add error handling and logging
- [ ] Test with Postman/Insomnia
- [ ] Test with actual voice recordings

### Day 10-11: Connect Voice → Database
- [ ] Wire up VoiceRecorder to API route
- [ ] Call Convex createTasks mutation after API response
- [ ] Verify tasks appear in Convex dashboard
- [ ] Add loading states
- [ ] Add success/error notifications
- [ ] Test end-to-end: Record → Transcribe → Save

---

## Week 3: Kanban UI (Days 15-21)

### Day 12-13: Kanban Board Layout
- [ ] Create `components/kanban-board.tsx`
- [ ] Create `components/kanban-column.tsx`
- [ ] Create `components/task-card.tsx`
- [ ] Set up 4 columns: Backlog, In Progress, Review, Done
- [ ] Use Convex useQuery to fetch tasks
- [ ] Group tasks by status
- [ ] Make it responsive (mobile + desktop)

### Day 14-15: Task Management
- [ ] Add task move buttons (← Backlog, → Next)
- [ ] Implement updateTaskStatus mutation
- [ ] Add delete task button
- [ ] Add task editing (inline or modal)
- [ ] Add priority color coding (High=red, Medium=yellow, Low=green)
- [ ] Add category badges
- [ ] Show due dates

### Day 16-17: Polish & UX
- [ ] Install shadcn/ui components (Button, Card, Badge, Dialog)
- [ ] Style voice recorder prominently
- [ ] Add empty states ("No tasks yet")
- [ ] Add loading skeletons
- [ ] Add transitions and animations (subtle)
- [ ] Improve mobile touch targets
- [ ] Add keyboard shortcuts (optional)

### Day 18: Dashboard Layout
- [ ] Create protected layout for /dashboard
- [ ] Add header with user profile (Clerk UserButton)
- [ ] Add logout button
- [ ] Position voice recorder at top
- [ ] Add stats (total tasks, completed today, etc.)
- [ ] Make it look good!

---

## Week 4: Testing & Launch (Days 22-28)

### Day 19-20: Testing
- [ ] Test voice recording quality (clear vs. noisy environments)
- [ ] Test transcription accuracy with 20+ recordings
- [ ] Calculate accuracy rate (target: 80%+)
- [ ] Test task extraction quality
- [ ] Fix prompt if tasks are too generic/weird
- [ ] Test all CRUD operations (create, read, update, delete)
- [ ] Test on different devices (iPhone, Android, Mac, Windows)
- [ ] Test on different browsers (Chrome, Safari, Firefox)
- [ ] Load test with 50+ tasks

### Day 21: Edge Cases & Error Handling
- [ ] Handle microphone permission denied
- [ ] Handle no audio detected
- [ ] Handle API rate limits (OpenAI/Anthropic)
- [ ] Handle network failures (offline mode or error message)
- [ ] Handle invalid audio formats
- [ ] Handle empty transcripts ("no tasks detected")
- [ ] Add retry logic for failed API calls
- [ ] Add helpful error messages (not just "Error occurred")

### Day 22-23: Deployment
- [ ] Deploy to Vercel (`vercel` command)
- [ ] Add all environment variables in Vercel dashboard
- [ ] Deploy Convex functions (`pnpm convex deploy`)
- [ ] Update Clerk redirect URLs to production domain
- [ ] Test production app thoroughly
- [ ] Set up custom domain (optional)
- [ ] Enable Vercel Analytics

### Day 24: Analytics & Monitoring
- [ ] Add basic event tracking (task_created, voice_used)
- [ ] Create Convex analytics table
- [ ] Track voice vs. text input usage
- [ ] Track task completion rate
- [ ] Set up error monitoring (Vercel logs or Sentry)
- [ ] Create internal dashboard to view metrics

### Day 25-26: Beta User Onboarding
- [ ] Invite 5 close friends first
- [ ] Create onboarding flow (3-step tutorial)
- [ ] Send personalized invite emails
- [ ] Create feedback form (Google Forms or Typeform)
- [ ] Schedule 15-min calls with first 5 users
- [ ] Fix critical bugs based on feedback
- [ ] Expand to 20 users (Twitter, LinkedIn, communities)

### Day 27-28: Monitor & Iterate
- [ ] Check metrics daily (retention, usage)
- [ ] Respond to user feedback within 24 hours
- [ ] Fix bugs as they're reported
- [ ] Improve prompts based on task quality feedback
- [ ] Add small features users request (if quick)
- [ ] Document everything for future reference

---

## Success Metrics (End of Week 4)

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| Beta users onboarded | 20+ | ___ | ⬜ |
| Weekly retention (Week 2) | 70%+ | ___ | ⬜ |
| Voice input usage | 80%+ | ___ | ⬜ |
| Task extraction accuracy | 80%+ | ___ | ⬜ |
| Tasks created per user/week | 5+ | ___ | ⬜ |
| User feedback: "Would pay $10/mo" | 50%+ | ___ | ⬜ |

---

## Go/No-Go Decision (End of Week 4)

### ✅ GO (Build v1.0 → Phase 2)
- Weekly retention > 70%
- Users prefer voice over text
- Positive feedback: "I can't go back"
- Users asking for more features

### 🛑 NO-GO (Pivot or Iterate)
- Retention < 50%
- Users abandon after first use
- Prefer text input over voice
- Feedback: "It's okay but I'll stick with Linear/Notion"

---

## Post-MVP Roadmap (If Success)

### v1.0 (Weeks 5-8) - Polish
- [ ] Drag & drop with @dnd-kit
- [ ] Keyboard shortcuts (⌘K command palette)
- [ ] Dark mode
- [ ] Animations (Framer Motion)
- [ ] Text input fallback
- [ ] Export tasks (CSV/JSON)
- [ ] ProductHunt launch

### Phase 2 (Weeks 9-16) - First Agent
- [ ] Meeting Prep Agent
- [ ] MCP integrations (Calendar, Drive, Exa)
- [ ] Inngest workflows
- [ ] Agent status dashboard
- [ ] Human-in-the-loop approval

### Phase 3 (Months 4-6) - Multi-Agent Framework
- [ ] Proposal Generation Agent
- [ ] Research Agent
- [ ] CRM Sync Agent
- [ ] Agent marketplace (templates)

---

## Resources

- **Full Implementation Guide:** `/docs/MVP_IMPLEMENTATION_GUIDE.md`
- **Architecture Docs:** `/docs/ARCHITECTURE.md`
- **Tech Stack Details:** `/docs/TECH_STACK.md`
- **Original Vision:** `/docs/VISION.md`

---

## Daily Standup Template

**What did I build yesterday?**
-

**What am I building today?**
-

**Any blockers?**
-

**Metrics check:**
- New users: ___
- Active users: ___
- Tasks created: ___
- Bugs reported: ___

---

## Quick Commands Reference

```bash
# Development
pnpm dev                    # Start Next.js dev server
pnpm convex dev            # Start Convex dev server (separate terminal)

# Deployment
vercel                     # Deploy to Vercel
pnpm convex deploy         # Deploy Convex functions

# Testing
pnpm build                 # Check for build errors
pnpm lint                  # Run ESLint

# Database
pnpm convex dashboard      # Open Convex dashboard
pnpm convex logs           # View Convex logs
```

---

**Start Date:** ___________
**Target Launch Date:** ___________ (28 days later)
**Actual Launch Date:** ___________

**Good luck! 🚀**
