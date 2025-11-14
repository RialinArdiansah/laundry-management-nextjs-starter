# Security Guidelines for Laundry Management System

**Purpose:** This document defines security requirements and best practices for the web-based Laundry Management System, ensuring robust protection across authentication, data handling, infrastructure, and development workflows.

---

## 1. Security by Design

- Embed security at every stage: design, implementation, testing, and deployment.
- Leverage Next.js App Router and Server Actions for clear separation of concerns and minimized attack surface.
- Conduct threat modeling for key flows: login, order entry, status updates, and reporting.

---

## 2. Authentication & Access Control

### 2.1 Robust Authentication

- Use **NextAuth.js** for session-based authentication with secure, rotating session cookies.
- Enforce **role-based access control (RBAC)**:
  - Roles: `Pegawai`, `Owner`, and public (no login).
  - Define permissions server-side in middleware to restrict route access:
    - `app/dashboard/(pegawai)/…` only for `Pegawai`.
    - `app/dashboard/(owner)/…` only for `Owner`.
- Implement **Multi-Factor Authentication (MFA)** for `Owner` accounts via TOTP or SMS.

### 2.2 Password Policy & Storage

- Enforce minimum 12-character passwords with uppercase, lowercase, number, and symbol.
- Use **bcrypt** or **Argon2** (via NextAuth adapter) with per-user salts.
- Provide secure “Forgot Password” and reset flows:
  - Generate time-limited, single-use tokens.
  - Send reset links via verified email.

### 2.3 Secure Session Management

- Set cookies with `HttpOnly`, `Secure`, and `SameSite=Lax` attributes.
- Implement **idle and absolute timeouts** (e.g., 30 min idle, 8 hrs max).
- Rotate session identifiers on privilege changes (login/logout).
- Protect against session fixation by regenerating session on authentication events.

---

## 3. Input Handling & Output Encoding

### 3.1 Server-Side Validation

- Use **Zod** schemas in Server Actions to validate all form inputs:
  - `OrderForm`, `CustomerForm`, `EmployeeForm`, etc.
- Reject requests with missing or malformed data before business logic.

### 3.2 Prevent Injection Attacks

- Employ **Drizzle ORM** with parameterized queries to prevent SQL injection.
- Avoid raw query strings; if necessary, strictly sanitize/escape inputs.

### 3.3 Cross-Site Scripting (XSS) Mitigation

- Escape all dynamic content in React templates (Next.js automatically escapes by default).
- For any `dangerouslySetInnerHTML`, sanitize with a library like **DOMPurify**.
- Enforce a **Content Security Policy (CSP)**:
  ```
  Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:;
  ```

### 3.4 CSRF Protection

- Apply **anti-CSRF tokens** on all state-changing endpoints (POST, PUT, DELETE).
- Leverage NextAuth’s built-in CSRF protection or integrate `next-csrf` middleware.

### 3.5 Secure File Handling (if applicable)

- Validate file types and sizes on upload.
- Store uploads outside the webroot with randomized filenames.
- Scan files with antivirus or sandbox prior to processing.

---

## 4. Data Protection & Privacy

### 4.1 Encryption in Transit & Rest

- Enforce **HTTPS/TLS 1.2+** for all traffic (Next.js hosted behind SSL).
- Use **AES-256** or more for any at-rest encryption (e.g., database backups).

### 4.2 Secrets Management

- Store credentials (DB connection strings, API keys) in a managed vault (e.g., Vercel Environment Variables or HashiCorp Vault).
- Rotate secrets periodically and on employee role changes.

### 4.3 Minimizing PII Exposure

- Only collect and store `Pelanggan` PII necessary for business (name, contact).
- Mask sensitive fields in logs (e.g., password reset tokens, API keys).
- Comply with GDPR/CCPA: allow data export and deletion upon user request.

---

## 5. API & Service Security

### 5.1 API Endpoint Protection

- Secure all API routes under `app/api/` with NextAuth session check middleware.
- Return only required fields in JSON responses (avoid over-exposure).

### 5.2 Rate Limiting & Throttling

- Implement rate limiting on login and public status-check endpoints (e.g., 5 req/min per IP).
- Use Vercel Edge Middleware or a library like **express-rate-limit** in custom servers.

### 5.3 CORS Policy

- Restrict CORS to trusted origins (your domain only).
- Deny all wildcard origins on secure endpoints.

### 5.4 API Versioning

- Prefix critical endpoints with `/v1/`, `/v2/` to manage breaking changes securely.

---

## 6. Web Application Security Hygiene

### 6.1 Security Headers

- `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `Referrer-Policy: same-origin`

### 6.2 Secure Cookies & Local Storage

- Do not store tokens or PII in `localStorage`/`sessionStorage`.
- Use **HttpOnly** cookies for session tokens.

### 6.3 Subresource Integrity (SRI)

- Add integrity attributes to any third-party `<script>` or `<link>`.

---

## 7. Infrastructure & Configuration

### 7.1 Server & Hosting

- Harden Next.js hosting environment (Vercel): disable default telemetry in production.
- Limit exposed ports/services; rely on managed platform security.

### 7.2 Software Updates

- Keep Node.js, Next.js, Tailwind CSS, Drizzle ORM, and other dependencies up to date.
- Subscribe to GitHub Dependabot or Snyk alerts.

### 7.3 File Permissions

- Restrict write permissions on code and config files in production.

### 7.4 Disable Debug in Production

- Set `NEXT_PUBLIC_NODE_ENV=production`; disable error overlays and verbose logs.

---

## 8. Dependency Management

- Vet third-party packages (`react-to-print`, `pusher-js`, etc.) for active maintenance.
- Use **yarn.lock** or **package-lock.json** for deterministic installs.
- Integrate **SCA scans** in CI (e.g., GH Actions, Azure Pipelines).

---

## 9. Testing & CI/CD Security

- Run unit tests (Vitest) and E2E tests (Playwright) on each pull request.
- Automate migrations, seed, and test pipelines in CI to catch schema drift and vulnerabilities early.
- Restrict CI secrets to required scopes; audit pipeline logs for inadvertent secrets exposure.

---

## 10. Monitoring & Incident Response

- Implement logging with masked PII only; ship logs to a centralized SIEM.
- Set up error tracking (e.g., Sentry) with rate-limited alerts.
- Define an incident response plan: roles, communication channels, and escalation path.

---

**Conclusion:** By adopting these guidelines, the Laundry Management System will achieve a defense-in-depth posture, protecting user data, ensuring secure operations, and supporting maintainable and resilient growth.