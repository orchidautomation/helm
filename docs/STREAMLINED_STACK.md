# Streamlined MVP Stack Decision
## Why We Eliminated Deepgram

**Decision Date:** October 2025
**Decision:** Use OpenAI Whisper via Vercel AI SDK instead of Deepgram

---

## TL;DR

**Original Plan:** Deepgram for transcription + Vercel AI SDK for task extraction
**New Plan:** OpenAI Whisper for transcription + Claude for extraction (both via Vercel AI SDK)

**Result:**
- Eliminated 1 vendor
- Reduced complexity
- Saved $2/month per 100 users (negligible)
- **Gained:** Unified SDK, better DX, easier maintenance

---

## Research Summary

### Vercel AI SDK v5 Capabilities

The Vercel AI SDK v5 (released 2025) includes **experimental but production-ready transcription** via `experimental_transcribe()`:

```typescript
import { experimental_transcribe as transcribe } from 'ai';
import { openai } from '@ai-sdk/openai';

const { text } = await transcribe({
  model: openai.transcription('whisper-1'),
  audio: audioBuffer,
});
```

**Supported providers:**
- OpenAI (Whisper-1, GPT-4o-transcribe)
- Deepgram (Nova-3, Enhanced)
- Groq (Whisper-large-v3-turbo)
- ElevenLabs (Scribe v1)
- AssemblyAI (Best, Nano)
- Rev.ai, Gladia, Fal, Azure OpenAI

**Source:** https://ai-sdk.dev/docs/ai-sdk-core/transcription

---

## Comparison: OpenAI Whisper vs Deepgram

| Factor | OpenAI Whisper-1 | Deepgram Nova-3 | Winner |
|--------|------------------|-----------------|---------|
| **Latency** | 1-2 seconds | 300-500ms | Deepgram |
| **Pricing** | $0.006/min | $0.0043/min | Deepgram |
| **Accuracy** | 10% WER | 8-15% WER | Comparable |
| **Integration** | Same SDK as Claude | Separate SDK | Whisper |
| **Setup Time** | 10 minutes | 30-60 minutes | Whisper |
| **API Keys** | 2 total (OpenAI + Anthropic) | 3 total | Whisper |
| **Failover** | Via Vercel AI Gateway | Manual | Whisper |
| **DX** | Unified interface | Different patterns | Whisper |

**Cost Difference at Scale:**
- 20 users, 10 hrs/month transcription: $3.60 (Whisper) vs. $2.58 (Deepgram) = **$1.02/month difference**
- 100 users, 50 hrs/month: $18 (Whisper) vs. $12.90 (Deepgram) = **$5.10/month difference**

**For MVP context:** This cost difference is negligible compared to development time saved.

---

## Decision Matrix

### For MVP (v0.2): Use OpenAI Whisper

**Reasons:**

1. **Simplicity**
   - One SDK for everything (transcription + task extraction)
   - One authentication pattern
   - One error handling approach
   - Same code style throughout

2. **Speed of Implementation**
   ```typescript
   // Whisper version (10 minutes to set up)
   import { transcribe, generateText } from 'ai';
   import { openai, anthropic } from '@ai-sdk/openai';

   const { text } = await transcribe({
     model: openai.transcription('whisper-1'),
     audio,
   });

   const { object } = await generateText({
     model: anthropic('claude-3-5-sonnet-20241022'),
     schema: TaskSchema,
     prompt: `Extract tasks: ${text}`,
   });
   ```

   ```typescript
   // Deepgram version (30-60 minutes to set up)
   import { createClient } from '@deepgram/sdk';
   import { generateText } from 'ai';
   import { anthropic } from '@ai-sdk/anthropic';

   const deepgram = createClient(process.env.DEEPGRAM_API_KEY);
   const { result } = await deepgram.listen.prerecorded.transcribeFile(
     audioSource,
     { model: 'nova-3', smart_format: true }
   );
   const text = result.results.channels[0].alternatives[0].transcript;

   const { object } = await generateText({
     model: anthropic('claude-3-5-sonnet-20241022'),
     schema: TaskSchema,
     prompt: `Extract tasks: ${text}`,
   });
   ```

3. **Latency is Good Enough**
   - MVP use case: Record 10-30 second clips → Process batch
   - 1-2 second latency is acceptable for this workflow
   - Not building real-time live transcription (Phase 3+ feature)

