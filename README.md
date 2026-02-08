# Codebase Explorer

A Claude Code skill that produces navigable maps of how code actually works — not summaries of file contents.

Point it at any repo and say "explore this codebase." It will classify the architecture, trace data flows, identify fragility points, and produce a structured orientation map you can act on.

## Install

```
npx skills add quicksilversurfer/codebase-explorer
```

## Usage

After installing, trigger it in Claude Code with natural language:

- "Explore this repo"
- "Help me understand this codebase"
- "I inherited this code — onboard me"
- "What does this project do?"
- "Audit this system's architecture"

## What You Get

The skill runs a structured 4-phase exploration:

**Phase 0 — Archetype Detection**: Classifies the repo (frontend, backend API, data pipeline, serverless, library, IaC, CLI, ML system) and selects the right exploration strategy.

**Phase 1 — Orientation**: Maps I/O boundaries, substrate constraints, implicit contracts, and development activity.

**Phase 2 — Vertical Slices**: Traces data from input to output using an archetype-specific strategy — visible values for frontends, record-through-stages for pipelines, request lifecycle for APIs.

**Phase 3 — Smell Detection**: Scans for architectural smells matched to the codebase type, including pipeline-specific patterns like drifting duplicates, implicit inter-stage contracts, and shared utility single points of failure.

**Phase 4 — Risk Profile**: Synthesizes findings into specific, actionable risk statements and explicitly marks what wasn't explored.

## Supported Archetypes

| Archetype | Exploration Strategy | Primary Output |
|-----------|---------------------|----------------|
| Frontend Application | Trace visible value backward | Render-site trace |
| Data / Serverless Pipeline | Trace record through stages | Data lineage map |
| Backend API | Trace request lifecycle | Request lifecycle trace |
| Library / SDK | Trace public API inward | Abstraction layer map |
| Infrastructure as Code | Trace resource dependencies | Blast radius map |
| CLI Tool | Trace command execution | Command execution trace |
| ML System | Trace training data flow | Data + model lineage map |

## Who This Is For

| Audience | What they get |
|----------|--------------|
| Developer inheriting code | Where to start, what's safe to change |
| Tech lead / architect | Risk map, coupling analysis, refactoring targets |
| Engineering manager | Change cost estimates, coordination needs |
| Product manager | What's hard vs. easy to change, in business terms |
| Security auditor | Attack surface, data flow, trust boundaries |
| New hire | Navigable mental model of the system |
| Consultant / contractor | Fast orientation before making changes |

## File Structure

```
SKILL.md                              # Main skill definition (353 lines)
references/
  smell-catalog.md                    # 13 architectural smells with detection guides
  documentation-templates.md          # 7 output templates (including stakeholder briefs)
  advanced-techniques.md              # Fan-in handling, dead code detection, verification
```

## How It Works

The skill is pure Markdown — no code, no dependencies. When triggered, Claude Code loads `SKILL.md` as context, which guides the exploration methodology. Reference files are pulled in on demand when the main skill references them.

The skill improves Claude's output by providing:
- A **repeatable protocol** instead of ad-hoc exploration
- **Archetype-specific strategies** that prevent wasted effort
- **Named concepts** (shared fate groups, implicit contracts, data lineage maps) that produce more actionable output
- **Audience adaptation** that adjusts depth and vocabulary to the reader's role

## License

MIT
