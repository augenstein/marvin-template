# Content Pipeline Pattern

Build an end-to-end content creation workflow from idea capture to publication.

## Why You'd Want This

Creating content consistently is hard. The pipeline breaks down into distinct phases, and each phase can be automated or agent-assisted. Instead of staring at a blank page, you feed ideas into a pipeline and ship finished pieces out the other end.

## Architecture

```
Capture ──→ Research ──→ Draft ──→ Review ──→ Produce ──→ Publish ──→ Log
```

### Phase 1: Capture
Ideas come from everywhere. Centralize them.
- Drop zone: `research/inbox/` for URLs, notes, ideas
- Mobile captures (links shared from your phone)
- Meeting takeaways, conversation threads
- RSS feeds, newsletters, social mentions

### Phase 2: Research
Turn raw ideas into informed outlines.
- Spawn `research-agent` for deep dives on the topic
- Gather competitor content, prior art, community discussions
- Save research output to `research/output/YYYY-MM-DD-<topic>.md`
- Research informs the brief, not the other way around

### Phase 3: Draft
Agent-assisted content creation.
- Spawn `content-agent` with topic + research + brief
- Agent produces outline, then section drafts
- Human reviews and provides voice/direction
- Iterate until the draft is solid

### Phase 4: Review
Quality and editorial check.
- Content-agent reviews for consistency, flow, and gaps
- Human does final editorial pass
- Check against brand voice and style guide
- Verify all claims and links

### Phase 5: Produce
Transform written content into final format.
- **Blog/article**: Format for CMS, add images, SEO metadata
- **Video**: Script to narration (TTS like ElevenLabs), video assembly (Remotion or similar)
- **Social**: Extract key insights, draft platform-specific posts
- **Newsletter**: Compile roundup, write subject line

### Phase 6: Publish
Distribute across channels.
- Publish to primary platform (blog, YouTube, etc.)
- Queue social posts via scheduling tool (Buffer, Typefully, etc.)
- Send newsletter if applicable
- Share in relevant communities

### Phase 7: Log
Track what shipped for accountability and goal tracking.
- Append to `content/log.md` with type, title, URL, goal
- Update `state/goals.md` progress
- Content-shipped skill auto-detects and logs

## Agent Orchestration

```
┌──────────┐     ┌────────────┐     ┌──────────────┐
│  Capture │ ──→ │  research  │ ──→ │content-agent │
│  (human) │     │   -agent   │     │  (drafting)  │
└──────────┘     └────────────┘     └──────┬───────┘
                                           │
                                    ┌──────▼───────┐
                                    │    Review    │
                                    │   (human)   │
                                    └──────┬───────┘
                                           │
                                    ┌──────▼───────┐
                                    │   Produce    │
                                    │ (TTS, video, │
                                    │  formatting) │
                                    └──────┬───────┘
                                           │
                                    ┌──────▼───────┐
                                    │   Publish    │
                                    │ (scheduling, │
                                    │  posting)    │
                                    └──────────────┘
```

## Integration Points

| Phase | Tool Category | Examples |
|-------|--------------|----------|
| Research | Web search, AI providers | Parallel AI, Perplexity, OpenAI |
| Draft | AI agents | content-agent, research-agent |
| Produce (narration) | Text-to-speech | ElevenLabs, Google TTS, OpenAI TTS |
| Produce (video) | Video framework | Remotion, FFmpeg |
| Publish (social) | Social scheduling | Buffer, Typefully, native APIs |
| Publish (video) | Video platform | YouTube API |
| Log | Internal | content/log.md, state/goals.md |

## Key Decisions

| Decision | What we learned |
|----------|----------------|
| Centralized inbox | Ideas captured anywhere, processed in one place |
| Research before drafting | Informed drafts are dramatically better than cold starts |
| Human review is mandatory | Agents draft, humans decide. Never auto-publish. |
| Log everything shipped | Can't improve cadence if you don't track it |
| Platform-native social | Don't cross-post identical content. Reframe per platform. |
| TTS for narration | Converts written content to audio/video at low cost |

## Directory Structure

```
research/
├── inbox/           # Raw captures (URLs, ideas, notes)
└── output/          # Processed research

content/
├── log.md           # All shipped content
├── blogs/           # Blog drafts by topic
├── social/          # Social post drafts
└── videos/          # Video scripts and assets

state/
└── goals.md         # Monthly content targets
```

## Cadence Monitoring

Set monthly targets in `state/goals.md`:
| Goal | Target |
|------|--------|
| Videos | 1/month |
| Written pieces | 2/month |
| Newsletter | 1/month |

Content-agent monitors progress and surfaces gaps:
> "2 videos shipped, 0 written pieces. 12 days left in month."

## Getting Started

1. Create `research/inbox/` as your capture drop zone
2. Start capturing ideas (URLs, notes, voice memos transcribed)
3. Pick one idea and run the full pipeline manually
4. Identify which phases benefit most from automation
5. Set monthly targets in `state/goals.md`
6. Build agent workflows for the phases that slow you down
7. Track everything in `content/log.md`
