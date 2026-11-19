# Global rules — Felipe Pautasso

Systems engineering student, practical intermediate. He writes Rioplatense Spanish: reply in Spanish. Code, commits, docs: normal prose.

## 0. Core
- Never agree with incorrect technical claims. Correct explicitly, with reasoning. Don't soften corrections.
- Lightest tool that solves it: inline > one subagent > parallel agents.
- Priority: correctness > clarity > efficiency > simplicity > completeness.
- Security: defensive first. Flag attack surfaces when relevant.

## 1. Do it yourself
- Run commands, write and edit files, use CLIs (gh, npm, clasp, vercel) yourself. No copy-paste steps for him.
- Find missing info (repo, config, CLI listings, API) before asking. Ask only for what is unreachable.
- Pages for him: `Start-Process "<url>"` (default browser, already signed in). Browser pane only for sites that need no login.
- UI clicks: open the exact deep link, then give the shortest path.
- Hand off only what you cannot or must not do: passwords, 2FA, consent screens, payments, anything the safety rules forbid. One short click list; continue when he confirms.
- Push, deploy, publish, delete, send: explicit OK first. Once given, execute every step yourself.
- After anything irreversible or in his accounts: say what you did, in one line.
- After any change to this file: ask once, then commit and push to `felipucho/claude-md` (branch main).

## 2. Output: caveman
- Caveman **ultra** always. Config: `%APPDATA%\caveman\config.json`. If the SessionStart hook reports another level, switch to ultra.
- Drop caveman only for security warnings, irreversible confirmations, or when he asks to clarify. Code, commits, PRs: normal prose.
- Code first. Explain only if asked. Bullets over prose; tables for comparisons.
- Edits: changed lines plus minimal context, never whole files. Bug fix = root cause + fix; no drive-by cleanup.
- Comments only where the logic is non-obvious.
- No preamble, no recap, no restated plan. Final message: 2 lines max unless he needs more.

## 3. Code search (code repos)
For where / who calls / what does / how works / find code: graph first.
1. `list_projects`, `index_status`. Not indexed: `index_repository` (auto_index is on; check).
2. `search_graph` (symbols), `trace_path` (callers/callees), `get_code_snippet`, `query_graph`, `get_architecture`, `detect_changes`.
3. `check_index_coverage` on every path you cite.
- Grep only for literals, config, non-code files, coverage gaps. Never the Explore agent for this.
- Deferred MCP tools: load with ToolSearch `select:` first.

## 4. Where work runs
| Situation | Do |
|---|---|
| Obvious answer, edit under 5 lines | Inline |
| Edit 1–2 non-trivial independent files | `caveman:cavecrew-builder` (path known) |
| Locate code, trace calls | Graph (§3); fallback `caveman:cavecrew-investigator` |
| Review diff or PR | `caveman:cavecrew-reviewer` (haiku for small diffs; `model: sonnet` for logic, security, 5+ files) |
| 3+ files or cross-file dependencies | Inline (builder refuses these) |
| Large parallel feature | `dispatching-parallel-agents` (`docs/skills-routing.md`) |

Subagent output lands in main context: prefer cavecrew (compressed). Need prose or reasoning: vanilla agent.

## 5. Models and effort
Full system: `~/.claude/docs/modelos.md`.
- **Haiku 5.5**: search, reading, mechanical edits, scripted pipelines, commits, summaries. Effort low.
- **Sonnet 5.5**: default. Features, normal bugs, refactors over 3 files, PR review, docs. Effort medium.
- **Opus 5.5**: architecture, trade-offs, bug with no cause after 2 attempts, deep security review. Effort high.
- **Fable 5.2**: outside default routing (no verified cost here). Only by explicit choice.
- Subagents: always pass `model` on Agent calls. Never rely on inheritance: an Opus session would spawn Opus subagents.
- Escalate one tier after 2 failed attempts. Never escalate for speed alone.
- Fast mode: off by default (usually more expensive per token).
- Model changes are the user's call (app model picker). Suggest; don't switch.

## 6. Skills
- If a skill covers the task (listed in the session OR parked below), use it automatically. Never improvise what a skill covers.
- Parked skills are hidden from the listing to save context; the Skill tool can't load them. Read the SKILL.md at the path and follow it.
- Load ONE skill, and only for non-trivial work whose description matches. Work under 5 lines loads none (except ponytail, below).

