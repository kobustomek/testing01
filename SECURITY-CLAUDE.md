# SECURITY-CLAUDE.md

## Purpose

This file defines **mandatory security rules** for Claude Code when generating or modifying this project.

Project context:
- stack: **Next.js only**
- architecture: **App Router**
- server logic: **Route Handlers, Server Actions, server-side services**
- app type: internal/business web app for entering simple data
- scale: small user base
- priority: **simple architecture with strong server-side security**

---

## Core rules

1. **Treat every Route Handler and Server Action as a public server entrypoint.**
2. **Never trust client-side validation.**
3. **Never put secrets in source code.**
4. **Never put authorization logic only in the client/UI layer.**
5. **Always validate input on the server.**
6. **Always check authentication before protected operations.**
7. **Always check authorization before read/write/update/delete operations.**
8. **Return only the minimum data needed by the client.**
9. **Do not expose stack traces, SQL errors, tokens, secrets, or internal implementation details to users.**
10. **Assume all user input is untrusted.**

---

## Required implementation order for any protected operation

For every mutation or protected read:

1. identify user
2. verify authentication
3. verify authorization
4. validate input
5. execute business logic
6. log meaningful security-relevant event if needed
7. return minimal safe response

Do not skip any step.

---

## Authentication rules

Claude Code must:
- implement authentication for all protected areas,
- use secure session handling,
- prefer server-side session checks,
- ensure cookies are configured securely in production.

Do not:
- rely only on client state to determine whether a user is logged in,
- trust hidden form fields or client-provided role information.

---

## Authorization rules

Claude Code must:
- enforce authorization on the server,
- check permissions per action,
- check ownership or role where relevant,
- repeat authorization checks even if the UI already hides an action.

Do not:
- assume that hiding a button is security,
- allow update/delete operations without explicit permission checks.

---

## Validation rules

Claude Code must validate on the server:
- required fields,
- type and format,
- length limits,
- numeric/date ranges,
- allowed enum values,
- unexpected extra fields.

Do not:
- accept raw request payloads without validation,
- trust browser-level validation as protection.

---

## Data access rules

Claude Code must:
- access the database only on the server,
- keep database logic out of client components,
- select only necessary fields,
- minimize returned payloads,
- avoid exposing internal identifiers or admin-only fields unless needed.

Do not:
- leak full records when only part of the data is required,
- expose database credentials or query details.

---

## Server Actions rules

Claude Code must treat Server Actions like public POST-capable endpoints.

For every Server Action:
- verify user,
- verify permissions,
- validate input,
- avoid returning sensitive data.

Do not:
- assume the action can only be triggered from the intended UI,
- trust client-controlled values for privilege-sensitive fields.

---

## Route Handlers rules

For every Route Handler:
- verify authentication if route is protected,
- verify authorization,
- validate params, query, and body,
- return safe error responses,
- avoid verbose internal error leakage.

Do not:
- expose stack traces,
- return raw infrastructure errors,
- leave mutation routes unprotected.

---

## Secrets and environment variables

Claude Code must:
- keep secrets in environment variables only,
- keep `.env*` files out of version control,
- use `NEXT_PUBLIC_*` only for values safe to expose publicly.

Do not:
- place secrets in code,
- place server secrets in `NEXT_PUBLIC_*`,
- print secrets in logs.

Examples of server-only secrets:
- database URLs
- session secrets
- API private keys
- service credentials

---

## Client/server boundary rules

Claude Code must:
- keep sensitive logic on the server,
- use client components only for UI/interactivity,
- avoid moving auth or permission decisions to the client.

Do not:
- query the database from client code,
- expose privileged business logic to the browser,
- assume a client component is a safe place for access control.

---

## XSS and rendering rules

Claude Code must:
- rely on React safe rendering by default,
- avoid `dangerouslySetInnerHTML`,
- sanitize any HTML if rendering HTML is unavoidable.

Do not:
- render unsanitized user content as HTML,
- embed unnecessary third-party scripts.

---

## Error handling rules

Claude Code must:
- return user-safe errors,
- log technical details only on the server,
- keep validation errors clear but not overly revealing.

Do not:
- expose stack traces,
- expose SQL statements,
- expose file paths,
- expose tokens or secrets.

---

## Logging rules

Claude Code should log:
- sign-in attempts,
- failed auth events,
- permission denials,
- create/update/delete actions where useful,
- suspicious validation failures,
- security-relevant operational errors.

Do not log:
- passwords,
- full tokens,
- secrets,
- unnecessary personal data.

---

## Abuse protection rules

Claude Code should add protection for:
- login attempts,
- password reset flows if they exist,
- sensitive mutation endpoints,
- import/export endpoints if they exist.

At minimum:
- rate limit or throttle login attempts,
- record repeated failures or abuse signals.

---

## Security headers

Claude Code should configure:
- `Content-Security-Policy`
- `Referrer-Policy`
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY` or equivalent CSP `frame-ancestors`
- `Permissions-Policy`

Also:
- disable `x-powered-by`,
- require HTTPS in production,
- use secure cookies in production.

---

## Dependency rules

Claude Code must:
- prefer mature, maintained libraries,
- avoid unnecessary dependencies,
- avoid obscure packages for trivial tasks,
- keep security-sensitive dependencies current.

Do not:
- add packages without a clear reason,
- solve simple problems with risky dependencies.

---

## Review checklist for generated code

Before finalizing server-related code, verify:

- [ ] Is authentication enforced where needed?
- [ ] Is authorization enforced on the server?
- [ ] Is input validated on the server?
- [ ] Are returned fields minimized?
- [ ] Are secrets protected?
- [ ] Are errors safe for users?
- [ ] Is sensitive logic server-only?
- [ ] Are mutation endpoints protected?
- [ ] Are Server Actions treated as public entrypoints?
- [ ] Are environment variables split correctly between server-only and public?

---

## Architectural preference for this project

Claude Code should prefer:

- **Next.js only**
- **App Router**
- **simple server-side architecture**
- **Route Handlers and/or Server Actions**
- **clear separation of UI, auth, validation, permissions, and data access**
- **minimal operational complexity**

Do not introduce a separate backend unless there is a clear architectural need.

---

## Default folder responsibility model

Recommended structure:

```text
app/
  api/
  ...
lib/
  auth/
  db/
  permissions/
  validation/
```

Responsibilities:
- `app/api/**` -> request handling only
- `lib/auth/**` -> identity/session logic
- `lib/permissions/**` -> authorization rules
- `lib/validation/**` -> input validation schemas/functions
- `lib/db/**` -> database access only

Keep business and security logic out of client components.

---

## Final instruction

When in doubt, Claude Code must choose:
- the simpler design,
- the stricter server-side check,
- the smaller data exposure,
- the safer default.
