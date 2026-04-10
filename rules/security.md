---
id: security
description: >
  Security rules for source code — input validation, authentication, secrets,
  injection prevention, error handling. Based on OWASP Top 10 (2025), CWE Top 25,
  and OWASP Agentic Applications Top 10. Tiered by AI failure rate — categories
  where AI-generated code most frequently introduces vulnerabilities.
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/*.jsx"
  - "**/*.cs"
  - "**/*.py"
  - "**/*.go"
  - "**/*.java"
  - "**/*.kt"
  - "**/*.rb"
  - "**/*.rs"
  - "**/*.php"
  - "**/*.swift"
  - "**/*.vue"
  - "**/*.svelte"
---

# Security Rules

## Tier 1 — AI fails these most often

These address the vulnerability categories where AI-generated code most frequently fails.

- Validate and sanitize all user input at system boundaries — reject by default, allow by pattern
- Encode all output rendered in HTML/templates to prevent XSS (CWE-79)
- Use parameterized queries for all database operations — never string concatenation (CWE-89)
- Never hardcode secrets, API keys, tokens, or credentials — use environment variables or a secrets manager
- Never log sensitive data (passwords, tokens, PII, session IDs) — use structured logging with explicit field allowlists (CWE-117)
- Never use `eval()`, `exec()`, `Function()`, or equivalent dynamic code execution with user-controlled input (CWE-94)
- Check authorization on every data access and state-changing operation — not just at the route level (CWE-862, surged to #4 in CWE Top 25)

## Tier 2 — Common AI gaps

- Include CSRF tokens on all state-changing requests (forms, AJAX) (CWE-352, #3 in CWE Top 25)
- Use secure defaults: HTTPS everywhere, secure/httpOnly/sameSite cookies, least-privilege permissions
- Handle errors explicitly — no empty catch blocks, no swallowing exceptions, no leaking stack traces or internal paths to users (CWE-209)
- Verify dependencies exist in official registries and are actively maintained before importing — AI hallucinated packages are a real supply chain risk
- Validate file uploads: check type by content (not just extension), enforce size limits, prevent path traversal in filenames (CWE-434)
- Apply rate limiting on public-facing endpoints and authentication flows
- Set CORS headers to specific trusted origins — never use `*` in production
- Use HTTPS for all external requests — reject plain HTTP

## Tier 3 — Compliance & hardening

Foundational for production apps. Tier 3 because AI handles these better, not because they're less important.

- Require authentication for all non-public endpoints — fail closed (deny by default)
- Implement session management with secure, httpOnly, sameSite cookies and server-side session expiry
- Use constant-time comparison for secrets, tokens, and password hashes — never `==`
- Prevent open redirects — validate redirect targets against an allowlist of trusted domains
- Set security headers: Content-Security-Policy, X-Content-Type-Options, Strict-Transport-Security
- Apply the principle of least privilege to all service accounts, API keys, and database users
