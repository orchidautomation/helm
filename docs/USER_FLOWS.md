# User Flows & Journeys

## Core User Flows

### Flow 1: Morning Voice Dump → Task Organization

**Scenario:** Brandon wakes up, grabs coffee, and voice-dumps everything on his mind while walking the dog.

```
┌─────────────────────────────────────────────────────────────┐
│  STEP 1: Voice Capture (Mobile App)                         │
└─────────────────────────────────────────────────────────────┘

User opens mobile app → Taps microphone icon → Speaks for 5 minutes:

"Okay, so I have a few things on my mind. First, I have a meeting with
Joe from Acme Corp on Tuesday. I need to prep for that - research his
company, check our CRM for history, and draft a proposal.

Also, I need to follow up with Jane from Beta Inc. We sent the proposal
last week and haven't heard back. Maybe send a friendly nudge.

Oh, and I need to update the Q1 roadmap in Notion before Friday's
board meeting. Need to pull usage stats from Mixpanel too.

Personal stuff: schedule dentist appointment, call mom on her birthday
(Thursday), and research vacation options for March.

Admin: file taxes before deadline, renew LLC registration, review
insurance policy.

That's it for now."

→ Tap "Stop" → "Processing..." (3-5 seconds)

┌─────────────────────────────────────────────────────────────┐
│  STEP 2: Task Extraction & Categorization                   │
└─────────────────────────────────────────────────────────────┘

Main Agent processes transcript:

Tasks Extracted:
1. Research Joe's company (Acme Corp)
2. Review Joe's CRM history
3. Draft proposal for Joe
4. Prep meeting brief for Joe meeting
5. Follow up with Jane (Beta Inc)
6. Update Q1 roadmap in Notion
7. Pull Mixpanel usage stats
8. Schedule dentist appointment
9. Call mom on Thursday (birthday)
10. Research vacation options (March)
11. File taxes
12. Renew LLC registration
13. Review insurance policy

Categories Detected:
- Business Development (tasks 1-5)
- Product & Planning (tasks 6-7)
- Personal (tasks 8-10)
- Administrative (tasks 11-13)

┌─────────────────────────────────────────────────────────────┐
│  STEP 3: Visual Confirmation (Mobile)                        │
└─────────────────────────────────────────────────────────────┘

App displays:

┌────────────────────────────────────────────────────────┐
│  13 tasks created from your voice dump                 │
├────────────────────────────────────────────────────────┤
│                                                        │
│  📊 Business Development (5 tasks)                     │
│  - Research Joe's company                              │
│  - Review Joe's CRM history                            │
│  - Draft proposal for Joe                              │
│  - Prep meeting brief for Joe meeting                  │
│  - Follow up with Jane (Beta Inc)                      │
│                                                        │
│  🎯 Product & Planning (2 tasks)                       │
│  - Update Q1 roadmap in Notion                         │
│  - Pull Mixpanel usage stats                           │
│                                                        │
│  🏡 Personal (3 tasks)                                 │
│  - Schedule dentist appointment                        │
│  - Call mom on Thursday (birthday)                     │
│  - Research vacation options (March)                   │
│                                                        │
│  📋 Administrative (3 tasks)                           │
│  - File taxes                                          │
│  - Renew LLC registration                              │
│  - Review insurance policy                             │
│                                                        │
│  [Edit Tasks]  [Confirm & Organize]                    │
└────────────────────────────────────────────────────────┘

User taps "Confirm & Organize" → Tasks saved

┌─────────────────────────────────────────────────────────────┐
│  STEP 4: Web App - Kanban View                              │
└─────────────────────────────────────────────────────────────┘

User switches to laptop, opens web app. Tasks appear in Kanban board:

┌──────────────────────────────────────────────────────────────┐
│  Business Development                                        │
├────────────────┬────────────────┬─────────────┬─────────────┤
│   Backlog (3)  │ In Progress (1)│  Review (0) │   Done (1)  │
├────────────────┼────────────────┼─────────────┼─────────────┤
│ □ Draft        │ ⚡ Research    │             │ ✓ Review    │
│   proposal     │   Joe's co.    │             │   CRM       │
│   for Joe      │   (Agent       │             │   history   │
│                │   running...)  │             │             │
│ □ Prep meeting │                │             │             │
│   brief        │                │             │             │
│                │                │             │             │
│ □ Follow up    │                │             │             │
│   with Jane    │                │             │             │
└────────────────┴────────────────┴─────────────┴─────────────┘

Agent is already working on "Research Joe's company"!

┌─────────────────────────────────────────────────────────────┐
│  STEP 5: Agent Execution (Background)                        │
└─────────────────────────────────────────────────────────────┘

Research Agent (auto-triggered for "Research Joe's company"):
1. Exa MCP → Search "Acme Corp"
2. Finds website, recent news, funding announcements
3. LinkedIn MCP → Find Joe's profile
4. Compiles report → Saves to Drive

Meeting Prep Agent (waiting for Research Agent to finish):
1. Receives research results
2. Attio MCP → Pull Joe's CRM record (done in parallel)
3. Combines research + CRM context
4. Generates meeting brief
5. Drive MCP → Save brief to /Meeting Briefs
6. Slack MCP → Notify user

Both agents complete in ~2 minutes.

┌─────────────────────────────────────────────────────────────┐
│  STEP 6: Notification & Review                               │
└─────────────────────────────────────────────────────────────┘

User receives notification (push + Slack):

"Meeting prep complete for Joe (Acme Corp). Review brief →"

User clicks link → Opens meeting brief in Drive:

┌────────────────────────────────────────────────────────────┐
│  Meeting Brief: Joe @ Acme Corp                            │
│  Date: Tuesday, Jan 16, 2025 at 2:00 PM                   │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  👤 Person: Joe Smith, VP of Sales                        │
│  - 10 years at Acme Corp                                  │
│  - Previously at BigCo (SaaS sales)                       │
│  - Interests: golf, AI, productivity tools                │
│                                                            │
│  🏢 Company: Acme Corp                                     │
│  - B2B SaaS for logistics                                 │
│  - 50 employees, Series A ($10M)                          │
│  - Recent news: Launched v2.0 last month                  │
│                                                            │
│  📋 CRM History:                                           │
│  - First contact: Dec 10 (intro call)                     │
│  - Pain points: Manual data entry, reporting gaps         │
│  - Budget: $50-75K annual                                 │
│                                                            │
│  💡 Talking Points:                                        │
│  1. Congratulate on v2.0 launch                           │
│  2. Reference their reporting gaps → position solution    │
│  3. Share case study from similar logistics client        │
│  4. Ask about Q1 priorities                               │
│                                                            │
│  ❓ Questions to Ask:                                      │
│  - How is v2.0 adoption going?                            │
│  - What's timeline for evaluating new tools?              │
│  - Who else is involved in decision?                      │
│                                                            │
└────────────────────────────────────────────────────────────┘

User reviews, makes minor edits, ready for meeting.

Kanban board auto-updates:

┌──────────────────────────────────────────────────────────────┐
│  Business Development                                        │
├────────────────┬────────────────┬─────────────┬─────────────┤
│   Backlog (2)  │ In Progress (0)│  Review (0) │   Done (3)  │
├────────────────┼────────────────┼─────────────┼─────────────┤
│ □ Draft        │                │             │ ✓ Research  │
│   proposal     │                │             │   Joe's co. │
│   for Joe      │                │             │             │
│                │                │             │ ✓ Review    │
│ □ Follow up    │                │             │   CRM       │
│   with Jane    │                │             │             │
│                │                │             │ ✓ Prep      │
│                │                │             │   meeting   │
│                │                │             │   brief     │
└────────────────┴────────────────┴─────────────┴─────────────┘
```

