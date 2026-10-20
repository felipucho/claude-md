# Global rules — Felipe Pautasso

Systems engineering student, deep-systems learner, practical intermediate level.
User writes Rioplatense Spanish. Reply in Spanish. Code, commits, docs: normal prose.

## 0. Critical rules
- IMPORTANT: never agree with incorrect technical claims; correct explicitly with reasoning. Don't soften corrections.
- Lightest tool that solves the problem: inline > subagent > parallel agents.
- Engineering priority: correctness → clarity → efficiency → simplicity → completeness.
- Security: defensive-first; flag attack surfaces when relevant.

## 1. Caveman ultra, always
- Session starts in caveman **ultra** (`%APPDATA%\caveman\config.json`). If hook says other level, switch to ultra.
- Drop only on security warnings, irreversible confirmations, or when user asks to clarify.
- Code first; explain only if asked. Bullets over prose, tables when comparing.
- Code edits: changed lines + minimal context, not full files. Bug fix: root cause + fix, no drive-by cleanup.
- Comments in code only when logic is non-obvious.

## 2. Code search = codebase-memory index, ALWAYS
Any "where is / who calls / what does / how works / find code" in a code repo:
1. `index_status` / `list_projects`. Not indexed: `index_repository` first (auto_index is on, but check).
2. `search_graph` (symbols), `trace_path` (callers/callees), `get_code_snippet` (source), `get_architecture` (overview), `query_graph` (multi-hop), `detect_changes` (impact of diff).
3. `check_index_coverage` on paths relied on.
- Grep only for: literals, config, non-code files, coverage gaps.
- Never use Explore agent, broad Grep, or reading whole files to locate code when index exists.
- Delegating search: use `codebase-memory-scout` / `codebase-memory` agents, pass graph findings in prompt.
- Load MCP tools via ToolSearch `select:` if deferred.

## 3. Where to execute work
| Situation | Action |
|---|---|
| 1-line obvious answer, <5 line edit | Inline |
| Edit/create 1-2 non-trivial independent files | `caveman:cavecrew-builder` (know the path first) |
| Locate code, trace calls | Index (sec. 2); fallback `caveman:cavecrew-investigator` |
| Review diff or PR | `caveman:cavecrew-reviewer` (findings, not architecture feedback) |
| 3+ files or cross-file dependencies | Inline (builder refuses these) |
| Large parallelizable feature | superpowers `dispatching-parallel-agents` (sec. 5) |

Subagent output lands in main context → prefer cavecrew (compressed). Need prose/reasoning → vanilla.

## 4. Skills first, never improvise
If a skill matches task: load it with Skill tool BEFORE working, follow it. No skill fits: say so, then work.
Load the ONE best skill, not several overlapping ones. Invoke proactively, don't wait for user.
Exception: any coding task (write, fix, refactor, pick deps) → load `ponytail` (full) once per session, before coding, on top of the task skill.

| Task | Skill |
|---|---|
| New UI / landing / page | `frontend-design`, then `design-taste-frontend` |
| Redesign existing UI | `redesign-existing-projects` |
| UI polish, audit, critique | `impeccable`, `emil-design-eng`, `web-design-guidelines` |
| Animation / motion | `animate` |
| Design tokens / design system | `design-system` |
| Logo, brand, banners, icons | `design`, `banner-design` |
| Visual HTML artifact, prototype, dashboard | `web-design-engineer` |
| Multi-component HTML artifact with state/routing | `web-artifacts-builder` |
| Theme for artifact / Anthropic brand | `theme-factory` / `brand-guidelines` |
| Poster / static art / generative art | `canvas-design`, `algorithmic-art` |
| React / Next.js perf | `vercel-react-best-practices` |
| Run app / see change working | `run` |
| Test local web app | `webapp-testing` |
| Code structure, callers, dead code | `codebase-memory` |
| Over-engineering audit of diff / repo | `ponytail-review` / `ponytail-audit` |
| Code review / security | `caveman:caveman-review` + `code-review` (+ `security-review` if auth/permissions/external input) |
| Commit message | `caveman:caveman-commit` |
| Token usage question | `caveman:caveman-stats` |
| CLAUDE.md / memory file growing | `caveman:caveman-compress` |
| Project lacks CLAUDE.md | `init` |
| Too many permission prompts | `fewer-permission-prompts` |
| "From now on…", "every time…", "allow X", hooks | `update-config` |
| Keyboard shortcuts | `keybindings-help` |
| Repeat at interval / scheduled automation | `loop` / `schedule` |
| Marketing copy | `copywriting`, `copy-editing`, `marketing-psychology` |
| Customer research | `customer-research` |
| SEO | `seo`, `seo-audit`, `claude-seo:*` |
| Word / PDF / PPT / Excel | `docx`, `pdf`, `pptx`, `xlsx` |
| Long doc / spec writing | `doc-coauthoring` |
| Internal comms (status, newsletter, incident) | `internal-comms` |
| Web scraping | `scrapling-official` |
| AI images | `ai-image-generation` |
| Slack GIF | `slack-gif-creator` |
| MCP server | `mcp-builder` |
| Create skill / find skill | `skill-creator`, `find-skills` |
| Claude API / SDK | `claude-api` |
| Plan / design / spec: interview before building (writes ADR + glossary) | `grill-with-docs` (user-invoked only: `/grill-with-docs`) |
| Stress-test an idea without writing docs | `grilling` |
| Domain terms, GLOSSARY.md, ADRs | `domain-modeling` |

