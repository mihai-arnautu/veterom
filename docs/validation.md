# Validation and evidence

[← Project overview](../README.md) · [Architecture](architecture.md) · [Delivery history](delivery-and-history.md)

## What was checked

| Area | Evidence | Practical limit |
| --- | --- | --- |
| Static output | Node 24 production build; 20 generated pages after retired page removal | A successful build does not prove mail or Redis availability |
| Responsive UI | 40 browser scenarios across RO/EN at 320, 390, 768 and 1440 pixels | Checks cover the selected routes and viewports |
| Gallery | Earlier browser checks decoded all 36 optimized photographs | Does not measure delivery speed on every connection |
| Tariffs | 17 cabinet rows preserved during content changes | Prices are clinic-supplied, not independently audited |
| Backend | Mocked legacy and serverless unit tests | External SMTP/Redis are replaced in these tests |
| Email delivery | Authorized technical submissions and user confirmation of receipt during implementation | Not a continuous inbox delivery monitor |
| Public deployment | HTTPS pages, custom-domain status, redirects and analytics script availability | Analytics dashboard collection was not inspected |

## Backend scenarios covered by the existing suite

- Availability routing and no-cache responses.
- Missing configuration and preview/development delivery restrictions.
- Supported Redis environment aliases and incomplete-pair rejection.
- Configuration diagnostics that do not return secret values.
- Tokens surviving worker reloads and being bound to the origin.
- Invalid hosts, origins, content types, body sizes and payloads.
- Honeypot and privacy checks.
- Concurrent duplicate submissions and shared request limits.
- Redis claim/limiter failures and fail-closed behavior.
- SMTP delivery errors and retained claims after ambiguous outcomes.
- Trusting forwarded client addresses only in the intended runtime.

The original validation and SMTP tests remain alongside the serverless tests. The suite uses intercepted SMTP and Redis calls and sends no real appointment emails.

## Recorded browser checks

The November content update was checked on Home, Contact, Services, Online Consultations and Privacy in both languages at four viewport widths: 5 routes × 2 languages × 4 widths = 40 scenarios.

Checks covered horizontal overflow, updated hours, translated online consultation content, WhatsApp messages, tariff row counts, the analytics component loading once per page and the revised privacy explanation. Earlier work also included keyboard/menu checks, no-JavaScript contact and language-switch paths, and selected automated accessibility checks.

Accessibility checks are evidence of tested behaviors, not a claim of complete WCAG certification.

## Reading the screenshots

The screenshots in this repository were captured from the live public pages using a fresh browser context. They show the visitor interface, not an administration screen or private appointment data.

| View | What it demonstrates |
| --- | --- |
| Desktop homepage | Clinic identity, imagery and main contact actions |
| Mobile homepage | Small-screen layout and mobile contact bar |
| Services | Category cards, cabinet tariffs and price-free online consultation summary |
| Appointment page | Contact alternatives and request interface |
| Gallery | Optimized photo grid |

## Current boundaries

The site handles requests for clinic follow-up, not guaranteed reservations. Opening hours are informational; the current preferred-time field is a general request input rather than an availability engine that enforces every day-specific schedule.

The online video service remains in preparation. Search exclusion remains enabled. There are no claimed conversion increases, uptime percentages, load benchmarks or search rankings in this case study.
