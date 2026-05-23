# CLAUDE.md Research & Evidence (2025–2026)

This README now contains an expanded, evidence-backed summary of the CLAUDE.md research you provided. It reproduces the findings, citations, and technical evidence verbatim where relevant so the repository holds the full provenance.

---

## Executive summary (short)
- Keep CLAUDE.md short (target ≤200 lines). Official Anthropic guidance and community experiments show large advisory files reduce adherence and consume attention budget.
- Use emphasis markers (IMPORTANT / YOU MUST) only for 1–2 load-bearing rules.
- Move deterministic enforcement (lint, format, path-blocking) to hooks/tools that run deterministically, not to CLAUDE.md.
- Avoid `@`-imports for long docs — imports expand at launch (recursive up to depth 5) and inflate session context.
- Use HTML comments for human annotations (they are stripped before injection).
- Use path-scoped rules and skills for lazy-loading behaviour.

---

## Full evidence and citations (expanded)
All the following items are taken from the research text you provided and the referenced sources. Quoted fragments are included where they were explicit.

1) Official Anthropic documentation and hard limits
- "target under 200 lines per CLAUDE.md file" — Anthropic memory docs (code.claude.com/docs/en/memory). This is cited repeatedly as the operational target used in the research.
- MEMORY.md hard limit: "The first 200 lines of MEMORY.md, or the first 25KB, whichever comes first, are loaded at the start of every conversation" — Anthropic docs (memory). This is the technical mechanism explaining why short files matter.
- Imported files expansion: "Imported files are expanded and loaded into context at launch" — official docs; imports expand recursively (depth limit ~5). Impact: `@docs/foo.md` will cause eager expansion.
- Block-level HTML comments are stripped before injection: official docs (code.claude.com/docs/en/memory). This validates using <!-- ... --> for human notes with zero token cost at runtime.

2) Evidence from community research, experiments, and proxy logs
- Jannes Klaas proxy reproduction (2025-07-20) — proxy logs showing the wrapper text: the CLAUDE.md content is wrapped with <system-reminder> and prefaced by "IMPORTANT: this context may or may not be relevant to your tasks. You should not respond to this context unless it is highly relevant to your task". Reported in Jannes Klaas blog and reproduced in GitHub issue anthropics/claude-code#18560. (User-supplied evidence: proxy log verbatim quoted in research.)
- Distyl AI (Jaroslawicz, Whiting, Shah, Maamari) — arXiv: "How Many Instructions Can LLMs Follow at Once?", arXiv:2507.11538 (2025-07-15). Quoted result: "even the best frontier models only achieve 68% accuracy at the max density of 500 instructions". This is used to justify conservative heuristics (150–200 instructions) as practical budget.
- Scott Spence (2026-02-08) — skill activation sandboxed trials: reported numbers like Haiku 4.5 baseline ~0%; Sonnet 4.5 baseline ~55%; with forced-eval hook 84–100%. Shows skill/instruction adherence is probabilistic and model-dependent.
- HumanLayer, Builder.io, DataCamp community posts — strong consensus to avoid putting linters/style rules in CLAUDE.md; use deterministic tooling.
- MorphLLM / Steve Kinney conflicting auto-compact thresholds — Morph reports auto-compact triggers at 64–75% capacity; Steve Kinney reports ~95% trigger. Anthropic official docs do not fix a number; this is an unresolved operational caveat in the research.

3) Specific technical statements reproduced from the research
- "CLAUDE.md is advisory, not enforcement. The content is injected as a user message wrapped in <system-reminder> with the phrase: 'IMPORTANT: this context may or may not be relevant to your tasks...'" — proxy log reproduction and Claude reproduction in issue #18560.
- "@path/file.md NO ahorra contexto. The doc official is explicit: 'Imported files are expanded and loaded into context at launch'" — official doc quote.
- "Project-root CLAUDE.md survives compaction: after /compact, Claude re-reads it from disk and re-injects it" — asserted by official docs and community tests (root file persists across /compact; nested files may not).
- "Path-scoped rules in .claude/rules/*.md with YAML frontmatter trigger only when files matching the pattern are read" — official docs: this is the recommended lazy-load mechanism for textual rules.

4) Anti-patterns (detailed, with cause)
- Stuffing commands/styles into one giant file: cause — wrapper filters file more aggressively when lots of irrelevant lines exist (empirical community observation). Source: HumanLayer blog (2025-11-25).
- Using `@docs/...` to include large documentation: cause — recursive expansion at launch inflates context silently. Source: official memory docs.
- Putting code style rules (indentation, import order) into CLAUDE.md rather than linters: cause — LLMs are probabilistic; linters are deterministic and cheap in enforcement. Source: HumanLayer / Builder.io consensus.
- Negations without alternatives ("never X"): cause — agent gets stuck when hitting the forbidden case. Solution: pair negation with an alternative: "Don't X; instead Y".
- Marking many rules with IMPORTANT/ YOU MUST: cause — sampling priority is diluted; if everything is important, nothing is. Source: DataCamp / Anthropic engineering guidance.