**Time saved:** 45 minutes (research + CRM lookup + brief writing)

---

### Flow 2: Reviewing & Refining Tasks

**Scenario:** User checks web app mid-day to review task progress.

```
┌─────────────────────────────────────────────────────────────┐
│  STEP 1: Open Web App Dashboard                             │
└─────────────────────────────────────────────────────────────┘

User sees overview:

┌────────────────────────────────────────────────────────────┐
│  Dashboard - January 15, 2025                              │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  📊 Today's Summary                                        │
│  - 8 tasks in progress (3 by agents, 5 manual)            │
│  - 5 tasks completed                                       │
│  - 2 tasks need review                                     │
│                                                            │
│  🤖 Agent Activity                                         │
│  ✓ Meeting Prep Agent (2 completions)                     │
│  ⚡ Proposal Generator (running, 70% done)                 │
│  ✓ CRM Sync Agent (1 completion)                          │
│                                                            │
│  ⚠ Needs Attention                                         │
│  - "Draft proposal for Joe" → Agent asks: Budget range?   │
│  - "Follow up with Jane" → Manual task, overdue 2 days    │
│                                                            │
└────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  STEP 2: Respond to Agent Question                          │
└─────────────────────────────────────────────────────────────┘

User clicks "Draft proposal for Joe" task:

┌────────────────────────────────────────────────────────────┐
│  Task: Draft proposal for Joe                              │
│  Status: Waiting for user input                            │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  🤖 Proposal Generator Agent:                              │
│  "I'm drafting the proposal for Acme Corp. Based on CRM   │
│  notes, budget is $50-75K. Should I propose:              │
│                                                            │
│  A) $60K (mid-range, safe bet)                            │
│  B) $72K (premium tier, more services)                    │
│  C) Let me customize the range"                           │
│                                                            │
│  [Option A] [Option B] [Option C]                         │
│                                                            │
└────────────────────────────────────────────────────────────┘

User selects "Option B" → Agent resumes

Proposal completed in 2 minutes → Saved to Drive

┌─────────────────────────────────────────────────────────────┐
│  STEP 3: Manually Complete Overdue Task                     │
└─────────────────────────────────────────────────────────────┘

User clicks "Follow up with Jane":

┌────────────────────────────────────────────────────────────┐
│  Task: Follow up with Jane (Beta Inc)                      │
│  Status: Overdue (created Jan 13, due Jan 15)              │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  💡 Suggested Agent: Sales Follow-Up Agent                 │
│  "I can draft a friendly follow-up email and send it      │
│  via Gmail. Should I proceed?"                            │
│                                                            │
│  [Yes, Send Follow-Up] [I'll Do It Manually]              │
│                                                            │
└────────────────────────────────────────────────────────────┘

User clicks "Yes, Send Follow-Up"

Agent drafts email → User reviews draft → Approves → Sent

CRM auto-updated with "Follow-up sent on Jan 15"

┌─────────────────────────────────────────────────────────────┐
│  STEP 4: Use Side Chat for Query                            │
└─────────────────────────────────────────────────────────────┘

User opens side chat panel (right side of screen):

┌────────────────────────────────────────────────────────────┐
│  💬 Chat                                                   │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  You: What tasks are blocked on external dependencies?    │
│                                                            │
│  🤖 Assistant: I found 2 tasks waiting on others:         │
│                                                            │
│  1. "Update Q1 roadmap" is blocked on:                    │
│     - "Pull Mixpanel stats" (agent running, ETA 5 min)   │
│                                                            │
│  2. "File taxes" is blocked on:                           │
│     - Waiting for W2 from accountant (external)          │
│                                                            │
│  Would you like me to send a reminder to your accountant? │
│                                                            │
│  [Yes, Send Reminder] [No Thanks]                         │
│                                                            │
└────────────────────────────────────────────────────────────┘

User: "Yes, send reminder"

Agent sends Slack message to accountant, marks task with reminder sent.
```

