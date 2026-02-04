---
name: advanced-research
description: "Interactive deep research with adaptive query refinement, clarifying questions, and iterative exploration"
category: command
complexity: advanced
mcp-servers: [sequential]
personas: [deep-research-agent]
---

# /sc:advanced-research - Interactive Deep Research

> **Context Framework Note**: This command activates interactive research capabilities with query refinement via AskUserQuestion, multi-hop search, and iterative exploration rounds.

## Triggers
- Complex research questions requiring clarification before searching
- Topics where scope, depth, or focus is ambiguous
- Multi-angle research benefiting from user-guided direction
- Comparative analysis, market research, technical deep-dives

## Context Trigger Pattern
```
/sc:advanced-research "[query]" [--depth quick|standard|deep|exhaustive] [--save]
```

## Behavioral Flow

### Phase 0: Query Analysis (Silent - DO NOT output to user)

Analyze the user's query internally to identify:
- **Intent**: factual | comparative | exploratory | tutorial | market analysis
- **Ambiguity**: which parts are vague or could mean multiple things
- **Scope**: too broad, too narrow, or well-defined
- **Knowledge gaps**: what context would improve search quality

### Phase 1: Clarification via AskUserQuestion

Generate 1-4 targeted questions dynamically based on Phase 0 analysis. Each question must:
- Address a specific ambiguity found in Phase 0
- Offer options representing meaningfully different search directions
- Include descriptions explaining how each choice affects the research
- Never ask what the query already answers

**Question dimensions** (use only when relevant to the query):
| Dimension | When to Ask | Example |
|-----------|-------------|---------|
| Focus | Query covers multiple aspects | "Technical deep-dive vs. business comparison?" |
| Scope | Breadth is unclear | "Specific tool vs. market overview?" |
| Recency | Time sensitivity matters | "Latest 2025-2026 vs. historical?" |
| Depth | Detail level unclear | "Overview vs. implementation details?" |
| Audience | Output format depends on reader | "Beginner guide vs. expert reference?" |
| Constraints | Hidden requirements possible | "Open-source only? Specific language?" |

### Phase 2: Search Planning (TodoWrite)

Create a structured search plan:
1. Decompose refined query into 2-5 sub-questions
2. Identify parallel vs. sequential dependencies
3. Create TodoWrite task hierarchy
4. Assign search strategy per sub-question:
   - `WebSearch`: broad information gathering
   - `WebFetch`: extract details from specific URLs

### Phase 3: Execute Searches

- **Parallel-first**: Batch all independent WebSearch calls in a single message
- **Evidence tracking**: Note source URLs and key findings
- **Gap detection**: Identify what searches didn't answer
- **Confidence assessment**: Rate source reliability

### Phase 4: Follow-up Exploration (Iterative)

After initial results, present findings summary and ask via AskUserQuestion:

```
"Initial research complete. How to proceed?"
  - "Go deeper on [specific finding]" → focused follow-up
  - "Explore related angle" → broaden scope
  - "Done, generate report" → proceed to Phase 5
```

Repeat Phase 3-4 until user selects "Done" or equivalent.

### Phase 5: Report Generation

**Always do both**:

1. **Save report** to `claudedocs/research_[topic]_[YYYYMMDD].md`:
```markdown
# Research Report: [Topic]
> Generated: [date] | Depth: [level] | Search rounds: [count]

## Executive Summary
[2-3 sentence key findings]

## Key Findings
### [Finding 1]
[Details with citations]

## Analysis
[Cross-source synthesis, patterns, contradictions]

## Gaps & Limitations
[Unverified claims, conflicting info, areas needing more research]

## Sources
1. [Title](URL) - [what was found]
```

2. **Output conversation summary**: Concise overview of findings directly in chat

## Key Patterns

### Parallel Execution
- Batch independent WebSearch calls
- Use multiple parallel tool calls in single message
- Sequential only for dependent searches (e.g., WebFetch after WebSearch finds URL)

### Evidence Management
- Track all source URLs
- Note confidence level per finding
- Flag contradictions between sources
- Distinguish fact from opinion

### Adaptive Depth
| Level | Search Rounds | Sub-questions | Hop Depth |
|-------|---------------|---------------|-----------|
| Quick | 1 | 2-3 | 1 |
| Standard | 1-2 | 3-4 | 2 |
| Deep | 2-3 | 4-5 | 3 |
| Exhaustive | 3+ | 5+ | 4+ |

## MCP Integration
- **Sequential**: Complex reasoning during synthesis and analysis phases
- **Context7**: Library/framework official documentation (complements WebSearch)
- **Serena**: Persist research context across sessions via memory

## Tool Coordination
- **WebSearch**: Primary search tool for all queries
- **WebFetch**: Extract detailed content from URLs found via WebSearch
- **AskUserQuestion**: Pre-search clarification and post-search follow-up
- **TodoWrite**: Track search progress and sub-task completion
- **Write**: Save research reports to claudedocs/
- **Read/Glob/Grep**: Search local codebase when research relates to project code

## Examples
```
/sc:advanced-research "best state management for React 2025"
# → Asks: focus (comparison vs tutorial), scope (Redux vs all options), constraints (SSR needed?)
# → Searches based on answers
# → Offers follow-up: "Zustand came up frequently. Deep-dive on Zustand vs Jotai?"

/sc:advanced-research "competitive analysis of AI coding assistants" --depth deep
# → Asks: which tools, what dimensions (features/pricing/performance)
# → Multiple search rounds with user-guided pivots
# → Comprehensive comparison report

/sc:advanced-research "security best practices for Next.js API routes"
# → Asks: auth method, deployment target, specific threat model
# → Targeted security-focused search
# → Actionable checklist output
```

## Boundaries

**Will**:
- Ask clarifying questions before every search
- Support iterative exploration with user guidance
- Provide source citations for all claims
- Save structured reports to claudedocs/

**Will NOT**:
- Implement code based on research findings
- Make architectural decisions from research
- Skip clarification for ambiguous queries
- Access restricted or paywalled content

**Next Step**: After research completes, use `/sc:design` for architecture or `/sc:implement` for coding based on findings.