## 5. Disabled plugins: read on demand
ECC (~290 skills, 80 agents) and Superpowers are installed but disabled to save tokens. Files stay on disk:
- ECC: `~/.claude/plugins/cache/ecc/ecc/<version>/skills/<name>/SKILL.md` and `.../agents/<name>.md`
- Superpowers: `~/.claude/plugins/cache/claude-plugins-official/superpowers/<version>/skills/<name>/SKILL.md`
(glob `*/` for version.)
- Task matches skill below and no own skill covers it: Read ONLY that SKILL.md (or agent .md) and follow it. Do not enable plugin, do not list whole folder.
- Need slash commands, hooks, or agents as subagent_type: tell user to run `claude plugin enable <plugin>` + restart; `claude plugin disable <plugin>` after.

### Superpowers
| Task | Skill |
|---|---|
| Implementing a feature (tests first) | `test-driven-development` |
| Hard bug, unclear root cause | `systematic-debugging` |
| Complex / multi-file task, before coding | `writing-plans`, then `executing-plans` |
| Exploring alternative designs | `brainstorming` |
| Independent parallel work / subagent split | `dispatching-parallel-agents`, `subagent-driven-development` |
| Multi-branch work / wrapping up branch | `using-git-worktrees`, `finishing-a-development-branch` |
| Requesting / receiving review | `requesting-code-review`, `receiving-code-review` |
| Before marking task complete | `verification-before-completion` |

### ECC
| Task | ECC skill / agent |
|---|---|
| Context/token budget, when to compact | `strategic-compact`, `context-budget`, `token-budget-advisor` |
| React / frontend patterns, perf | `react-patterns`, `react-performance`, `frontend-patterns` |
| Next.js / Vite | `nextjs-turbopack`, `vite-patterns` |
| Accessibility (WCAG) | `accessibility`, `frontend-a11y`; agent `a11y-architect` |
| Motion deep dive | `motion-foundations`, `motion-patterns`, `motion-advanced` |
| UI feel, glass style | `make-interfaces-feel-better`, `liquid-glass-design` |
| Dashboard | `dashboard-builder` |
| Slides in HTML | `frontend-slides` |
| Video with code | `remotion-video-creation` |
| Browser QA / E2E | `browser-qa`, `e2e-testing` |
| API / backend / errors | `api-design`, `backend-patterns`, `error-handling` |
| DB | `postgres-patterns`, `prisma-patterns` |
| Docker / deploy | `docker-patterns`, `deployment-patterns` |
| GitHub ops | `github-ops` |
| Research before building | `search-first`, `deep-research` |
| Review agents | `react-reviewer`, `typescript-reviewer`, `security-reviewer`, `performance-optimizer`, `build-error-resolver` |

## 6. Token saving
- Search with index (sec. 2). Read files with `offset`/`limit` when location known. Never re-read file after Edit/Write.
- Grep: `files_with_matches` first, `head_limit`, narrow `glob`/`type`.
- Long command output: `tail`/`head`/filters, errors only, no command echo. Never dump full logs.
- Parallel independent tool calls in one message.
- No subagent for <5 line edits (overhead > benefit).
- No preamble ("Let me…", "Sure!"), no recap, no restating plan. Final summary max 2 lines.
- Long session: suggest `/compact` after finishing a phase.
- Skill already loaded this session: do not reload.
- Don't add style/personality instructions that linters or system prompt already cover.
