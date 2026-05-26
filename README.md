# claude-md-harness-audit

> A Claude Code skill that audits your `CLAUDE.md` against the built-in harness — and tells you which rules to cut because Claude already follows them anyway.

## The 30-second version

Claude Code ships with a **harness system prompt** that's injected before your `CLAUDE.md` every session. It already enforces ~50 instructions you've probably never seen.

Most `CLAUDE.md` authors don't know this and re-state rules the harness already enforces:

> "Read the file before editing." → the Edit tool already errors out if you don't.
>
> "Keep it simple. No unnecessary abstractions." → the harness says *"Three similar lines is better than a premature abstraction."*
>
> "Be concise. No fluff." → the harness says *"Your responses should be short and concise."*

Anthropic's own guidance is blunt about this:

> *"If Claude already does something correctly without the instruction, delete it or convert it to a hook."*
>
> *"Bloated CLAUDE.md files cause Claude to ignore your actual instructions!"*

So this skill reads your `CLAUDE.md`, sweeps it against a catalog of verbatim harness rules, and produces a line-by-line report of what to **cut**, what to **trim**, and what to **keep** because it's genuinely additive.

## Example output

```
# CLAUDE.md harness audit — ~/.claude/CLAUDE.md

## Stats
- Lines: 130  (Anthropic target: <200)
- Rules classified: 48
- Duplicates: 1 · Partial: 9 · Unique: 36 · Tool-enforced: 0 · Uncertain: 2
- Estimated cuttable lines: ~7

## Recommended cuts (DUPLICATE / TOOL-ENFORCED)

| Line in CLAUDE.md | Harness coverage (verbatim) | Action |
|---|---|---|
| L121: `Long-form prose only on explicit request.` | `# Text output`: "Match responses to the task: a simple question gets a direct answer, not headers and sections." | Cut |

## Keep (UNIQUE — defensible content)
- Autonomy matrix (work-type → default behavior)
- Failure protocol with retry budget
- Apply / defer / reject for root config edits
- Confidence calibration: "I know" vs "I think" vs "I'd guess"
...
```

Then you pick which cuts to apply, and the skill produces the diff.

## Install

### Option A — Ask Claude Code to install it for you

Paste this into any Claude Code session:

> Install the `claude-md-harness-audit` skill from https://github.com/vibecodoor/claude-md-harness-audit — clone the repo into `~/.claude/skills/claude-md-harness-audit/`, verify `SKILL.md` and `references/harness-rules.md` are in place, and confirm the skill is registered. Then run it on my global `CLAUDE.md` so I can see a sample audit.

Claude will clone the repo, verify the install, and immediately demo the skill on your own `CLAUDE.md`.

### Option B — Manual

```bash
git clone https://github.com/vibecodoor/claude-md-harness-audit \
  ~/.claude/skills/claude-md-harness-audit
```

That's it. Next time you start Claude Code, the skill is available.

## Use

In a Claude Code session, say one of:

- *"audit my CLAUDE.md"*
- *"check my global CLAUDE.md for duplicates with the harness"*
- *"what can I cut from my CLAUDE.md?"*
- *"is my CLAUDE.md any good?"*

The skill triggers, reads your file + the harness catalog, produces the report, waits for your approval before touching anything.

## What's inside

```
claude-md-harness-audit/
├── SKILL.md                   ← Workflow + classification rules
└── references/
    └── harness-rules.md       ← Catalog of harness sections with verbatim quotes
```

`harness-rules.md` is the actual intellectual content. It's sourced from:

- [Anthropic Best Practices for Claude Code](https://code.claude.com/docs/en/best-practices)
- [Anthropic Memory / CLAUDE.md docs](https://code.claude.com/docs/en/memory)
- [Piebald-AI/claude-code-system-prompts](https://github.com/Piebald-AI/claude-code-system-prompts) — version-tracked extraction of the harness
- Live `<claudeMd>` block visible in active Claude Code sessions

## Why this exists (and what it isn't)

There's already an official `claude-md-improver` skill and a bunch of community CLAUDE.md auditors. **None of them know about the harness.** They treat `CLAUDE.md` as a standalone artifact. This one treats it as a **delta over the harness** — which is what Anthropic's own docs say you should do.

This skill **subtracts**. It will not:

- Add new sections (commands, architecture, paths) → use `claude-md-improver`
- Optimize all `.md` files in your project → use `tokenusage-optimizer`
- Generate a `CLAUDE.md` from scratch → use `/init`

## Limitations & disclaimer

**The harness changes per Claude Code version.** The catalog in `references/harness-rules.md` is a snapshot dated at the top of the file. If the snapshot is older than ~3 months, treat findings as approximate and verify against the current harness.

**The "cut duplicates" recommendation is grounded in Anthropic's official guidance, but the empirical claim that cuts measurably improve adherence is not yet validated by public experiment.** It's plausible duplication acts as reinforcement instead — Anthropic explicitly rejects this and recommends emphasis (`IMPORTANT:` / `YOU MUST`) over restatement, but the question isn't settled. Use your judgment for high-stakes rules.

**This skill quotes the harness verbatim** so you can verify every recommendation yourself. If a recommended cut surprises you, check the cited section in the catalog before applying.

## Updating the harness catalog

When Claude Code ships a new version:

1. Diff the current system prompt against the snapshot date at the top of `references/harness-rules.md`. Sources: [Piebald-AI/claude-code-system-prompts](https://github.com/Piebald-AI/claude-code-system-prompts) or the live `<claudeMd>` block in your session.
2. Add new sections with verbatim quotes.
3. Mark removed quotes as `[REMOVED in vYYYY-MM]` rather than deleting — old audits may reference them.
4. Bump the snapshot date at the top.

PRs welcome.

## License

MIT — see [LICENSE](LICENSE).
