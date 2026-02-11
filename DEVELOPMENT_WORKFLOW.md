# Development Workflow

## The Challenge
OpenClaw forgets context. Claude Web Interface doesn't have access to codebase.
You need continuity across different AI interfaces.

## The Solution: Documentation-Driven Development

### Before Writing Code

1. **Update PROJECT_PLAN.md**
   - What are you building?
   - Why are you building it?
   - How does it fit in the roadmap?

2. **Create/Update Technical Design Doc**
   - If new domain: Create `docs/domains/{domain_name}.md`
   - If new feature: Update `ARCHITECTURE.md`

3. **Create GitHub Issue**
   - Title: Clear, specific feature/bug
   - Body: Link to relevant docs, explain why this matters
   - Labels: `enhancement`, `bug`, `documentation`, etc.
   - Assign to milestone

### When Coding in OpenClaw

**Start of Session (OpenClaw forgets everything):**
```
Me to OpenClaw: "Read openclaw-hub-docs/PROJECT_PLAN.md and 
openclaw-hub-docs/CURRENT_TASK.md. We're working on Issue #5."
```

**During Development:**
- OpenClaw reads current code
- Makes changes
- Commits with good messages
- Updates CURRENT_TASK.md with progress

**End of Session:**
Update `CURRENT_TASK.md`:
```markdown
# Current Task

## What I'm Working On
Issue #5: Implement Kie.ai provider in media domain

## Status
✅ Created `domains/media/providers/kie.py`
✅ Implemented basic API client
⏸️  IN PROGRESS: Error handling for rate limits
❌ TODO: Unit tests
❌ TODO: Add to orchestrator routing

## Next Session Should:
1. Complete error handling in kie.py (lines 45-60)
2. Write unit tests in tests/test_kie_provider.py
3. Update media orchestrator to include Kie option

## Blockers
None

## Notes
- Kie.ai rate limit is 100/hour (documented in provider code)
- Need to decide on retry backoff strategy (exponential vs linear)
- Consider adding cost estimation before calling API

## Context for Next LLM
The Kie.ai integration is mostly done. Focus on making it production-ready
with proper error handling and tests. The orchestrator already has the
interface defined in base.py, just needs to call kie.generate_video().
```

### When Planning in Claude Web (This Interface)

**You can do "free" planning here without token costs:**

1. **Architectural discussions** - Saved in ARCHITECTURE.md
2. **Feature planning** - Create GitHub issues from our discussion
3. **Documentation writing** - Draft in chat, then commit to docs repo
4. **Problem solving** - Work through design challenges, document decisions

**At end of session:**
```
Me: "Create a summary I can commit to PROJECT_PLAN.md"
Claude: [Generates markdown you copy/paste]
```

### Context Switching Protocol

**From Claude Web → OpenClaw:**
1. Commit any plan updates to docs repo
2. Create/update GitHub issue if new task
3. Update CURRENT_TASK.md
4. Tell OpenClaw: "Read docs/CURRENT_TASK.md and Issue #X"

**From OpenClaw → Claude Web:**
1. OpenClaw commits code changes
2. OpenClaw updates CURRENT_TASK.md
3. You read CURRENT_TASK.md in web interface
4. Continue planning/architecture work

### Git Workflow

**Branch Strategy:**
- `main` - Always deployable
- `develop` - Integration branch
- `feature/issue-X-description` - Feature branches

**Commit Messages:**
```
feat: Add Kie.ai provider to media domain (#5)

- Implements BaseProvider interface
- Handles rate limiting
- Adds cost estimation

Related: #5
```

**Pull Requests:**
- Even solo, create PRs to document decisions
- Write description explaining WHY
- Link to issues, reference docs
- Merge to develop, then to main periodically

## File Organization for Context
```
openclaw-hub-docs/
├── PROJECT_PLAN.md           # The master plan (read this first)
├── ARCHITECTURE.md           # Technical design
├── CURRENT_TASK.md          # Where you are RIGHT NOW
├── DEVELOPMENT_WORKFLOW.md  # This file
├── decisions/               # Architecture Decision Records
│   ├── 001-why-python.md
│   ├── 002-single-vs-multi-server.md
│   └── 003-yaml-config.md
├── domains/                 # Per-domain deep dives
│   ├── media.md
│   ├── git.md
│   └── data.md
└── guides/
    ├── adding-provider.md
    ├── adding-domain.md
    └── testing-guide.md
```

## The Rule

**If an LLM (OpenClaw or Claude) needs to know it, it goes in docs.**

Code comments are for HOW.
Documentation is for WHY and WHAT.