**Time saved:** 20 minutes (email drafting, status checking, reminders)

---

### Flow 3: Progressive Learning - Creating New Sub-Agent

**Scenario:** User notices recurring pattern → System suggests creating specialized agent.

```
┌─────────────────────────────────────────────────────────────┐
│  STEP 1: System Detects Pattern                             │
└─────────────────────────────────────────────────────────────┘

After 2 weeks of use, system analyzes task history:

Pattern Detected:
- User creates "Client update for [Client]" tasks 5x/week
- All follow same structure:
  1. Pull recent activity from CRM
  2. Check project status in Linear/Notion
  3. Draft email update
  4. Send to client
  5. Log in CRM

System triggers notification:

┌────────────────────────────────────────────────────────────┐
│  💡 Suggestion: Create "Client Update Agent"                │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  I noticed you send client updates ~5x/week following a   │
│  consistent pattern. Should I create a specialized agent  │
│  to automate this?                                        │
│                                                            │
│  The agent would:                                         │
│  ✓ Pull CRM activity for the client                      │
│  ✓ Check project status (Notion/Linear)                  │
│  ✓ Draft weekly update email                             │
│  ✓ Send for your review before sending                   │
│  ✓ Log update in CRM                                      │
│                                                            │
│  Estimated time saved: 30 min/week                        │
│                                                            │
│  [Create Agent] [Customize First] [Not Interested]        │
│                                                            │
└────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  STEP 2: User Customizes Agent                              │
└─────────────────────────────────────────────────────────────┘

User clicks "Customize First" → Agent configuration wizard:

┌────────────────────────────────────────────────────────────┐
│  Create Client Update Agent                                │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Step 1: Name Your Agent                                  │
│  [Client Weekly Update Agent_________________]            │
│                                                            │
│  Step 2: Describe Your Workflow (Your Words)              │
│  "Every Friday, pull this week's activity from Attio for  │
│  the client, check their project board in Notion, and     │
│  draft an email highlighting progress and next steps.     │
│  Send me the draft to review. After I approve, send it    │
│  and log in CRM."                                         │
│                                                            │
│  Step 3: Grant MCP Access                                 │
│  ✓ Attio (read client records, write notes)              │
│  ✓ Notion (read project boards)                           │
│  ✓ Gmail (send emails after approval)                     │
│  □ Slack (optional: notify you when draft ready)          │
│                                                            │
│  Step 4: Trigger Conditions                               │
│  When: [Every Friday at 9 AM ▼]                           │
│  Or: When task contains keywords: [client update]         │
│                                                            │
│  Step 5: Review Settings                                  │
│  ☑ Require approval before sending email                  │
│  ☑ QA Agent review for typos                              │
│  ☐ Auto-send if no response in 24 hours                   │
│                                                            │
│  [Preview Agent] [Save & Test] [Cancel]                   │
│                                                            │
└────────────────────────────────────────────────────────────┘

User clicks "Save & Test"

┌─────────────────────────────────────────────────────────────┐
│  STEP 3: Test Agent with Sample Task                        │
└─────────────────────────────────────────────────────────────┘

System creates test task: "Client update for Acme Corp"

Agent runs:
1. Attio MCP → Pull Acme Corp activity (last 7 days)
2. Notion MCP → Check Acme project board status
3. Draft email:

   Subject: Acme Corp - Weekly Update (Jan 15)

   Hi Joe,

   Quick update on this week's progress:

   ✓ Completed: Dashboard redesign (launched Tuesday)
   ✓ In Progress: API integration (70% done, on track)
   → Next: User testing (starting Monday)

   No blockers. On track for Jan 31 delivery.

   Let me know if questions!

   Best,
   Brandon

4. Shows draft to user:

┌────────────────────────────────────────────────────────────┐
│  🤖 Client Update Agent - Draft Ready                      │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  [Email draft shown above]                                │
│                                                            │
│  Quality Check: ✓ No typos ✓ Professional tone            │
│                                                            │
│  [Approve & Send] [Edit First] [Regenerate]               │
│                                                            │
└────────────────────────────────────────────────────────────┘

User approves → Email sent → CRM updated → Task marked done

┌─────────────────────────────────────────────────────────────┐
│  STEP 4: Agent Added to Library                             │
└─────────────────────────────────────────────────────────────┘

Confirmation:

┌────────────────────────────────────────────────────────────┐
│  ✓ Agent Created Successfully!                             │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  "Client Weekly Update Agent" is now active.              │
│                                                            │
│  It will auto-trigger:                                    │
│  - Every Friday at 9 AM                                   │
│  - When you say "client update" in voice dump             │
│                                                            │
│  Want to share this agent with the community?             │
│  [Publish to Marketplace] [Keep Private]                  │
│                                                            │
└────────────────────────────────────────────────────────────┘

User keeps it private for now.

Next Friday, agent auto-runs for all active clients!
```

