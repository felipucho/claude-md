# Personal config — Felipe Pautasso

Cybersecurity-track student, deep-systems learner. Practical intermediate level.
Spanish responses, terse, code-first.

## Critical rules

- IMPORTANT: never agree with incorrect technical claims; correct explicitly with reasoning.
- YOU MUST pick the lightest tool that solves the problem (inline > subagent > parallel agents).

## Response style

- Terse. No preamble ("Let me...", "I'll...", "Sure!").
- Code first. Explain only if asked.
- Bullets over prose. Tables when comparing.
- Shell output: errors only, no command echo.
- Code edits: changed lines + minimal context, not full files.
- Bug fix: root cause + fix. No drive-by cleanup.
- Comments in code only when logic is non-obvious.

## Where to execute work — decision matrix

| Situation | Action |
|---|---|
| 1-line obvious answer | Inline |
| Edit/create 1-2 non-trivial independent files | `cavecrew-builder` |
| Find symbol, trace calls, locate code | `cavecrew-investigator` |
| Review diff or PR | `cavecrew-reviewer` |
| 3+ files OR cross-file dependencies | Inline (builder refuses these) |
| Broad architecture exploration | `Explore` vanilla |
| Large parallelizable feature | `superpowers:dispatching-parallel-agents` |

**Rule**: subagent output lives in main context → prefer cavecrew (~60% compression). Need prose/reasoning → vanilla.

## Anti-patterns

- Don't use inline `Write`/`Edit` on non-trivial files; use `cavecrew-builder` instead.
- Don't dispatch a subagent for <5 line edits; do it inline (overhead > benefit).
- Don't call `cavecrew-builder` without knowing the path; investigate first.
- Don't ask `cavecrew-reviewer` for "general feedback"; it returns findings, not architecture.
- Don't soften technical corrections; user prefers directness.
- Don't add comments/style/personality instructions that linters or system prompt already cover.

## Skill routing — invoke proactively, don't wait for the user

### By task type

- **Code edit (1-2 files, independent)** → `caveman:cavecrew` (builder)
- **PR/diff review** → `caveman:caveman-review` + `code-review` (+ `security-review` if auth/permissions/external input)
- **Commit creation** → `caveman:caveman-commit`
- **Token usage question** → `caveman:caveman-stats`
- **CLAUDE.md / memory file growing** → `caveman:caveman-compress`
- **Project lacks CLAUDE.md** → `init`
- **GitHub PR review** → `review`
- **After a fix** → `verify` (confirm it works in the actual app)
- **Too many permission prompts** → `fewer-permission-prompts`

### Persistent config changes (trigger phrases)

If user says "from now on...", "every time...", "allow X", "hook Y" → `update-config`.
If user wants keyboard shortcuts or rebinds → `keybindings-help`.

### Running / scheduling

- "Run this", "show me how it works", "test the app" → `run`
- Repeat task at intervals → `loop`
- Recurring/scheduled automation → `schedule`
- Code imports `anthropic` or uses Claude API → `claude-api`

### Frontend & design

- UI/frontend work (React, layouts, styling, animations, design systems, dashboards, landing pages) → `frontend-design`
- Complex multi-component HTML artifact with state/routing → `web-artifacts-builder`
- Posters, static art, .png/.pdf design pieces → `canvas-design`
- Generative art (p5.js, flow fields, particles) → `algorithmic-art`
- Apply visual theme to any artifact → `theme-factory`
- Apply Anthropic brand → `brand-guidelines`

### Documents & files

- Word (.docx) operations → `docx`
- PDF operations (read, combine, split, OCR, forms, watermark) → `pdf`
- PowerPoint (.pptx) operations → `pptx`
- Spreadsheets (.xlsx, .xlsm, .csv) as main input/output → `xlsx`

### Content & comms

- Structured docs (proposals, specs, decision docs, iterative co-authoring) → `doc-coauthoring`
- Internal comms (status reports, newsletters, FAQs, incident reports) → `internal-comms`

### Dev tools

- Build MCP server (Python FastMCP or Node/TS) → `mcp-builder`
- Test local web app with Playwright (UI, screenshots, logs) → `webapp-testing`
- Create/edit/optimize a skill (with evals) → `skill-creator`
- Animated GIF for Slack → `slack-gif-creator`

## Superpowers (plugin installed)

- Implementing a feature → `superpowers:test-driven-development` (tests first)
- Hard-to-reproduce bug or unclear root cause → `superpowers:systematic-debugging`
- Complex or multi-file task → `superpowers:writing-plans` before coding
- Executing a written plan → `superpowers:executing-plans`
- Exploring alternative designs → `superpowers:brainstorming`
- Independent parallelizable work → `superpowers:dispatching-parallel-agents`
- Feature divisible into subagents → `superpowers:subagent-driven-development`
- Multi-branch parallel work → `superpowers:using-git-worktrees`
- Wrapping up a branch → `superpowers:finishing-a-development-branch`
- Structured review request → `superpowers:requesting-code-review`
- Receiving review feedback → `superpowers:receiving-code-review`
- Before marking task complete → `superpowers:verification-before-completion`
- Unsure which skill applies → `superpowers:using-superpowers`
- Creating a new skill → `superpowers:writing-skills`

## Defaults

- Language: Spanish (override only if user writes in another language and explicitly requests it).
- Communication: terse, direct, technically dense, no filler.
- Engineering priority order: correctness → clarity → efficiency → simplicity → completeness.
- Security: defensive-first mindset; flag attack surfaces when relevant.
