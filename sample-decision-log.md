# Product Decision Log
Generated: 2026-07-16 · Scan: full · Decisions found: 11
> Your AI made 11 product decisions in this codebase. Review the 5 flagged below, then sign off.

*(This is a real run of the skill against the 4-file demo app in `demo-app/` — a deliberately typical vibe-coded backend.)*

## 🔴 Dangerous — review before shipping (5)

### [A1] Login tokens never expire
- **Where:** `src/auth.js:4`
- **What was decided:** JWTs are signed with no expiry.
- **Consequence:** Anyone who steals a login token — from a shared laptop, a leaked log, anywhere — can use it forever. There is no way to log a stolen session out.
- **The alternative:** 14–30 day expiry with refresh rotation; industry default.
- **Fix prompt:** `Add expiresIn: '14d' to jwt.sign in src/auth.js:4 and implement refresh token rotation`

### [E4] The free AI feature has no spend cap
- **Where:** `src/ai.js:4-6`
- **What was decided:** Any user can call the paid LLM API unlimited times, for free.
- **Consequence:** One person with a loop script spends your API budget overnight. You pay; they don't.
- **The alternative:** Per-user daily cap plus a global monthly budget check.
- **Fix prompt:** `Add a per-user daily cap (e.g. 20 calls) and a global monthly budget check before the openai call in src/ai.js:4`

### [C1] "Delete account" doesn't delete anything
- **Where:** `src/user.js:12`
- **What was decided:** Deletion sets a flag; the email, name, and all user content stay in the database forever.
- **Consequence:** Users are told their account is deleted when it isn't — a broken promise, and a legal problem in most markets.
- **The alternative:** Cascade deletion (or anonymization), or honestly label the button "Deactivate".
- **Fix prompt:** `Make deleteAccount in src/user.js:11 hard-delete the user row and cascade to their content, or rename the UI action to Deactivate`

### [F1] Users see raw stack traces when the AI feature fails
- **Where:** `src/ai.js:9`
- **What was decided:** The full error stack is returned to the browser.
- **Consequence:** Users see terrifying internals; attackers see your file paths and dependencies.
- **The alternative:** Friendly message + error ID to the user; details to server logs.
- **Fix prompt:** `In src/ai.js:9 return a generic error message with an error ID and log e.stack server-side instead`

### [H2] Comment emails ignore unsubscribe — and failures are swallowed
- **Where:** `src/mailer.js:3`
- **What was decided:** Every comment emails the user with no unsubscribe check, no frequency cap, and a silently-ignored failure (`.catch(() => {})`).
- **Consequence:** An active thread means 40 emails in an hour with no way out (illegal in most markets) — and if sending breaks, nobody ever knows.
- **The alternative:** Check an unsubscribe flag before sending, batch per hour, log failures.
- **Fix prompt:** `In src/mailer.js:3 check user.unsubscribed before sending, batch comment notifications hourly, and log send failures`

## 🟡 Worth a look (4)

### [G1] User profiles are public by default
- **Where:** `src/user.js:8` · New users are discoverable before they've chosen anything. Most products default private with an explicit publish step.
- **Fix prompt:** `Change profileVisibility default to 'private' in src/user.js:8`

### [G2] Marketing emails are on by default
- **Where:** `src/user.js:7` · Opt-out marketing is legal in some markets, illegal in others (EU). Decide your market first.
- **Fix prompt:** `Change marketingEmails default to false in src/user.js:7`

### [B3] Logins log user emails in plaintext
- **Where:** `src/auth.js:11` · Logs get shipped to third parties and kept forever; PII in them leaks quietly.
- **Fix prompt:** `Replace user.email with user.id in the log statement at src/auth.js:11`

### [A3] Nothing limits password guessing — *Verify*
- **Where:** `src/auth.js:6-13` · No rate limiting in the login handler; grepped `rateLimit|attempts|lockout` across src/ with no hits. Marked Verify in case limiting exists at proxy level.
- **Fix prompt:** `Add rate limiting (5 attempts per 15 min per IP+account) to the login handler in src/auth.js:6`

## 🟢 Fine (2)
- [A2] Passwords stored hashed, checked via `check()` (`src/auth.js:8`)
- [F1] Failed logins return a neutral message that doesn't reveal whether the email exists (`src/auth.js:9`)

## Skipped categories
D. Money — no payment code found (grepped `stripe|price|invoice|billing`).

## Sign-off
- [ ] I have reviewed the 5 Dangerous items above
Reviewed by: ______  Date: ______

<!-- ledger: do not edit below. Machine-readable state for diff mode -->
<!-- {"schema":1,"decisions":[{"id":"A1","status":"dangerous","where":"src/auth.js:4","summary":"tokens never expire"},{"id":"E4","status":"dangerous","where":"src/ai.js:4","summary":"no spend cap on LLM calls"},{"id":"C1","status":"dangerous","where":"src/user.js:12","summary":"soft delete presented as delete"},{"id":"F1","status":"dangerous","where":"src/ai.js:9","summary":"raw stack traces to users"},{"id":"H2","status":"dangerous","where":"src/mailer.js:3","summary":"no unsubscribe check, swallowed send failures"},{"id":"G1","status":"review","where":"src/user.js:8","summary":"profiles public by default"},{"id":"G2","status":"review","where":"src/user.js:7","summary":"marketing on by default"},{"id":"B3","status":"review","where":"src/auth.js:11","summary":"PII in logs"},{"id":"A3","status":"review","where":"src/auth.js:6","summary":"no login rate limiting (verify)"},{"id":"A2","status":"fine","where":"src/auth.js:8","summary":"passwords hashed"},{"id":"F1b","status":"fine","where":"src/auth.js:9","summary":"neutral auth error"}]} -->