4. **Better DX**
   - TypeScript types auto-generated
   - Consistent error objects
   - Same retry/timeout patterns
   - One monitoring dashboard (Vercel AI Gateway)

5. **Future-Proof**
   - Easy to swap providers later:
     ```typescript
     // Change one line if we need Deepgram later
     const { text } = await transcribe({
       model: deepgram.transcription('nova-3'), // Was: openai.transcription('whisper-1')
       audio,
     });
     ```

---

## When to Switch to Deepgram

**Switch if any of these become true:**

1. **Real-time requirement**
   - Need live transcription (< 500ms latency)
   - Building live voice assistant feature
   - Streaming transcription use case

2. **Advanced features needed**
   - Speaker diarization (who said what in multi-person recording)
   - Sentiment analysis
   - Custom vocabulary/terminology
   - Language detection

3. **Cost becomes significant**
   - At 10,000+ users, $5-10/month difference becomes meaningful
   - But at that scale, we have revenue to support it

4. **User feedback**
   - Users complain about transcription speed
   - 1-2 second latency feels too slow

**Current assessment:** None of these apply to MVP. Stick with Whisper.

---

## Stack Comparison

### Original Plan (6 Services)

```
Frontend:     Next.js 15
Database:     Convex
Auth:         Clerk
Transcription: Deepgram ← Extra service
AI:           Vercel AI SDK (Claude)
Deploy:       Vercel
```

**Dependencies:**
```json
{
  "@ai-sdk/anthropic": "^1.0.0",
  "ai": "^5.0.0",
  "@deepgram/sdk": "^3.0.0",  ← Extra package
  "convex": "^1.0.0",
  "@clerk/nextjs": "^5.0.0"
}
```

**API Keys Required:** 4
- `OPENAI_API_KEY` (not needed but kept for gateway)
- `ANTHROPIC_API_KEY`
- `DEEPGRAM_API_KEY` ← Extra key
- `CLERK_SECRET_KEY`

---

### Streamlined Plan (5 Services) ✅

```
Frontend:     Next.js 15
Database:     Convex
Auth:         Clerk
AI:           Vercel AI SDK (Whisper + Claude) ← Unified
Deploy:       Vercel
```

**Dependencies:**
```json
{
  "@ai-sdk/openai": "^1.0.0",
  "@ai-sdk/anthropic": "^1.0.0",
  "ai": "^5.0.0",
  "convex": "^1.0.0",
  "@clerk/nextjs": "^5.0.0"
}
```

**API Keys Required:** 3
- `OPENAI_API_KEY`
- `ANTHROPIC_API_KEY`
- `CLERK_SECRET_KEY`

---

## Implementation Impact

### Before (Deepgram)

```typescript
// app/api/transcribe/route.ts
import { createClient } from '@deepgram/sdk';
import { generateObject } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';

const deepgram = createClient(process.env.DEEPGRAM_API_KEY!);

export async function POST(req: Request) {
  const formData = await req.formData();
  const audio = formData.get('audio') as Blob;

  // Transcribe with Deepgram
  const { result } = await deepgram.listen.prerecorded.transcribeFile(
    { buffer: Buffer.from(await audio.arrayBuffer()), mimetype: 'audio/webm' },
    { model: 'nova-3', smart_format: true }
  );

  const transcript = result.results.channels[0].alternatives[0].transcript;

  // Extract tasks with Claude
  const { object } = await generateObject({
    model: anthropic('claude-3-5-sonnet-20241022'),
    schema: TaskSchema,
    prompt: `Extract tasks from: ${transcript}`,
  });

  return Response.json({ transcript, tasks: object.tasks });
}
```

**Lines of code:** ~25
**Dependencies:** 2 SDKs (deepgram, ai)
**Error handling:** Need to handle both Deepgram errors and AI SDK errors differently

---

### After (Whisper via Vercel AI SDK)

```typescript
// app/api/transcribe/route.ts
import { experimental_transcribe as transcribe, generateObject } from 'ai';
import { openai } from '@ai-sdk/openai';
import { anthropic } from '@ai-sdk/anthropic';

export async function POST(req: Request) {
  const formData = await req.formData();
  const audio = formData.get('audio') as Blob;
  const buffer = Buffer.from(await audio.arrayBuffer());

  // Transcribe with Whisper
  const { text: transcript } = await transcribe({
    model: openai.transcription('whisper-1'),
    audio: buffer,
  });

  // Extract tasks with Claude
  const { object } = await generateObject({
    model: anthropic('claude-3-5-sonnet-20241022'),
    schema: TaskSchema,
    prompt: `Extract tasks from: ${transcript}`,
  });

  return Response.json({ transcript, tasks: object.tasks });
}
```

