# Engineering decisions and problems solved

[← Project overview](../README.md) · [Architecture](architecture.md) · [Deployment history](delivery-and-history.md)

## 1. Preserve a working site during a hosting migration

**Problem.** The clinic already had a Docker-hosted website and backend. Adapting that runtime directly could have mixed local and cloud assumptions or disturbed the working service.

**Decision.** Create a separate Vercel adaptation while preserving the original repository and containers. Keep the frontend and content consistent, replace only the backend assumptions that do not survive serverless execution, and validate on the temporary hosting address before changing DNS.

**Result.** The cloud adaptation could be configured and tested independently. The original deployment remained available as a fallback. The tradeoff was maintaining two deployment descriptions during the transition.

## 2. Replace process-local protection with shared state

**Problem.** In-memory duplicate tracking and request counters belong to one process. Multiple function instances would each have their own view, and a generated per-process secret would invalidate tokens when a different worker handled the submission.

**Decision.** Use a stable signing secret and atomic Redis REST operations. Put counters, protected client identifiers and consumed-token claims in a shared store with expiry.

**Result.** Separate workers can validate the same token format and enforce the same limits. The tradeoff is a dependency on Redis availability; when that protection cannot run, the handler rejects the request and exposes contact alternatives.

## 3. Treat an ambiguous SMTP result carefully

**Problem.** A network timeout can happen after the mail server has accepted an email. Automatically releasing a token or retrying may send a duplicate appointment request.

**Decision.** Claim the token before sending and retain the claim even after an ambiguous SMTP failure. Report that sending could not be confirmed, with telephone/email alternatives.

**Result.** A repeated submission with the same token cannot trigger another email. This favors duplicate prevention over automatic retry. SMTP acceptance is still distinct from inbox arrival and clinic confirmation.

## 4. Prevent preview deployments from sending mail

**Problem.** Preview environments may be created frequently, and environment settings can be copied incorrectly. A preview should not accidentally send real appointment emails.

**Decision.** Refuse real delivery on Vercel Preview and Development deployments even if SMTP values are present. Require enabled booking, complete configuration, exact origins and a working shared store in production.

**Result.** The production enablement path is explicit and can be tested separately from content previews.

## 5. Resolve Python function detection

**Observed issue.** The first cloud deployment did not recognize a Python entrypoint that only imported a handler under an alias.

**Change.** Define an explicit top-level `handler` class in each entrypoint, inheriting the shared implementation.

**Verification.** The deployment builder recognized both Python functions and the static site. The shared request logic remained in one place rather than being copied into each function.

## 6. Handle Marketplace Redis configuration accurately

**Observed issue.** The Vercel Marketplace integration supplied `KV_REST_API_URL` and `KV_REST_API_TOKEN`, while the application originally expected the `UPSTASH_REDIS_REST_*` names.

**Change.** Accept either complete REST credential pair. Never combine a URL from one pair with a token from another. Ignore read-only tokens and TCP connection URLs for write operations.

**Verification.** Tests cover supported aliases and incomplete-pair rejection. This avoided unnecessary manual renaming or adopting unrelated framework snippets.

## 7. Diagnose configuration without exposing it

**Observed issue.** Booking remained unavailable after environment setup, but the public status response did not explain which required setting was missing.

**Change.** Return safe diagnostic codes such as a disabled booking flag, without returning environment values or upstream error bodies.

**Result.** Deployment configuration could be corrected without asking for secrets in chat or making them part of a public error response.

## 8. Match the custom domain to the request origin

**Observed issue.** The temporary Vercel hostname passed availability checks, while the clinic hostname returned a forbidden response after migration.

**Change.** Update the explicit allowed origins and canonical site setting for the custom domain, then redeploy.

**Verification.** Availability was checked again on both hostnames. This was a configuration correction, not evidence that every possible end-to-end booking scenario had been retested.

## 9. Keep bilingual and mobile behavior part of the implementation

The design keeps the page structure shared between languages, translates form feedback and WhatsApp messages, and preserves the selected page when switching language. Phone-sized screens receive large contact targets and a fixed contact bar with reserved space below the content.

Responsive checks include 320-pixel widths, where long text, tariffs and navigation can reveal issues hidden at common desktop sizes. Form controls use readable mobile input sizing, and telephone/email alternatives remain useful when scripting or backend availability is limited.

## Remaining tradeoffs

- The system emails requests; it has no appointment-slot inventory or automatic booking confirmation.
- An SMTP timeout is reported conservatively rather than silently retried.
- The fallback deployment creates operational overhead until deliberately retired.
- Search indexing and the online video service have their own readiness decisions; they are not activated simply because the site is deployed.
