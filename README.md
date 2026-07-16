# shadow-decisions

**Your AI made 47 product decisions in your app last night. You reviewed zero of them.**

`shadow-decisions` is a Claude skill that scans an AI-built codebase and surfaces every product decision the AI silently made for you — session lifetimes, what "delete account" actually does, whether your free tier is bounded, what users see when things fail — triaged so you review the 5 that matter, not a wall of 47.

> ADRs record what your engineers decided. This surfaces what your AI decided **without asking you.**

## The problem

When you vibe-code an app, the AI doesn't just write code — it makes product choices:

- Sessions never expire (or expire in an hour)
- "Delete account" flags a row and keeps everything
- The free AI feature has no spend cap — your API key, their loop
- Marketing emails default to on; profiles default to public
- Errors show users a raw stack trace

Nobody chose these. They were defaults, and you find out when a user complains, a bill arrives, or a regulator writes. Security scanners won't catch them — none of these are vulnerabilities. They're **product decisions**, and you're the PM whether you noticed or not.

## What you get

Run the skill on your project. It writes `SHADOW-DECISIONS.md` to your repo:

- **🔴 Dangerous (max 5)** — review before shipping. User harm, money loss, legal exposure, unbounded cost.
- **🟡 Worth a look** — defensible defaults to consciously accept.
- **🟢 Fine** — one-line inventory, no homework.

Every finding cites **file and line** (no vibes-based claims), explains the consequence in plain English (no code literacy required), and ships a **paste-ready fix prompt** — detection to correction in one paste.

**Diff mode:** re-run after changes and it reports only what's NEW, CHANGED, or RESOLVED against the last sign-off. It's a pre-ship habit, not a one-time audit.

See a full real run in [`examples/sample-decision-log.md`](examples/sample-decision-log.md) — 11 decisions extracted from a 4-file demo app.

## Install

**Claude Code:** copy this folder into `~/.claude/skills/` (or your project's `.claude/skills/`), then ask: *"Review this app's product decisions before I ship."*

**Claude.ai / Cowork:** upload the folder as a skill in Settings → Capabilities, or attach `SKILL.md` and the `references/` folder to your project.

## What it deliberately is NOT

- **Not a security scan.** Use a security-review tool for vulnerabilities; this covers choices, not exploits.
- **Not code review.** It won't comment on your architecture, framework, or code quality.
- **Not an ADR generator.** ADR tools document engineering decisions for engineers. This extracts *product* decisions for the human accountable for the product.

## Pairs with

[`claude-md-for-builders`](https://github.com/aditti30-prog/claude-md-for-builders) — behavioral guardrails that shape how the AI codes. That repo is **prevention while building**; this one is **inspection before shipping**. Use both: rules in the root, review before launch.

## The taxonomy

The skill's engine is a curated taxonomy of ~40 product decisions across 8 categories — auth & sessions, data retention & privacy, deletion & offboarding, money, limits & quotas, errors, defaults & permissions, notifications ([`references/taxonomy.md`](references/taxonomy.md)). Each entry encodes what to look for, why it matters, how severe it defaults to, and how to fix it.

## Contributing

Found a decision your AI made that isn't in the taxonomy? Open an issue with the category, the code pattern, and what it cost you. The goal is the same as its sibling repo: a living document grounded in real production surprises, not hypotheticals.

## License

MIT