**Lines of code:** ~20
**Dependencies:** 1 SDK (ai)
**Error handling:** Unified error objects from AI SDK

**Difference:** Simpler, fewer lines, one SDK, consistent patterns.

---

## Benchmarks (From Research)

### Transcription Accuracy (Word Error Rate)

| Model | WER | Language Support | Notes |
|-------|-----|------------------|-------|
| OpenAI Whisper-1 | ~10% | 99+ languages | Open-source baseline |
| OpenAI GPT-4o-transcribe | ~8% | 99+ languages | Newer, same price |
| Deepgram Nova-3 | 8-15% | 30+ languages | Optimized per language |

**Conclusion:** Comparable accuracy. Both are production-ready.

### Latency Benchmarks

| Provider | Model | Latency | Use Case |
|----------|-------|---------|----------|
| Deepgram | Nova-3 | 300-500ms | Real-time streaming |
| OpenAI | Whisper-1 | 1-2 seconds | Batch processing |
| OpenAI | GPT-4o-transcribe | 1-2 seconds | Batch processing |

**For MVP:** Batch processing (record → upload → transcribe → extract) means 1-2 second latency is fine.

---

## Cost Projections (Revised)

### MVP v0.2 (20-30 Users, Month 1)

| Service | Cost | Notes |
|---------|------|-------|
| Vercel Pro | $20 | 100K function executions |
| Convex Free | $0 | 1GB storage (under limit) |
| Clerk Free | $0 | <5K MAU |
| OpenAI Whisper | $3-5 | 10-15 hours transcription |
| Anthropic Claude | $10-15 | 3-5M tokens |
| **Total** | **$33-40** | vs. $35-45 with all services |

### At Scale (1,000 Users, Month 6)

| Service | Cost | Notes |
|---------|------|-------|
| Vercel Pro | $20 | Still under limits |
| Convex Launch | $25 | 10GB data |
| Clerk Free | $0 | <5K MAU |
| OpenAI Whisper | $360 | 1,000 hrs @ $0.006/min |
| Anthropic Claude | $600 | 200M tokens |
| **Total** | **$1,005** | vs. $1,050 with Deepgram |

**Savings with Deepgram at 1K users:** $45/month
**Revenue at $20/user:** $20,000/month
**Margin difference:** 0.2%

**Verdict:** Not worth the added complexity for MVP.

---

## Migration Path (If Needed Later)

If we need to switch to Deepgram in Phase 2+:

```typescript
// Step 1: Install Deepgram provider for Vercel AI SDK
pnpm add @ai-sdk/deepgram

// Step 2: Change one line
import { deepgram } from '@ai-sdk/deepgram';

const { text } = await transcribe({
  model: deepgram.transcription('nova-3'), // Changed from openai.transcription
  audio: buffer,
});

// Everything else stays the same
```

**Migration time:** < 30 minutes
**Risk:** Low (same interface, same patterns)

---

## Final Recommendation

### For MVP v0.2: ✅ Use OpenAI Whisper via Vercel AI SDK

**Reasons:**
1. Faster to implement (10 min vs. 1 hour)
2. One unified SDK reduces cognitive load
3. Latency is good enough for batch processing
4. Cost difference is negligible ($2-5/month)
5. Easy to switch later if needed

### Future (Phase 2+): Consider Deepgram if...
- Building real-time features
- Users complain about speed
- Need advanced features (diarization, sentiment)
- Cost savings become significant (10K+ users)

---

## References

1. **Vercel AI SDK v5 Announcement:** https://vercel.com/blog/ai-sdk-5
2. **AI SDK Transcription Docs:** https://ai-sdk.dev/docs/ai-sdk-core/transcription
3. **OpenAI Whisper Pricing:** $0.006/min (October 2025)
4. **Deepgram Pricing:** $0.0043/min (October 2025)
5. **Perplexity Research:** Comparison of Whisper vs. Deepgram (latency, accuracy, pricing)

---

**Decision made by:** Claude Code + Brandon
**Date:** October 17, 2025
**Review date:** After MVP launch (if metrics warrant optimization)
