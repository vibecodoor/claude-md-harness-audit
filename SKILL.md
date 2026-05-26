---
name: claude-md-harness-audit
description: Audits a CLAUDE.md file against the Claude Code harness that is currently loaded in your active session (your own system prompt) and flags lines that duplicate behavior the harness already enforces. Anthropic's own guidance says to cut anything Claude already does — this skill finds those lines. Use whenever the user wants to improve, slim, shorten, audit, deduplicate, declutter, or review a CLAUDE.md — global (~/.claude/CLAUDE.md) or project-level — even if they don't say the word "harness". Trigger on phrases like "is my CLAUDE.md good", "what can I cut", "review my global config", "make CLAUDE.md leaner", "my CLAUDE.md feels bloated", "audit my Claude instructions", "find duplicates in CLAUDE.md", "system prompt overlap". Also trigger before publishing a CLAUDE.md template, after a major rewrite, or when the file is over ~150 lines. Prefer this skill over claude-md-improver when the user's goal is subtraction (removing redundancy) rather than addition (adding project-specific content).
---

# CLAUDE.md harness audit

Audits a CLAUDE.md against the Claude Code harness — the system prompt currently active in this session. Flags rules that duplicate harness behavior, citing your active harness verbatim. Anthropic's guidance: *"If Claude already does something correctly without the instruction, delete it or convert it to a hook."*

## Why this exists

Claude Code injects a substantial harness system prompt before any CLAUDE.md. Most CLAUDE.md authors never see it, so they re-state rules the harness already enforces — bloating context and (per Anthropic) *reducing* adherence to the rules that actually matter.

Standard CLAUDE.md auditors (including the official `claude-md-improver`) treat CLAUDE.md as standalone. This skill treats it as a **delta over the harness** — and uses **the live harness loaded in your current session** as the comparison baseline, so the audit is always against the exact version you're running.

## When to use

- User asks to audit / improve / slim / review a CLAUDE.md
- User mentions: harness, system prompt, duplicates, overlap, what to cut, redundant rules
- CLAUDE.md is over ~150 lines or grew organically
- Before publishing a CLAUDE.md as a template

## When NOT to use

- User wants project-specific *content* added (commands, architecture, paths) → use `claude-md-improver`
- User wants generic token reduction across many MD files → use `tokenusage-optimizer`
- User wants a fresh CLAUDE.md from scratch → use `init`

## Workflow

### 1. Locate the target CLAUDE.md

Default search order:
1. Path the user named explicitly
2. `./CLAUDE.md` in cwd
3. `~/.claude/CLAUDE.md` (user-global)
4. If multiple candidates exist, ask which one.

Read the whole file.

### 2. Use your own active harness as the baseline

You — the Claude instance running this skill — already have the harness loaded as your system prompt. That is your comparison baseline. You do not need to fetch it from anywhere; it's already in your context as the rules that govern your behavior in this session.

When auditing, recall and quote the relevant harness sections **verbatim** from your own system prompt. Harness sections to keep in mind (this is the typical structure — your exact section names may differ slightly):

