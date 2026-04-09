# Security Checklist

Quick-reference checklist for security reviews. Extracted from `rules/security.md`
for on-demand loading during `/review`, `/eng`, and `/audit`.

## Pre-Commit Checks

- [ ] No hardcoded secrets, API keys, tokens, or credentials in the diff
- [ ] No sensitive data logged (passwords, tokens, PII, session IDs)
- [ ] `.env` files are gitignored; `.env.example` has placeholders only
- [ ] No `eval()`, `exec()`, `Function()` with user-controlled input

## Input & Output

- [ ] All user input validated and sanitized at system boundaries
- [ ] Parameterized queries for all database operations (no string concatenation)
- [ ] Output encoded for HTML/templates to prevent XSS
- [ ] File uploads validated by content type (not just extension), size limited
- [ ] CORS restricted to specific trusted origins (no wildcard `*` in production)

## Authentication & Authorization

- [ ] Authentication required for all non-public endpoints
- [ ] Authorization checked on every data access and state-changing operation
- [ ] CSRF tokens on all state-changing requests
- [ ] Rate limiting on public endpoints and auth flows
- [ ] Passwords hashed with bcrypt/scrypt/argon2 (12+ rounds)
- [ ] Sessions use httpOnly, secure, sameSite cookies

## Infrastructure

- [ ] HTTPS for all external requests
- [ ] Security headers set: CSP, HSTS, X-Content-Type-Options, X-Frame-Options
- [ ] Constant-time comparison for secrets and tokens
- [ ] Redirect targets validated against allowlist
- [ ] Stack traces and internal paths not exposed to users
- [ ] Dependency audit clean (no known critical/high vulnerabilities)
- [ ] Least privilege on service accounts, API keys, database users