**Time saved:** 2.5 hours/week (5 clients × 30 min each)

---

### Flow 4: Agent Marketplace - Installing Community Agent

**Scenario:** User discovers helpful agent created by another user.

```
┌─────────────────────────────────────────────────────────────┐
│  STEP 1: Browse Marketplace                                 │
└─────────────────────────────────────────────────────────────┘

User clicks "Agent Marketplace" tab:

┌────────────────────────────────────────────────────────────┐
│  🛍 Agent Marketplace                                       │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  🔥 Trending This Week                                     │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ 📞 Call Summary Agent           ★★★★★ (342 reviews) │ │
│  │ Automatically summarize call transcripts from        │ │
│  │ Airtable and update your CRM.                        │ │
│  │ By: @founder_tools | Free                            │ │
│  │ [Install] [Preview]                                  │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │ 📝 Meeting Minutes Agent        ★★★★☆ (156 reviews) │ │
│  │ Generate meeting minutes and action items from       │ │
│  │ calendar invites + notes.                            │ │
│  │ By: @productivity_hacks | $5/month                   │ │
│  │ [Install] [Preview]                                  │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  📂 Categories: [Sales] [Content] [Support] [Finance]     │
│  🔍 Search: [________________________________]             │
│                                                            │
└────────────────────────────────────────────────────────────┘

User clicks "Preview" on Call Summary Agent

┌─────────────────────────────────────────────────────────────┐
│  STEP 2: Review Agent Details                               │
└─────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│  📞 Call Summary Agent                                     │
│  By: @founder_tools | v2.1.0 | ★★★★★ (342 reviews)        │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Description:                                             │
│  Automatically processes call transcripts from Airtable   │
│  and creates concise summaries with:                      │
│  - Key discussion points                                  │
│  - Action items                                           │
│  - Next steps                                             │
│  - Sentiment analysis                                     │
│                                                            │
│  Then updates your Attio CRM with the summary.           │
│                                                            │
│  MCP Access Required:                                     │
│  ✓ Airtable (read transcripts table)                     │
│  ✓ Attio (write summaries to contacts)                   │
│                                                            │
│  Reviews:                                                 │
│  ⭐⭐⭐⭐⭐ "Saves me 2 hours/week!" - @sales_founder       │
│  ⭐⭐⭐⭐⭐ "Works perfectly with my workflow" - @agency_ceo│
│  ⭐⭐⭐⭐☆ "Good but needs LinkedIn integration" - @vc    │
│                                                            │
│  [Install Agent] [View Source Code] [Contact Author]      │
│                                                            │
└────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  STEP 3: Install & Configure                                │
└─────────────────────────────────────────────────────────────┘

User clicks "Install Agent"

┌────────────────────────────────────────────────────────────┐
│  Install Call Summary Agent                                │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  This agent needs access to:                              │
│  ✓ Airtable → Your "Transcripts" base                    │
│  ✓ Attio → Your CRM contacts                             │
│                                                            │
│  Grant permissions? [Yes, Grant Access] [Cancel]          │
│                                                            │
└────────────────────────────────────────────────────────────┘

User grants access → Installation completes

┌────────────────────────────────────────────────────────────┐
│  ✓ Agent Installed!                                        │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Call Summary Agent is ready to use.                      │
│                                                            │
│  Test it now with a sample transcript?                    │
│  [Run Test] [Configure Settings] [Done]                   │
│                                                            │
└────────────────────────────────────────────────────────────┘

User runs test → Agent processes sample transcript → CRM updated

Agent now auto-triggers whenever new transcripts appear in Airtable!
```