- `# Doing tasks` — YAGNI, no premature abstraction, validate only at boundaries, no comments by default, OWASP awareness, prefer editing existing files
- `# Executing actions with care` — destructive-operation guardrails, root-cause over shortcut, investigate unfamiliar state
- `# Tone and style` — concise output, file:line reference convention
- `# Text output` — no preamble, end-of-turn summary cap, match response size to task
- `# Using your tools` — Read-before-Edit (tool-enforced), parallel independent calls, prefer dedicated tools over Bash
- Tool specs themselves enforce some rules without instruction (e.g. Edit errors if Read wasn't called)

If your active harness uses different section names or covers different rules, **trust your active harness over this list**. This list is a hint about what to look for, not a substitute for what's actually in your system prompt right now.

For Anthropic best-practices quotes that aren't in the harness itself (e.g. *"If Claude already does something correctly without the instruction, delete it or convert it to a hook"* from the Best Practices doc), you may cite them from training-data recall. If a quote feels uncertain, label it as a paraphrase rather than verbatim.

### 3. Classify every line/bullet

For each rule or bullet in the target CLAUDE.md, assign one bucket:

| Bucket | Meaning | Action |
|---|---|---|
| **DUPLICATE** | Your active harness already enforces the same behavior — quote the exact harness sentence | Recommend **cut** |
| **PARTIAL** | Harness covers the spirit but user's phrasing adds a specific constraint or nuance | Recommend **trim** to just the delta, or convert to `IMPORTANT:` emphasis if the harness rule needs reinforcement |
| **UNIQUE** | No harness equivalent — additive value | **Keep** |
| **TOOL-ENFORCED** | A tool spec already errors out without this rule (e.g. Read-before-Edit) | Recommend **cut** |
| **UNCERTAIN** | Plausibly duplicated but you can't quote the harness verbatim with confidence | Flag for user judgment, don't auto-recommend |

For DUPLICATE and PARTIAL: always quote the harness line verbatim from your active system prompt. Don't paraphrase — the user must be able to verify against what they can see in their session.

### 4. Produce the report

Output exactly this structure (no preamble, no trailing summary):

```
# CLAUDE.md harness audit — <path>

## Stats
- Lines: <N>  (Anthropic target: <200)
- Rules classified: <count>
- Duplicates: <count> · Partial: <count> · Unique: <count> · Tool-enforced: <count> · Uncertain: <count>
- Estimated cuttable lines: <N>

## Recommended cuts (DUPLICATE / TOOL-ENFORCED)

| Line in CLAUDE.md | Harness coverage (verbatim) | Action |
|---|---|---|

## Trim / convert to emphasis (PARTIAL)

| Line in CLAUDE.md | What harness covers | Suggested delta |
|---|---|---|

## Keep (UNIQUE — defensible content)

- <bullet list of high-value rules>

## Uncertain — needs your call

- <bullet list with both interpretations>

## Suggested next step

<one sentence: e.g. "Apply cuts → file drops from 130 → 95 lines. Want me to produce the diff?">
```

**Worked example** of one populated DUPLICATE row (for calibration):

| Line in CLAUDE.md | Harness coverage (verbatim) | Action |
|---|---|---|
| L42: `Keep it simple. YAGNI. No unnecessary abstractions or "flexibility" I didn't ask for.` | `# Doing tasks`: *"Don't add features, refactor, or introduce abstractions beyond what the task requires... Three similar lines is better than a premature abstraction."* | Cut — verbatim coverage |

Note the format: line number, the user's exact text, the harness section name + verbatim quote, and a one-word action. If you can't fill the verbatim column from your own active system prompt, the classification was wrong — downgrade to UNCERTAIN.

### 5. On approval, produce the diff

Only after the user approves cuts:
- Show the exact diff (old line → removed, or old line → new line for PARTIAL trims)
- Per global CLAUDE.md conventions on root-config edits: present each change with **apply / defer / reject** options
- Apply approved cuts in a single pass

## Hard rules

- **Quote, don't paraphrase.** Every DUPLICATE claim must include the verbatim harness sentence from your active session's system prompt — so the user can verify the overlap by inspecting the same prompt you're running on. A paraphrase is unfalsifiable. If you can't locate the exact quote in your own harness, downgrade the classification to UNCERTAIN.
- **Trust the live harness over assumptions.** Your active system prompt is the ground truth for this audit. If a rule you'd expect to be there isn't actually present in your session's harness, don't flag the user's CLAUDE.md line as DUPLICATE — mark it UNIQUE or UNCERTAIN.
- **State which session this audit ran in.** Include a one-line note like *"Audited against the harness loaded in this Claude Code session (model: <model>, date: <today>)."* The user needs to know the audit is tied to a specific runtime, not a static catalog.
- **Report first, edit second.** Always emit the audit report and wait for the user to pick which cuts to apply. Auto-editing a CLAUDE.md without review is a unilateral root-config change.
- **Subtract, don't add.** Recommending new content (Code Style sections, Architecture overviews, generic best-practices) is `claude-md-improver`'s job and contradicts this skill's purpose. If during the audit you notice a missing project-specific section, mention it once at the end and suggest invoking `claude-md-improver` separately — don't try to do both jobs.
