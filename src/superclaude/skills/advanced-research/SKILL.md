---
name: Advanced Research
description: Interactive deep research with adaptive query refinement. Analyzes user queries, asks clarifying questions via AskUserQuestion before searching, supports iterative exploration with follow-up rounds, and produces structured research reports.
allowed-tools: WebSearch, WebFetch, Read, Write, Glob, Grep, AskUserQuestion, TodoWrite
---

# Advanced Research Skill

## Purpose

Deep web research with **interactive query refinement**. Unlike basic search, this skill:
1. Analyzes the query to identify ambiguity and missing context
2. Asks targeted clarifying questions via `AskUserQuestion` before searching
3. Executes structured, parallel searches based on refined understanding
4. Offers iterative follow-up exploration rounds
5. Saves reports to `claudedocs/` and outputs a summary in conversation

## When to Use

- Complex research questions requiring multiple search angles
- Topics where scope, depth, or focus needs clarification
- Multi-hop research that benefits from user guidance between rounds
- Any research where getting the question right matters more than searching fast

## Behavioral Flow

### Phase 0: Query Analysis (Silent)

Analyze the user's query internally:
- **Intent Classification**: What type of answer is needed? (factual, comparative, exploratory, tutorial, market analysis)
- **Ambiguity Detection**: Which parts of the query are vague or could mean multiple things?
- **Scope Assessment**: Is the scope too broad, too narrow, or unclear?
- **Knowledge Gaps**: What additional context would dramatically improve search quality?

Do NOT output this analysis to the user. Use it to generate targeted questions.

### Phase 1: Clarification (AskUserQuestion)

Based on Phase 0 analysis, ask 1-4 targeted questions using `AskUserQuestion`. Questions should be **dynamic and query-specific**, not generic templates.

**Question Generation Principles**:
- Each question must address a specific ambiguity found in Phase 0
- Options should represent meaningfully different search directions
- Include a description that explains how each choice affects the research
- Never ask obvious questions the query already answers

**Common Question Dimensions** (use only when relevant):
- **Focus**: Which aspect matters most? (e.g., technical implementation vs. business strategy)
- **Scope**: How broad or narrow? (e.g., specific framework vs. general comparison)
- **Recency**: Time sensitivity? (e.g., latest 2024-2025 vs. historical overview)
- **Depth**: Surface overview vs. deep technical detail?
- **Audience**: For whom? (e.g., beginner tutorial vs. expert reference)
- **Constraints**: Any specific requirements? (e.g., open-source only, specific language)

**Example**:
```
Query: "AI coding assistants"

Phase 0 analysis (silent):
  - Intent unclear: comparison? how-to? market overview?
  - Scope too broad: which assistants? what aspect?

AskUserQuestion:
  Q1: "What aspect of AI coding assistants interests you?"
      - "Competitive comparison" → compare features, pricing, capabilities
      - "Technical architecture" → how they work internally (RAG, fine-tuning, etc.)
      - "Best practices for usage" → tips and workflows for productivity

  Q2: "Any specific tools to focus on?"
      - "GitHub Copilot vs Cursor vs Claude Code" → direct head-to-head
      - "All major players" → comprehensive market overview
      - "Open-source alternatives" → self-hosted, privacy-focused options
```

### Phase 2: Search Planning

Based on clarified requirements, create a search plan:
1. Decompose the refined query into 2-5 search sub-questions
2. Identify which sub-questions can run in parallel
3. Create TodoWrite tasks for tracking
4. Determine search strategy per sub-question:
   - `WebSearch` for broad information gathering
   - `WebFetch` for extracting details from specific URLs found in search results

### Phase 3: Execute Searches

- **Parallel-first**: Batch all independent WebSearch calls
- **Evidence tracking**: Note source URLs and key findings per search
- **Gap detection**: Identify what the searches didn't answer
- **Confidence scoring**: Assess reliability of each source

### Phase 4: Follow-up Exploration (Iterative)

After initial results, ask the user via `AskUserQuestion`:

```
Q: "Initial research complete. Want to explore further?"
   - "Go deeper on [specific finding]" → focused follow-up searches
   - "Explore a related angle" → broaden the research
   - "Done, generate report" → proceed to output
```

Repeat Phase 3-4 as needed until user is satisfied.

### Phase 5: Report Generation

1. **Save to file**: `claudedocs/research_[topic]_[YYYYMMDD].md`
2. **Output summary**: Concise findings in conversation

**Report Structure**:
```markdown
# Research Report: [Topic]
> Generated: [date] | Depth: [quick/standard/deep] | Queries: [count]

## Executive Summary
[2-3 sentence overview of key findings]

## Key Findings
### [Finding 1 Title]
[Details with source citations]

### [Finding 2 Title]
[Details with source citations]

## Analysis
[Cross-source synthesis, patterns, contradictions]

## Gaps & Limitations
[What couldn't be verified, conflicting information, areas needing more research]

## Sources
1. [Title](URL) - [brief note on what was found]
2. [Title](URL) - [brief note on what was found]
```

## Integration Notes

### With Other Commands
- Use `/sc:design` after research to architect solutions based on findings
- Use `/sc:implement` to act on research recommendations
- Use `/sc:analyze` to evaluate research findings against existing codebase

### MCP Server Compatibility
- **Sequential MCP**: Can be used for complex multi-step reasoning during synthesis
- **Context7 MCP**: Complements WebSearch for library/framework-specific documentation
- **Serena MCP**: Can persist research context across sessions via memory

## Boundaries

**Will**:
- Ask clarifying questions before searching
- Provide source citations for all claims
- Support iterative exploration rounds
- Save structured reports

**Will NOT**:
- Implement code based on research findings
- Make architectural decisions
- Access restricted/paywalled content
- Skip clarification for ambiguous queries
