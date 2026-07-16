---
name: shadow-decisions
description: Surface the product decisions an AI silently made inside a codebase, and present them for human sign-off. Use this skill whenever the user asks to review, audit, or check an AI-built or vibe-coded app before launch or shipping; asks "what did the AI decide", "what am I missing", "is this safe to ship", or "review my app/product"; asks for a product review of a codebase (as opposed to a code or security review); or re-runs a review on a project that already contains a SHADOW-DECISIONS.md file. Trigger even if the user doesn't use the word "decision" — any pre-ship review request on an AI-built product qualifies.
---

# Shadow Decisions

When an AI builds an app, it makes dozens of product decisions the builder never sees: how long sessions last, what happens to data when an account is deleted, whether the free tier has limits, what a user sees when something fails. Builders discover these when users complain.

This skill scans a codebase, extracts those implicit **product** decisions (not engineering decisions — see Scope), and produces a triaged Product Decision Log the builder can review and sign off in ~10 minutes.

**Positioning to keep in mind:** ADRs record what engineers decided. This surfaces what the AI decided without asking anyone. Do not drift into architecture review.

## Scope — what counts as a product decision

IN: anything a PM would decide — user-facing behavior, data handling, money handling, limits, defaults, permissions, communications.
OUT: engineering choices (framework, database, code structure), security vulnerabilities (point users to a security-review tool instead), code quality. If a finding is purely technical, drop it.

## Workflow

### Step 0 — Detect mode

Check the project root for `SHADOW-DECISIONS.md`.
- Absent → **Full scan** (Steps 1–4).
- Present → **Diff mode** (Step 5).

### Step 1 — Inventory the codebase

Map the project before judging it. Identify:
- Auth/session handling (middleware, token issuance, login routes)
- Data models and what personal data they store
- Deletion/cancellation paths
- Payment/billing code (if any)
- Rate limits, quotas, tier logic
- Error handlers and what they expose to users
- Default values in models, configs, and settings
- Email/notification triggers

Use file search (grep for signals like `expires`, `delete`, `retry`, `limit`, `default`, `session`, `stripe`, `sendEmail`, `catch`) rather than reading every file. Note file paths and line numbers as you go — every finding must cite them.

### Step 2 — Extract decisions against the taxonomy

Read `references/taxonomy.md`. For each applicable taxonomy item, determine what the codebase actually does and record it as a decision. Skip categories with no relevant code (e.g., skip Money if nothing touches payments) — note skipped categories in one line.

**Accuracy rules (non-negotiable):**
1. Never report a decision without a `file:line` citation. If you can't point to code, don't claim it.
2. If behavior is ambiguous or depends on runtime config you can't see, mark the finding **Verify** instead of asserting it.
3. Report what the code DOES, including decisions-by-omission ("no rate limiting exists on /login" — cite the login handler that lacks it).
4. Absence claims require evidence of search: state what you grepped for before claiming something doesn't exist.

### Step 3 — Triage

Sort findings into exactly three buckets:

- **🔴 Dangerous — review before shipping.** Hard cap of 5. Real user harm, money loss, legal/privacy exposure, or unbounded cost. If more than 5 qualify, keep the worst 5 and demote the rest.
- **🟡 Worth a look.** Defensible defaults the builder should consciously accept.
- **🟢 Fine.** Reasonable decisions. List as a one-line-each inventory — no elaboration.

The output is a sign-off ritual, not an inventory. Ten minutes of the builder's attention goes to the red bucket.

### Step 4 — Write the Product Decision Log

Write `SHADOW-DECISIONS.md` in the project root using this exact structure:

```markdown
# Product Decision Log
Generated: <date> · Scan: full · Decisions found: <n>
> Your AI made <n> product decisions in this codebase. Review the <k> flagged below, then sign off.

## 🔴 Dangerous — review before shipping (<k>)

### [<taxonomy-id>] <Decision in one plain sentence>
- **Where:** `path/to/file.ts:42`
- **What was decided:** <what the code does, one sentence>
- **Consequence:** <plain English, what a real user experiences / what it costs you>
- **The alternative:** <what most products do instead>
- **Fix prompt:** `<one paste-ready instruction to an AI coding tool, referencing the file>`

## 🟡 Worth a look (<m>)
<same finding format, may compress Consequence/Alternative to one line each>

## 🟢 Fine (<j>)
- [<id>] <one line> (`file:line`)

## Skipped categories
<one line: which taxonomy categories had no relevant code>

## Sign-off
- [ ] I have reviewed the Dangerous items above
Reviewed by: ______  Date: ______

<!-- ledger: do not edit below. Machine-readable state for diff mode -->
<!-- {"schema":1,"decisions":[{"id":"A1","status":"dangerous","where":"src/auth.ts:42","summary":"..."}]} -->
```

The HTML-comment ledger at the bottom must contain every finding (all buckets) as compact JSON — it is what diff mode compares against.

Rules for fix prompts: one line, imperative, names the file, states the target value ("Set JWT expiry to 14 days with refresh rotation in src/auth/session.ts"). Never vague ("improve session handling").

After writing the file, tell the user in chat: the headline number, the count per bucket, and the top 1–2 Dangerous items in one sentence each. Do not restate the whole log in chat.

### Step 5 — Diff mode

If `SHADOW-DECISIONS.md` exists:
1. Parse the JSON ledger from the bottom comment.
2. Re-run Steps 1–3 (a re-scan may be scoped to changed files if git history makes that clear — otherwise full re-scan).
3. Report **only**:
   - **NEW** — decisions not in the ledger
   - **CHANGED** — same taxonomy item, different behavior or severity ("session expiry changed: none → 30 days")
   - **RESOLVED** — previously flagged, now fixed
4. Rewrite `SHADOW-DECISIONS.md` in full (updated findings + updated ledger), and mark the header `Scan: diff`.
5. In chat, report the delta only. If nothing changed: say so in one line. An unchanged re-scan is a passing result, not a failure.

## Tone

Plain English throughout. The reader may not be able to read the code — every Consequence line must make sense to someone who has never opened the file. No jargon in findings ("anyone who steals a login token can use it forever", not "JWTs lack TTL enforcement").