**Time saved:** 2 hours/week (call summarization)

---

## Key Interaction Patterns

### Pattern 1: Voice-First Input

**Design Principle:** Minimize friction for thought capture.

```
Mobile App:
┌────────────────────────────────┐
│                                │
│                                │
│         [   🎤   ]             │
│      Hold to Record            │
│                                │
│                                │
│   Recent Voice Dumps:          │
│   - Today at 8:30 AM (5 min)   │
│   - Yesterday at 9:15 AM       │
│                                │
└────────────────────────────────┘
```

**Best Practices:**
- Single tap to start recording (no menu diving)
- Visual waveform feedback while speaking
- Auto-stop after 30s silence (or manual stop)
- Immediate processing (< 5s to see tasks)

---

### Pattern 2: Kanban Task Management

**Design Principle:** Spatial organization mirrors mental model.

```
┌───────────────────────────────────────────────────────────┐
│  [Business Development ▼] [+ New Category]                │
├─────────────┬─────────────┬────────────┬─────────────────┤
│  Backlog    │ In Progress │   Review   │      Done       │
├─────────────┼─────────────┼────────────┼─────────────────┤
│ □ Task 1    │ ⚡ Task 3   │ 👁 Task 5  │ ✓ Task 7       │
│   Manual    │   (Agent)   │   (Human)  │   (Complete)   │
│             │             │            │                 │
│ □ Task 2    │ ⚡ Task 4   │            │ ✓ Task 8       │
│             │   (Agent)   │            │                 │
└─────────────┴─────────────┴────────────┴─────────────────┘
```