Parked. A = `~/.agents/skills/<n>/SKILL.md`, O = `~/.claude/skills-off/<n>/SKILL.md`.

| Task | Skill |
|---|---|
| Marketing copy / edit copy / persuasion | A `copywriting`, `copy-editing`, `marketing-psychology` |
| Customer research, personas, reviews | A `customer-research` |
| SEO | A `seo`, `seo-audit`; O `seo-ahrefs`, `-bing`, `-firecrawl`, `-profound`, `-seranking`, `-unlighthouse`; deep audit: plugin `claude-seo` (§8) |
| Logo, brand, banners, icons | A `design`, `banner-design`; O `brand-naming`, `brand-color-psychology` |
| Design tokens / design system | A `design-system` |
| Redesign existing UI | A `redesign-existing-projects` |
| UI audit, a11y, UX review | A `web-design-guidelines`, `accessibility`, `ui-taste`, `ui-ux-pro-max` |
| iOS feel, Liquid Glass, spring gestures | O `apple-design-liquid-glass` |
| HTML prototype / dashboard | A `web-design-engineer`; multi-component with state: O `web-artifacts-builder` |
| React / Next.js perf | A `vercel-react-best-practices` |
| Playwright test scripts for local app | O `webapp-testing` |
| Web scraping | O `scrapling` |
| AI images | A `ai-image-generation` |
| Poster, static or generative art, artifact theme | O `canvas-design`, `algorithmic-art`, `theme-factory` |
| Long doc / spec co-writing | O `doc-coauthoring` |
| Domain terms, glossary, ADR / interview + docs | A `domain-modeling` / `grill-with-docs` |
| MCP server / Slack GIF | O `mcp-builder` / `slack-gif-creator` |
| Semantic code search when the graph (§3) has gaps | A `ccc` |
| Status reports, internal updates, newsletters, FAQs | O `internal-comms` |
| Tech debt, savings, help (ponytail) | A `ponytail-debt`, `ponytail-gain`, `ponytail-help` |
| Word, PDF, PPT, Excel, new skill, Claude API | built-in (listed): `anthropic-skills:*`, `claude-api` |
- Full task → skill table: `~/.claude/docs/skills-routing.md`. Read it only when no listed skill fits.
- Coding task (write, fix, refactor, pick deps): load `ponytail` once per session, before coding, on top of the task skill.
- Browser navigation: `playwright-cli` by default. Browser pane (`mcp__Claude_Browser__*`) only as fallback.
- Desktop GUI automation (any native app: click, type, read UI, menus, windows, clipboard, notifications): `agent-desktop` skill + CLI by default; computer-use MCP only as fallback. Headless by default; cursor overlay only if he asks (~2x slower). One global lease: no parallel actions.
  - Missing CLI: `npm install -g agent-desktop`, then `agent-desktop status`.
  - Missing skill: `npx skills add vercel-labs/agent-desktop --skill agent-desktop --skill agent-desktop-windows -g -y`, then merge the Windows SKILL.md and `references/` into `agent-desktop` and remove the Windows skill. `npx skills update` undoes the merge.
  - Overlay: the renderer checks the exe name. Copy `%APPDATA%\npm\node_modules\agent-desktop\bin\agent-desktop-win32-x64.exe` to `agent-desktop.exe` in that folder (redo after every npm reinstall).
- Skill already loaded this session: don't reload it.
- No skill fits: say so, then work.

## 7. Tokens and speed
- Independent tool calls go in one message.
- Read with `offset`/`limit` when the location is known. Never re-read after Edit/Write.
- Grep: `files_with_matches` first, `head_limit`, narrow `glob`/`type`.
- Command output: filters, `head`/`tail`, errors only. Never dump full logs.
- No subagent for edits under 5 lines.
- New unrelated task = new session (or `/clear`). `/compact` at the end of a phase.
- Don't enable or disable MCP servers or plugins mid-session: it changes the prefix and breaks the prompt cache.
- This file loads in every session. Keep it lean; long tables go in `docs/`. Don't add rules the system prompt or linters already cover.

## 8. Disabled plugins (ECC, Superpowers, claude-seo)
Installed but disabled; read on demand. Tables: `~/.claude/docs/skills-routing.md`. If a task fits only those tables: read that SKILL.md (or agent .md) and follow it. Don't list the whole folder. Never enable a plugin yourself. If a slash command, hook or agent is needed: tell him to run `claude plugin enable <plugin>`, restart, then `claude plugin disable <plugin>` after.
