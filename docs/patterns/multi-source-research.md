# Multi-Source Research Pattern

Fan out a research question to multiple AI providers and synthesize the results.

## Why You'd Want This

No single AI provider has all the answers. Each has different training data, different search capabilities, and different reasoning strengths. Fanning out to multiple sources and synthesizing gives you more comprehensive, more reliable research.

## Architecture

```
                    Research Question
                          │
                ┌─────────┼─────────┐
                │         │         │
                ▼         ▼         ▼
           ┌────────┐ ┌────────┐ ┌────────┐
           │Source 1 │ │Source 2 │ │Source 3 │
           │(Web    │ │(Deep   │ │(Search-│
           │Search) │ │Research│ │augment)│
           └───┬────┘ └───┬────┘ └───┬────┘
               │          │          │
               ▼          ▼          ▼
          findings_1  findings_2  findings_3
               │          │          │
               └──────────┼──────────┘
                          │
                          ▼
                   ┌─────────────┐
                   │  Synthesis  │
                   │  (combine,  │
                   │   dedup,    │
                   │  attribute) │
                   └──────┬──────┘
                          │
                          ▼
                   Combined Output
```

## Two Modes

### Quick Mode
Single source, fast answer. Use for:
- Simple fact-finding ("What version of X was released?")
- Documentation lookups
- Quick definitions or explanations
- News checks

Workflow: search with available tools, compile, respond inline.

### Deep Mode
Multiple sources, synthesized answer. Use for:
- Content creation research (blog posts, talks, videos)
- Competitive analysis
- Technical deep dives
- Complex topics with multiple perspectives
- When citations and thoroughness matter

Workflow: fan out to multiple providers, gather results, synthesize, save to file.

## Source Types

| Source Type | Strengths | Examples |
|------------|-----------|----------|
| **Web search** | Current information, broad coverage | Parallel AI, Brave Search, Google |
| **Deep research AI** | Thorough analysis, reasoning | OpenAI deep research, Claude |
| **Search-augmented AI** | Cited answers, source attribution | Perplexity, Gemini with grounding |
| **Specialized search** | Domain expertise | GitHub code search, arxiv, HN |

## Deep Research Workflow

### Step 1: Parse the Question
Extract:
- Main topic (for file naming)
- Key aspects to investigate
- Specific angles or constraints

### Step 2: Plan Search Strategy
Identify 4-6 distinct search angles:
```
Topic: "API gateway patterns for AI agents"
Angles:
1. Current best practices (general web search)
2. Open-source implementations (GitHub search)
3. Production case studies (deep research AI)
4. Community discussions (HN, Reddit, forums)
5. Academic/technical papers (search-augmented AI)
```

### Step 3: Execute in Parallel
Kick off searches simultaneously where possible:
- Start slow sources first (deep research APIs may take minutes)
- Run quick searches while waiting
- Gather from specialized sources

### Step 4: Collect Results
Save each source's findings separately:
```
research/output/
├── YYYY-MM-DD-topic_source1.md
├── YYYY-MM-DD-topic_source2.md
├── YYYY-MM-DD-topic_source3.md
└── YYYY-MM-DD-topic_combined.md    # Final synthesis
```

### Step 5: Synthesize
Combine findings into a single coherent output:
- **Deduplicate**: Multiple sources often cover the same points
- **Attribute**: Note which source provided each finding
- **Reconcile**: Highlight areas of agreement and disagreement
- **Prioritize**: Lead with the most relevant and well-supported findings
- **Identify gaps**: Note what wasn't well-covered by any source

### Step 6: Save and Report
Write the combined output to `research/output/YYYY-MM-DD-<topic>.md`:

```markdown
# Research: {Topic}
Date: {YYYY-MM-DD}
Mode: Deep (multi-source)

## Executive Summary
3-5 sentences covering the key findings.

## Detailed Findings

### {Aspect 1}
- Finding (Source: web search)
- Finding (Source: deep research)
- Conflicting view (Source: community discussion)

### {Aspect 2}
...

## Source Summary
| Source | What It Contributed |
|--------|-------------------|
| Web search | Current state, recent developments |
| Deep research | Historical context, analysis |
| Community | Practical experience, gotchas |

## Open Questions
- Areas that need further investigation
- Topics where sources disagreed
```

## Key Decisions

| Decision | What we learned |
|----------|----------------|
| Quick mode as default | Most questions don't need 3 sources |
| Deep mode explicit | User or agent decides when depth is needed |
| Save each source separately | Enables re-synthesis without re-running searches |
| Synthesis is the value | Raw findings from 3 sources aren't useful; combined analysis is |
| Start slow sources first | Deep research APIs take minutes; web search takes seconds |
| Attribute everything | Trust requires knowing where information came from |

## Research Agent Integration

The `research-agent` already supports this pattern with quick and deep modes. To add multi-source support:

1. Configure API keys for additional providers in `.env`
2. Update the research-agent to fan out when in deep mode
3. Add a synthesis step that combines results
4. Save to `research/output/` with the naming convention above

## Getting Started

1. Start with quick mode (single web search tool)
2. Add one deep research provider (OpenAI or Perplexity)
3. Run a few deep research queries and evaluate quality
4. Add more sources only if you need broader coverage
5. Build the synthesis step once you have 2+ sources
6. Create command shortcuts (`/research` for quick, `/deep-research` for deep)