5) What belongs in CLAUDE.md vs where else (explicit mapping)
- Build/test/lint/typecheck commands: YES — one line per command (helps the model execute correct commands).
- Project layout map (3–10 lines): YES.
- Universal repo policies ("never commit .env"): YES — one or two critical rules with IMPORTANT.
- Conventions that Claude cannot infer from code (commit format, branch naming): YES.
- Code style / formatting / import order: NO — use linter/formatter + pre/post hooks.
- Pre-commit deterministic checks: NO — use PreToolUse hooks that can actually block with exit codes.
- Path blocking for sensitive files: NO — implement via PreToolUse hook that aborts.
- Large docs / API schema / DB schema: NO inlined via @import; reference via bare path and let Read tool fetch on demand.
- Organization-wide managed policies: NO in project file — these live in managed policy CLAUDE.md locations (/etc/claude-code/CLAUDE.md or platform equivalent).

6) Deliverables referenced (what to commit)
- CLAUDE.md — clean, committable version (≤200 lines), quick project-specific placeholders.
- CLAUDE.annotated.md — maintainer reference file with HTML comments explaining why each line exists (comments stripped on injection, zero runtime cost).
- CLAUDE.minimal.md — tiny template for quick projects (<500 tokens target).
- CLAUDE.monorepo.md — root monorepo guidance and pointers to per-package CLAUDE.md files (loaded on demand).

7) Operational rules for long sessions (workflow guidance)
- Run `/clear` between unrelated tasks to reset noise.
- Run `/compact <instructions>` proactively (don’t rely on auto-compact) to preserve chosen decisions.
- Use `/context` to inspect what files are loaded if a rule is not being applied.
- Use subagents / skills for heavy repository reading or long research tasks (they have isolated contexts and return compressed summaries).
- Prompt caching reduces billing cost for repeated turns but does not remove the file from the context window: it still occupies attention.

8) Caveats & versioning
- The guide is calibrated to Claude Code v2.1.x (Opus 4.7 / Sonnet 4.6 / Haiku 4.5) as of 2026-05-23. Future changes to model/system behavior can alter these recommendations.
- Some numeric thresholds like "150–200 instructions" are community heuristics derived from experiments (not single-paper facts). Use as conservative heuristics rather than hard limits.
- The wrapper <system-reminder> behavior is reproduced from proxy logs and issue reproductions; Anthropic has not published an explicit engineering note acknowledging that exact wrapper string.
- Auto-compact thresholds are inconsistent across community sources; proactively using `/compact` is recommended.
- .claude/ config dirs (hooks/settings/MCP servers) do NOT inherit hierarchically the same way CLAUDE.md does — this is an observed limitation/bug (issue: anthropics/claude-code#37344). In monorepos you must duplicate settings or launch from subdir.

---

## Full references (detailed)
- Anthropic — Memory & CLAUDE.md docs (official): https://code.claude.com/docs/en/memory  (referenced for: 200-line target, imports expansion, HTML comments behavior, root-file compaction behavior)
- Anthropic engineering guidance — emphasis usage (IMPORTANT / YOU MUST): internal engineering posts and guidance (cited in the research summary; relevant to sampling/priority tuning).
- Distyl AI, Jaroslawicz et al., "How Many Instructions Can LLMs Follow at Once?" — arXiv:2507.11538 (2025-07-15). Quote: "even the best frontier models only achieve 68% accuracy at the max density of 500 instructions".
- Scott Spence — "Measuring Claude Code skill activation with sandboxed evals" (2026-02-08). Reports skill activation % by model and shows strong effect of hooks/forced-eval.
- HumanLayer — blog posts and community templates (2025) recommending compact CLAUDE.md and avoiding overloading with style rules.
- Builder.io — blog posts on CLAUDE.md best practices and trimming /init output.
- DataCamp — tutorial on writing effective CLAUDE.md (emphasis on pairing negations with alternatives and limiting IMPORTANT markers).
- GitHub issues and proxy reproductions: anthropics/claude-code#18560 (Claude reproduces wrapper), anthropics/claude-code#37344 (reported config inheritance issue)
- MorphLLM and Steve Kinney blog posts — conflicting measurements on auto-compact thresholds (Morph: 64–75% vs Steve Kinney: ~95%). Use `/compact` proactively.
- Anthropic Support article on models usage & limits (prompt caching notes): https://support.claude.com/en/articles/14552983-models-usage-and-limits-in-claude-code

Full bibliography and the larger list of 164 sources referenced in the research are preserved in the uploaded CLAUDE.annotated.md and the main CLAUDE.md in this repo. See those files for per-line evidence links and timestamps.

---

## What changed in this README
- Expanded References and Evidence sections with the detailed citations you provided.
- Reproduced technical quotes and issue numbers (e.g., #18560 and #37344) so provenance is explicit.
- Clarified which items are heuristics vs. official documented behavior.
- Added operational caveats and explicit migration suggestions for monorepos and hooks.

---

If anything is missing from this expanded reference dump (URLs, dates, or verbatim quotes you want added), paste them and I'll append them verbatim. Commit: expanded README with full evidence.
