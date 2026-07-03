---
name: top-security
description: >
  Security review checklist for Tent of Presence engineers on Node.js, Express.js, PHP, MySQL,
  and React. Use this whenever you're writing or reviewing code that touches authentication,
  database queries, API endpoints, user input, secrets management, or third-party dependencies
  — even if nobody explicitly asked for a security review. Trigger when someone asks "is this
  safe?", "should I use X?", when reviewing a PR that changes auth logic, data access, or server
  config, or before opening any PR involving sensitive functionality.
---

# top-security — Tent of Presence Security Checklist

Catch the most common vulnerabilities before they reach code review or production.

**Stack:** Node.js · Express.js · PHP · MySQL · React · AWS

Note findings as:
```
[SEVERITY] Location: description — recommended fix
```
Severity: CRITICAL / HIGH / MEDIUM / LOW (mirrors incident P-levels)

---

## 1. SQL Injection (MySQL)

Parameterised queries are the single most effective protection. String concatenation is never
safe — future refactors silently remove validation assumptions.

- Any query building a string with `+`, `.`, or template literals containing variables?
- User input (URL params, body, headers) passed directly into a query string?
- ORM raw query escape hatches (`.query()`, `DB::statement()`) also parameterised?

```js
// Vulnerable — never do this
db.query(`SELECT * FROM users WHERE email = '${req.body.email}'`);

// Safe
db.query('SELECT * FROM users WHERE email = ?', [req.body.email]);
```

---

## 2. Authentication and Session Security

**JWTs:**
- `exp` (expiry) set on every token? Tokens without expiry are permanent credentials if stolen.
- Signing secret in an environment variable, not hardcoded?
- Algorithm explicitly set to `HS256` or `RS256`? The `none` algorithm attack is real.
- Tokens validated on every protected route, not just at login?

**Passwords:**
- Hashed with `bcrypt` (Node) or `password_hash(PASSWORD_BCRYPT)` (PHP)? MD5/SHA1 are not acceptable.
- Minimum length enforced server-side (not just client-side)?

**Sessions (PHP):**
- `session_regenerate_id(true)` called after login? Prevents session fixation.
- Session cookies use `HttpOnly` and `Secure` flags?

---

## 3. Sensitive Data Exposure

- Log statements include passwords, tokens, card numbers, or full emails?
- API response returns more data than the client needs (e.g. password hash in user response)?
- PII or payment data encrypted at rest?
- React front end logging auth tokens to the browser console?

PII includes: names, emails, phone numbers, payment data, health info, government IDs.

---

## 4. Broken Access Control (IDOR)

Every endpoint returning or modifying user-specific data must verify the caller owns that record.

```js
// Broken — any logged-in user can read any order
app.get('/orders/:id', auth, async (req, res) => {
  const order = await db.query('SELECT * FROM orders WHERE id = ?', [req.params.id]);
  res.json(order);
});

// Fixed — restrict to the authenticated user's orders
app.get('/orders/:id', auth, async (req, res) => {
  const order = await db.query(
    'SELECT * FROM orders WHERE id = ? AND user_id = ?',
    [req.params.id, req.user.id]
  );
  res.json(order);
});
```

- Auth middleware applied to every protected route?
- Admin-only endpoints protected by role check, not just login check?

---

## 5. Security Misconfiguration

- `NODE_ENV=production` set? Many frameworks expose stack traces without it.
- S3 buckets private by default? Public ACLs should be explicit exceptions.
- Default credentials changed (database users, Redis, managed services)?
- Debug endpoints (`/debug`, PHPInfo pages) removed before deployment?
- CORS restricted to known origins? `Access-Control-Allow-Origin: *` on authenticated APIs is dangerous.

---

## 6. Credentials and Secrets

- Diff adds any literal API key, password, token, or private key? **Rotate immediately.**
- All secrets via `process.env.VARIABLE` (Node) or `$_ENV['VARIABLE']` (PHP)?
- Long-lived secrets in AWS Secrets Manager, not plain `.env` files in production?
- `.env` in `.gitignore`? `.env.example` is the committed reference?

---

## 7. Insecure Deserialization

- `JSON.parse()` runs on untrusted data without schema validation (zod, joi)?
- PHP `unserialize()` processes user input? Use `json_decode()` instead.
- `eval()` or `new Function()` called on user-supplied strings? Flag for removal.

---

## 8. Dependency Vulnerabilities

After any change to `package.json`:
```bash
npm audit
```

- CRITICAL/HIGH → block the PR, fix before merging
- MODERATE → create ticket, fix within current sprint
- LOW → log, review at next sprint planning

Run `npm audit` on `master` at least once per sprint even without dependency changes.

---

## Output template

```
## Security Review
Scope: [what was reviewed]
Date: [YYYY-MM-DD]
Reviewer: [name]

### Findings
[CRITICAL] src/routes/orders.js:42 — IDOR: missing user_id filter
  Fix: add AND user_id = ? with req.user.id

[HIGH] src/auth/token.js:18 — JWT created without exp claim
  Fix: add expiresIn: '8h' to jwt.sign() options

### No issues found in
- SQL queries (all parameterised)
- Secret handling (all via process.env)
- Dependencies (npm audit clean)
```
