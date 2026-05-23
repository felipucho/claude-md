# CLAUDE.md Research Summary (2025–2026)

This README summarizes and explains the research, evidence, and rationale behind the CLAUDE.md best-practices document uploaded to this repository. It condenses key recommendations, the supporting evidence, and why each recommendation matters in practice.

## TL;DR
- Keep CLAUDE.md short (target ≤200 lines). Evidence: Anthropic memory docs show large files consume model instruction/attention budget. Rationale: smaller advisory context increases adherence and reduces signal dilution.
- Use emphatic markers (IMPORTANT / YOU MUST) for only 1–2 load-bearing rules. Evidence: Anthropic engineering guidance and community studies. Rationale: emphasis affects sampling; overuse nullifies the signal.
- Move deterministic checks to hooks/linters, not CLAUDE.md. Evidence: community consensus (HumanLayer, Builder.io). Rationale: deterministic tools ensure reproducible enforcement without consuming LLM context.
- Avoid @-imports of large docs. Evidence: official docs state imports expand at launch (recursively). Rationale: @-imports inflate context at session start; prefer bare paths and Read tool on demand.
- Use HTML comments (<!-- -->) for human notes. Evidence: comments are stripped before injection in runtime. Rationale: keep on-disk annotations without token cost.
- Prefer path-scoped rules (.claude/rules/*.md with frontmatter) and skills for lazy-loading. Evidence: Anthropic docs on path-scoped rule triggering. Rationale: reduces always-loaded context and keeps per-subdir policies contextual.

## Key recommendations — evidence & rationale
1) Keep file size small (≤200 lines)
   - Evidence: Anthropic docs: "target under 200 lines per CLAUDE.md file" (memory docs). Community benchmarks show adherence drops with larger advisory contexts.
   - Why: CLAUDE.md is injected at session start; long advisory text consumes the model's attention and reduces compliance for important rules.

2) Emphasize only the critical invariants
   - Evidence: Anthropic engineering posts and community tests about emphasis markers increasing adherence when used sparingly.
   - Why: Emphasis affects sampling/priority. If everything is marked IMPORTANT, no rule is prioritized.

3) Deterministic rules → hooks/linters
   - Evidence: Community best-practices (HumanLayer, Builder.io). Linters and hooks are deterministic and cheap.
   - Why: LLMs are probabilistic; do not use them for enforcement of invariants that must never be skipped.

4) Avoid eager expansion (@imports)
   - Evidence: Official memory docs note imports expand at launch recursively up to a depth limit.
   - Why: Eager expansion causes large, hidden context increases and surprises cost budget early in sessions.

5) Use comments for annotations
   - Evidence: Block-level HTML comments are stripped before injection (official behavior).
   - Why: Preserve rich inline rationale for humans without token cost when the CLAUDE.md is injected.

6) Use path-scoped rules and skills for targeted behavior
   - Evidence: Anthropic docs: path-scoped rules trigger only when files matching the pattern are read.
   - Why: Keeps global CLAUDE.md small; loads extra rules only when relevant.

## Templates included
- `CLAUDE.md` — example clean project CLAUDE.md (committable, target ≤200 lines).
- `CLAUDE.annotated.md` — annotated reference with HTML comments for maintainers.
- `CLAUDE.minimal.md` — minimal template (<500 tokens) for quick projects.
- `CLAUDE.monorepo.md` — monorepo guidance and cross-package rules.

(These templates live in this repo alongside the uploaded CLAUDE.md.)

## Operational rules & workflow notes
- Run `/clear` between unrelated tasks to reduce cross-task noise.
- Run `/compact <instructions>` proactively during long sessions to preserve chosen decisions.
- Use `/context` to verify what files are currently loaded and to debug missing rules.
- Prefer subagents/skills for heavy research or repo-wide reading tasks; they isolate context.
- Prompt caching reduces billing cost but does not eliminate context-window occupation.

## Caveats & limitations
- Many numeric thresholds are heuristics from community benchmarks (e.g., 150–200 instructions). Treat them as conservative heuristics, not absolute limits.
- Some features/behaviors are version-dependent (this guide is calibrated to Claude Code v2.1.x — Opus 4.7 / Sonnet 4.6 / Haiku 4.5 at the time of research).
- Evidence mixes official documentation, community experiments, proxy logs, and published benchmarks. Each source is noted in references below.

## References (selected)
- Anthropic — Memory & CLAUDE.md docs: https://code.claude.com/docs/en/memory
- Anthropic engineering posts (emphasis guidance)
- HumanLayer, Builder.io — community best-practices
- Distyl AI — "How Many Instructions Can LLMs Follow at Once?" (arXiv)
- Scott Spence — skill activation measurements (sandboxed trials)
- Proxy logs & issue reproductions cited in the original research (see uploaded CLAUDE.md for full bibliography)

---

If you want a different tone, more/less detail, or to include inline citations (URLs per-finding), say which sections to expand. This README is intentionally concise; the uploaded CLAUDE.md and CLAUDE.annotated.md contain the full research and per-item evidence.