**Visual Cues:**
- ⚡ = Agent actively working
- 👁 = Requires human review
- ✓ = Completed
- □ = Not started
- 🔴 = Blocked/issue

**Drag & Drop:**
- Move tasks between columns
- Reorder by priority (drag up/down)
- Assign to different category (drag to different board)

---

### Pattern 3: Side Chat Queries

**Design Principle:** Natural language access to all task data.

```
Example Queries:

"What tasks are overdue?"
→ Agent lists 3 overdue tasks with suggested actions

"Show me all Business Dev tasks for Acme Corp"
→ Filters Kanban to show only Acme-related tasks

"What did I accomplish last week?"
→ Summary: 23 tasks completed, 5 by agents, 18 manual

"Which agent saved me the most time?"
→ Meeting Prep Agent: 3.5 hours saved (7 executions)

"Create a task to follow up with Joe in 2 weeks"
→ Task created, scheduled for Jan 29

"Remind me to review proposals every Friday at 3 PM"
→ Recurring reminder created
```

**Chat Capabilities:**
- Task creation via chat
- Filter/search tasks
- Analytics queries
- Agent performance insights
- Scheduling & reminders

---

## Mobile vs. Web Experience

### Mobile (iOS/Android)
- **Primary use:** Voice input, quick task checks, notifications
- **Features:**
  - Voice recording (primary input)
  - Task list view (simpler than Kanban)
  - Push notifications
  - Quick actions (mark done, snooze, delegate to agent)
  - Offline mode (sync when online)

### Web (Desktop)
- **Primary use:** Task organization, agent config, detailed review
- **Features:**
  - Full Kanban board
  - Drag & drop task management
  - Agent marketplace & configuration
  - Side chat panel
  - Multi-category view
  - Analytics dashboard

**Sync:** Real-time via WebSocket (changes on mobile appear instantly on web, vice versa)

---

## Next Steps

Use these flows to:
1. **Design mockups** - Wireframe each screen/state
2. **User testing** - Walk through flows with potential users
3. **Prioritize features** - Which flows are MVP-critical?
4. **Engineer sprints** - Break flows into implementable chunks
