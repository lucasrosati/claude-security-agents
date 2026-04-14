# Claude Code Security Agents: Automated Red Team + Blue Team

> **50+ vulnerabilities found and fixed** using two Markdown files that turn Claude Code into offensive and defensive security specialists.

Drop two `.md` files into your project. Get automated pentesting and remediation — no frameworks, no infra, no security background required.

🇧🇷 [Leia em Português](./README.pt-BR.md)

---

## Table of Contents

1. [The Problem](#the-problem)
2. [The Solution](#the-solution)
3. [How It Works](#how-it-works)
4. [Setup (2 minutes)](#setup-2-minutes)
5. [Red Team — Offensive Agent](#red-team--offensive-agent)
6. [Blue Team — Defensive Agent](#blue-team--defensive-agent)
7. [Running the Agents](#running-the-agents)
8. [Real Vulnerabilities Found](#real-vulnerabilities-found)
9. [Customizing for Your Stack](#customizing-for-your-stack)
10. [FAQ](#faq)

---

## The Problem

You're building fast. Shipping features. But security keeps getting pushed to "later."

Meanwhile, your codebase quietly accumulates vulnerabilities: unsanitized HTML rendering, public API endpoints, admin routes without protection, webhook handlers that don't verify signatures. You won't find them in a code review — they hide in the interaction between components, in the assumptions you made at 2 AM, in the third-party integration you copy-pasted from the docs.

Hiring a pentester costs thousands. Running SAST tools gives you 200 generic warnings, most of them false positives. And you still have features to ship.

---

## The Solution

Two specialized agents for Claude Code — each is a single Markdown file (~120 lines):

| Agent | Role | Mode | What it does |
|-------|------|------|-------------|
| **Red Team** | Attacker | Read-only | Scans your codebase for vulnerabilities. Thinks like a pentester. Generates a structured report with severity, attack scenarios, and fix recommendations |
| **Blue Team** | Defender | Edit mode | Takes the Red Team report and fixes every vulnerability. Writes code, adds tests, commits changes |

They work as a feedback loop:

```
Red Team scans → finds 14 vulnerabilities → generates report
                                                    ↓
Blue Team reads report → fixes 12/14 → commits fixes
                                                    ↓
Red Team re-scans → finds 3 new issues → generates report
                                                    ↓
Blue Team fixes → hardening complete
```

Each cycle takes minutes, not days.

---

## How It Works

Claude Code supports [custom agents](https://docs.anthropic.com/en/docs/claude-code/agents) — Markdown files in `.claude/agents/` that give Claude a specialized persona, methodology, and constrained tool access.

The Red Team agent gets **read-only tools** (Read, Grep, Glob, npm audit). It can analyze everything but change nothing.

The Blue Team agent gets **edit tools** (Read, Write, Edit, git). It can modify code, install packages, run tests, and commit.

Both agents inherit your project context from `CLAUDE.md`, so they understand your stack, architecture, and constraints without any extra configuration.

---

## Setup (2 minutes)

### 1. Create the agents directory

```bash
mkdir -p .claude/agents
```

### 2. Copy the agent files

```bash
# Option A: Clone this repo and copy
git clone https://github.com/lucasrosati/claude-security-agents.git /tmp/security-agents
cp /tmp/security-agents/agents/red-team-scanner.md .claude/agents/
cp /tmp/security-agents/agents/blue-team-defender.md .claude/agents/

# Option B: Download directly
curl -o .claude/agents/red-team-scanner.md https://raw.githubusercontent.com/lucasrosati/claude-security-agents/main/agents/red-team-scanner.md
curl -o .claude/agents/blue-team-defender.md https://raw.githubusercontent.com/lucasrosati/claude-security-agents/main/agents/blue-team-defender.md
```

### 3. Run your first audit

Open Claude Code in your project and type:

```
Run the red-team-scanner agent on this codebase
```

That's it. No config files, no API keys, no Docker containers.

---

## Red Team — Offensive Agent

**File:** [`agents/red-team-scanner.md`](./agents/red-team-scanner.md)

**Model:** Sonnet (fast, cost-effective for scanning)
**Permission mode:** `plan` (read-only — cannot modify your code)

### What it analyzes

| Category | Examples |
|----------|---------|
| Authentication & Authorization | Broken auth, weak JWT validation, missing rate limiting, IDOR, privilege escalation |
| API Security | Public endpoints, missing object-level auth, mass assignment, enumeration |
| Input Validation & Injection | SQL/NoSQL injection, XSS (stored/reflected/DOM), SSRF, path traversal |
| File Uploads | MIME spoofing, missing magic byte validation, public storage exposure |
| Data Security & Compliance | Hardcoded secrets, PII in logs, insecure token storage, LGPD/HIPAA gaps |
| Infrastructure | CORS misconfiguration, missing security headers (CSP, HSTS), container issues |
| Admin Panel | Unprotected admin routes, hidden endpoints, session hijacking |
| Payment & Billing | Missing webhook verification, price manipulation, subscription bypass |
| Supabase-Specific | Broken RLS policies, service_role key exposure, cross-tenant access |

### Output format

For each vulnerability found:

```
[VULN-001] XSS via unsanitized AI content rendering
Severity: HIGH
Location: src/pages/StudySession.tsx:267
Description: AI-generated HTML rendered via dangerouslySetInnerHTML without sanitization
Attack scenario: Attacker injects <script> via manipulated study content → steals session tokens
Impact: Account takeover, data exfiltration
Fix: Sanitize with DOMPurify before rendering
```

---

## Blue Team — Defensive Agent

**File:** [`agents/blue-team-defender.md`](./agents/blue-team-defender.md)

**Model:** Sonnet
**Permission mode:** `acceptEdits` (can modify code with your approval)

### What it fixes

| Category | Actions |
|----------|---------|
| Authentication | Rate limiting, JWT hardening, session invalidation, role validation |
| API Security | Auth middleware, input validation (Zod/Joi), object-level authorization |
| Injection Prevention | DOMPurify for XSS, parameterized queries, path traversal checks, SSRF whitelists |
| File Uploads | Magic byte validation, size limits, file renaming, signed URLs |
| Data Security | Secrets to env vars, PII scrubbing from logs, encryption at rest, audit logging |
| Infrastructure | Security headers (CSP, HSTS, X-Frame-Options), strict CORS, HTTPS enforcement |
| Admin Panel | Auth + role middleware, admin action logging, rate limiting |
| Payment | Webhook signature verification, server-side price validation, idempotency keys |
| Supabase | RLS policy audit and fix, Edge Function input validation, cross-tenant prevention |

### Output format

For each fix applied:

```
[FIX for VULN-001] Sanitize AI content with DOMPurify
Status: FIXED
What was done: Added sanitizeAIContent() wrapper using DOMPurify with restrictive tag whitelist
Files modified: src/lib/sanitize.ts (new), src/pages/StudySession.tsx, + 3 other pages
Test: Injecting <script>alert(1)</script> into content now renders as plain text
Prevention: All AI content must pass through sanitizeAIContent() before dangerouslySetInnerHTML
```

### Hardening checklist

After fixing all vulnerabilities, the Blue Team generates a final checklist:

```
- [x] Security headers implemented
- [x] Rate limiting active on auth and API
- [x] CORS restricted to known origins
- [x] Logging and audit trail working
- [x] Sensitive data encrypted at rest and in transit
- [x] Secrets in env vars (none hardcoded)
- [x] Dependencies updated (0 critical/high CVEs)
- [x] RLS policies audited
- [x] Input validation on all endpoints
- [x] File upload secured
- [x] Admin panel protected
- [x] Webhooks verified
- [ ] LGPD compliance (consent, data retention, audit log)
```

---

## Running the Agents

### First audit (Red Team)

```bash
# In Claude Code
> Run the red-team-scanner agent on this codebase
```

The Red Team will:
1. Map your stack, endpoints, routes, and middleware
2. Run static analysis for vulnerable patterns
3. Run `npm audit` for known CVEs in dependencies
4. Review configuration (env vars, CORS, headers)
5. Trace sensitive data flows (PII, tokens, credentials)
6. Generate a numbered report (VULN-001, VULN-002, ...)

### Apply fixes (Blue Team)

```bash
# In Claude Code, after receiving the Red Team report
> Run the blue-team-defender agent. Here is the Red Team report: [paste report]
```

The Blue Team will:
1. Triage by severity (CRITICAL and HIGH first)
2. Reverse-engineer each attack vector
3. Implement the fix in code
4. Run tests to verify nothing broke
5. Commit each fix group: `fix(security): [VULN-XXX] description`
6. Generate a hardening checklist

### Re-scan (iterate)

```bash
# Run Red Team again to catch regressions or new issues
> Run the red-team-scanner agent again. Previous audit fixed VULN-001 through VULN-012.
```

Repeat until the Red Team report comes back clean.

### Quick commands

| Command | What it does |
|---------|-------------|
| `Run red-team-scanner` | Full security audit |
| `Run blue-team-defender` | Fix vulnerabilities from Red Team report |
| `Run red-team-scanner on src/api/` | Scan specific directory |
| `Run red-team-scanner focusing on authentication` | Targeted scan |

---

## Real Vulnerabilities Found

These are real vulnerabilities the agents found in a production SaaS application (React + Supabase + Edge Functions):

### Critical

| ID | Vulnerability | Impact |
|----|--------------|--------|
| VULN-002 | Self-referral in affiliate system | Users could generate commissions for themselves |
| VULN-010 | Duplicate commission processing | Same payment could trigger double commission |

### High

| ID | Vulnerability | Impact |
|----|--------------|--------|
| VULN-003 | XSS in 4 pages via `dangerouslySetInnerHTML` | Script injection in study content, session theft |
| VULN-004 | API endpoints with zero authentication | Anyone could consume AI quota |
| VULN-005 | Admin routes accessible via race condition | Regular users could access admin panel |
| VULN-006 | Referral field could be overwritten | Affiliate attribution manipulation |
| VULN-007 | Email templates vulnerable to HTML injection | Phishing via manipulated user names |
| VULN-008 | JWT parsed without signature verification | Authentication bypass |

### Medium

| ID | Vulnerability | Impact |
|----|--------------|--------|
| VULN-009 | No payload limit on bulk import endpoint | DoS via oversized request |
| VULN-011 | CORS wildcard on server-to-server webhook | Cross-origin request abuse |
| VULN-012 | CPF enumeration without authentication | LGPD violation, data harvesting |
| VULN-013 | SQL functions with incorrect volatility | Cache poisoning in query planner |

All fixed automatically by the Blue Team agent.

---

## Customizing for Your Stack

The agents work out of the box with any stack Claude Code supports. To optimize them for your project:

### 1. Change the identity line

Replace the project description in both files:

Both agent files start with a generic identity line. Replace it with your project context:

```markdown
# Default (generic)
You are the **Red Team Scanner** for this project.

# Customized (example)
You are the **Red Team Scanner** for Acme Corp — a B2B fintech SaaS handling payment processing.
```

### 2. Add stack-specific sections

If you use Firebase instead of Supabase, replace section 9:

```markdown
### 9. Firebase-Specific
- Firestore security rules audit
- Firebase Auth configuration
- Cloud Functions input validation
- Storage rules (public vs private)
```

### 3. Add compliance requirements

```markdown
### 10. SOC 2 Compliance
- Access logging for all sensitive operations
- Data encryption at rest and in transit
- Incident response procedures documented
- Vendor security assessment
```

### 4. Change the model

For deeper analysis, switch to Opus:

```yaml
model: opus  # more thorough, higher cost
```

For faster scans on large codebases:

```yaml
model: haiku  # faster, cheaper, less thorough
```

### 5. Change the language

The agents work in any language Claude supports. Just rewrite the instructions:

```markdown
# English version
You are the **Red Team Scanner** for this project.
## Identity
- Senior Security Engineer specialized in SaaS, OWASP Top 10, API Security
```

---

## FAQ

**Do I need security experience to use these agents?**
No. The Red Team explains each vulnerability with attack scenarios in plain language. The Blue Team fixes them automatically. You just review the changes.

**Will the Red Team modify my code?**
No. It runs in `plan` mode (read-only). It can read files, search code, and run `npm audit`, but cannot write, edit, or execute arbitrary commands.

**Will the Blue Team break my app?**
It runs tests after each fix and commits changes separately. If something breaks, you can revert individual commits. It also runs in `acceptEdits` mode, so you approve each change.

**How much does it cost?**
The agents use your existing Claude Code subscription or API credits. A full audit of a medium project (~100 files) typically takes 5-15 minutes with Sonnet.

**Can I run them in CI/CD?**
Not directly — Claude Code agents are interactive. But you can schedule periodic audits as part of your security workflow.

**Are the agents language-specific?**
The instructions are in Portuguese but work with any programming language. Claude Code understands the codebase regardless. Feel free to translate the agent files to your preferred language.

**What's the difference from tools like Snyk or SonarQube?**
SAST tools find pattern-based issues (known CVEs, simple code smells). These agents understand your architecture, trace data flows across components, and find logic vulnerabilities (self-referral, race conditions, privilege escalation) that static tools miss.

---

## Architecture

```
.claude/
└── agents/
    ├── red-team-scanner.md      ← ~120 lines, read-only
    └── blue-team-defender.md    ← ~140 lines, edit mode

Workflow:
┌──────────────┐    report    ┌──────────────┐    fixes    ┌──────────┐
│  Red Team    │ ──────────→ │  Blue Team   │ ─────────→ │ Codebase │
│  (scanner)   │             │  (defender)  │            │ (safer)  │
└──────────────┘             └──────────────┘            └──────────┘
       ↑                                                       │
       └───────────────── re-scan ─────────────────────────────┘
```

---

## Credits & Links

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — Anthropic's coding agent
- [Claude Code Agents](https://docs.anthropic.com/en/docs/claude-code/agents) — Custom agent documentation
- [OWASP Top 10](https://owasp.org/www-project-top-ten/) — Security vulnerability reference

---

**If this helped you secure your app, give the repo a ⭐ and share it with other devs shipping fast.**
