# Audit dimensions: what to look at

Each finder gets only its own dimension. These questions are not a checklist: they point to where bugs usually hide. The project's written rules override anything here.

1. [Logic, data and spec](#1-logic-data-and-spec)
2. [Security](#2-security)
3. [Operations and reliability](#3-operations-and-reliability)
4. [Performance](#4-performance)
5. [Tests](#5-tests)
6. [Configuration and variables](#6-configuration-and-variables)
7. [Simplification](#7-simplification)
8. [UI and accessibility](#8-ui-and-accessibility)
9. [Docs and consistency](#9-docs-and-consistency)

## 1. Logic, data and spec
- **Does it do what was asked?**
  - spec requirements that are missing or partial;
  - behavior nobody asked for;
  - requirements that look done but are wrong.

  Quote the spec line.
- **Edges:**
  - empty, null or zero;
  - minimum and maximum;
  - empty and single-item lists;
  - dates and time zones;
  - money rounding (integers or exact decimals, never `float`);
  - units: grams or kilos, cm or m, with or without tax.
- **Error paths, not only the happy one:** what if the external API is slow, returns 500 or returns something unexpected?
- **States and transitions:** can an impossible state be reached? Can an order be paid twice or shipped unpaid?
- **Concurrency:**
  - two orders for the last item;
  - races between the payment webhook and the checkout return;
  - locks and their release.
- **Idempotency:** retries and repeated notifications must not duplicate effects (charges, stock, emails, invoices).
- **Money and stock:**
  - the server decides price, stock and promotions, never the client;
  - the checkout return doesn't prove payment: verify it with the provider.
- **Data:**
  - reversible migrations;
  - constraints in the database (unique, not null, foreign keys), not only in the app;
  - soft vs hard delete.
- **New logic stuffed into an unrelated flow:** an `if` added where it doesn't belong is a design smell, not a detail.

## 2. Security
References: OWASP ASVS 5.0, OWASP Top 10 2025, OWASP Secure Code Review and Webhook Security cheat sheets.
- **Input:**
  - validation at the trust boundary;
  - injection (SQL, NoSQL, shell, templates);
  - XSS (output encoding);
  - file uploads (type, size, content);
  - redirects only to an allowlist;
  - SSRF on user-supplied URLs.
- **Authentication and authorization:**
  - checks on every protected route;
  - IDOR (can I see someone else's order by changing the id?);
  - login rate limiting;
  - `httpOnly`, `secure` and `sameSite` sessions;
  - single-use expiring tokens;
  - 2FA where it matters.
- **Secrets:**
  - out of code, Git, logs, the browser and build artifacts;
  - file mode 600 or directory 700;
  - one secret per integration;
  - rotation possible;
  - `.env` never published.
- **Webhooks:**
  - HMAC signature checked in constant time before processing;
  - timestamp and event ID against replays;
  - idempotent processing;
  - `POST` only;
  - generic errors (400 malformed, 401 bad signature);
  - size limit.
- **Infrastructure:**
  - headers (CSP, HSTS, X-Frame-Options);
  - CORS limited to the real origins;
  - error messages without stack traces;
  - least privilege for service users and containers (`cap_drop`, `no-new-privileges`, non-root);
  - only the ports that are needed;
  - admin panel not public.
- **Networks:**
  - databases with no internet egress and no public ingress;
  - services that call external APIs do have egress, otherwise the integration fails silently.
- **Dependencies:** known vulnerabilities, trusted sources, pinned versions.
- **Personal data:** only what is needed, kept out of logs, encrypted backups, defined retention.
- **LLMs, if any:**
  - model output is untrusted data (never into `eval`, SQL, shell or `innerHTML`);
  - the system prompt is not a security boundary;
  - tool permissions kept narrow;
  - token and recursion limits.

## 3. Operations and reliability
- **Backups:**
  - do they exist, and how often?
  - is there an off-server copy?
  - are they encrypted?
  - **has a restore been tested?** A backup never restored doesn't count;
  - do old ones get pruned?
  - do failures alert someone?
- **Alerts:**
  - does anyone find out when the service goes down, the disk fills, a backup fails, a certificate expires or a scheduled job stops?
  - does the alert reach a human, and was that tested?
- **Rollback:** how do you go back to the previous version, and how long does it take? Has it been rehearsed?
- **Health:**
  - real health checks;
  - automatic restart;
  - memory limits;
  - graceful shutdown;
  - timeouts on external calls;
  - retries with backoff.
- **Deployment:**
  - reproducible;
  - migrations before replicas start;
  - secrets injected at runtime;
  - the same artifact in staging and production;
  - no hidden manual steps.
- **Resources:** disk, memory, CPU, free-tier limits, domain/certificate/plan expirations.
- **Observability:**
  - errors sent to Sentry or similar: confirm they actually arrive (a wrong DSN fails silently);
  - logs with context and without secrets.
- **Leftovers:** processes, containers, volumes, images, files and packages that are no longer used.

## 4. Performance
- N+1 queries, missing indexes, over-fetching, no pagination.
- Heavy work on the hot path or in the request thread; what could run in the background.
- **Frontend:**
  - bundle size;
  - images without dimensions or a modern format;
  - LCP, CLS and INP;
  - unnecessary re-renders;
  - render-blocking fonts.
- **Caches:** are they invalidated correctly? A stale cache is a logic bug.
- Measure before proposing: an optimization without numbers is an opinion.

## 5. Tests
- Is there a test for every critical behavior (money, stock, authorization, webhooks)?
- Does it test behavior or implementation? Would it fail if the code broke? Break the code for a moment and watch it fail.
- Does it cover edges and error paths?
- Do tests use fake data or providers where appropriate, and never touch production?
- Are there tests that always pass, or that are skipped silently?

## 6. Configuration and variables
- **Every environment variable justified:**
  - what code reads it?
  - what happens if it's missing?
  - if its value equals what the code assumes when absent, it's unnecessary;
  - if the code treats absence differently (`!== 'false'`), it stays.
- **Non-secrets out of the secrets manager:** URLs, public IDs and constants are versioned.
- **Each secret in one place:** no URLs repeating passwords, no copies across files.
- **Values that look valid but aren't:** a DSN, key or ID with the right shape but pointing to the wrong project or account. Check against the provider.
- **Flags:** off-by-default flags are not declared; every flag that's on has an owner and a reason.
- **Compose and CI:**
  - pinned images;
  - nothing downloaded without verification;
  - permissions;
  - minimal networks and ports;
  - unused profiles or services.
- **Misleading names:** "staging" in production, "temporary" for months, "test" defaults in a production file.

## 7. Simplification
- **Tags,** one line per finding: `<tag>: <what to cut>. <replacement>. [path]`, ending with `net: -N lines, -M deps`.
  - `delete:` dead or speculative code; replaced by nothing;
  - `stdlib:` reimplements the standard library; name the function;
  - `native:` the platform already does it; name the feature;
  - `yagni:` an abstraction with one use, or config nobody sets;
  - `shrink:` the same logic in fewer lines; show the short form.
- **What to look for:**
  - dependencies the platform already ships;
  - interfaces with a single implementation;
  - factories with a single product;
  - wrappers that only delegate;
  - files that export one thing;
  - dead flags and config;
  - reimplemented stdlib;
  - code "for later".
- **Fowler smells** (always judgment, never a hard rule; the repo's standard wins):
  - mysterious name;
  - duplicated code;
  - feature envy;
  - data clumps;
  - primitive obsession;
  - repeated switches;
  - shotgun surgery;
  - divergent change;
  - speculative generality;
  - message chains;
  - middle man;
  - refused bequest.
- **Does the refactor reduce complexity or just move it?** Count the concepts someone must hold in their head: if the count doesn't drop, it isn't simpler.
- Don't flag the only test or minimal self-check for deletion: it's the minimum, not fat.

## 8. UI and accessibility
- Main flows at 390 and 1440 px, with no horizontal overflow.
- Keyboard: visible focus, logical order, everything reachable.
- Contrast, alt text, form labels, clear error messages.
- States: empty, loading, error, out of stock.
- Long real data (names, large prices) doesn't break the layout.
- Browser console clean; no unexpected 4xx/5xx requests.
- For visual design quality, hand off to the project's design skill.

## 9. Docs and consistency
- Do the handoff, operations and deployment docs describe what the system does **today**? Every mismatch is a finding: the next person or agent will act on the old version.
- Are there instructions pointing to paths, commands or services that no longer exist?
- Are the rules in `AGENTS.md` and `CLAUDE.md` followed in the audited scope?
- Do the code or config contradict written decisions?
