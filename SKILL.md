---
name: claude-md-harness-audit
description: Audits a CLAUDE.md file against the Claude Code harness (the built-in system prompt that ships with Claude Code) and flags lines that duplicate behavior the harness already enforces. Anthropic's own guidance says to cut anything Claude already does — this skill finds those lines. Use whenever the user wants to improve, slim, shorten, audit, deduplicate, declutter, or review a CLAUDE.md — global (~/.claude/CLAUDE.md) or project-level — even if they don't say the word "harness". Trigger on phrases like "is my CLAUDE.md good", "what can I cut", "review my global config", "make CLAUDE.md leaner", "my CLAUDE.md feels bloated", "audit my Claude instructions", "find duplicates in CLAUDE.md", "system prompt overlap". Also trigger before publishing a CLAUDE.md template, after a major rewrite, or when the file is over ~150 lines. Prefer this skill over claude-md-improver when the user's goal is subtraction (removing redundancy) rather than addition (adding project-specific content).
---

# CLAUDE.md harness audit

Audits a CLAUDE.md against the Claude Code harness (the built-in system prompt). Flags rules that duplicate harness behavior — Anthropic's guidance: *"If Claude already does something correctly without the instruction, delete it or convert it to a hook."*

## Why this exists

Claude Code injects a substantial harness system prompt before any CLAUDE.md. Most CLAUDE.md authors never see it, so they re-state rules the harness already enforces — bloating context and (per Anthropic) *reducing* adherence to the rules that actually matter.

Standard CLAUDE.md auditors (including the official `claude-md-improver`) treat CLAUDE.md as standalone. This skill treats it as a **delta over the harness**.

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

### 1. Locate the target file

Default search order:
1. Path the user named explicitly
2. `./CLAUDE.md` in cwd
3. `~/.claude/CLAUDE.md` (user-global)
4. If multiple candidates exist, ask which one.

Read the whole file. Also read `references/harness-rules.md` from this skill — it's the catalog you'll compare against.

### 2. Classify every line/bullet

For each rule or bullet in the target CLAUDE.md, assign one bucket:

| Bucket | Meaning | Action |
|---|---|---|
| **DUPLICATE** | Harness already enforces the same behavior — cite the exact harness section | Recommend **cut** |
| **PARTIAL** | Harness covers the spirit but user's phrasing adds a specific constraint or nuance | Recommend **trim** to just the delta, or convert to `IMPORTANT:` emphasis if the harness rule needs reinforcement |
| **UNIQUE** | No harness equivalent — additive value | **Keep** |
| **TOOL-ENFORCED** | A tool spec already errors out without this rule (e.g. Read-before-Edit) | Recommend **cut** |
| **UNCERTAIN** | Plausibly duplicated but harness wording is ambiguous | Flag for user judgment, don't auto-recommend |

For DUPLICATE and PARTIAL: always quote the harness line verbatim. Don't paraphrase — the user must be able to verify.

### 3. Produce the report

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

Note the format: line number, the user's exact text, the harness section name + verbatim quote, and a one-word action. If you can't fill the verbatim column from `references/harness-rules.md`, the classification was wrong — downgrade to UNCERTAIN.

### 4. On approval, produce the diff

Only after the user approves cuts:
- Show the exact diff (old line → removed, or old line → new line for PARTIAL trims)
- Per global CLAUDE.md rule on root-config edits: present each change with **apply / defer / reject** options
- Apply approved cuts in a single pass

## Hard rules

- **Quote, don't paraphrase.** Every DUPLICATE claim must include the verbatim harness sentence so the user can verify the overlap themselves. A paraphrase is unfalsifiable — they can't tell whether the harness really says what you claim. If you can't locate the exact quote, downgrade the classification to UNCERTAIN.
- **Don't invent harness rules.** Only cite entries that appear in `references/harness-rules.md`. The harness is a moving target across Claude Code versions; if you fabricate or recall a rule from training data, you'll recommend cuts based on rules that no longer exist. When in doubt: UNIQUE (keep), not DUPLICATE.
- **State the harness snapshot date** (top of `references/harness-rules.md`) in the report. The user needs to know how stale the comparison baseline is — a 6-month-old snapshot may miss new harness behaviors.
- **Report first, edit second.** Always emit the audit report and wait for the user to pick which cuts to apply. Auto-editing a CLAUDE.md without review is exactly the kind of unilateral root-config change the user's own CLAUDE.md forbids (`apply / defer / reject` rule).
- **Subtract, don't add.** Recommending new content (Code Style sections, Architecture overviews, generic best-practices) is `claude-md-improver`'s job and contradicts this skill's purpose. If during the audit you notice a missing project-specific section, mention it once at the end and suggest invoking `claude-md-improver` separately — don't try to do both jobs.

## Reference

`references/harness-rules.md` — catalog of harness sections and the behaviors they enforce, with verbatim quotes. **Always read this** when running an audit; it's the comparison baseline. Update when the harness ships a new version.
