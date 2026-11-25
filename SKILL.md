---
name: giga-audit
description: Evidence-based full audit. Covers logic and data, security, operations, performance, tests, configuration and simplification (YAGNI). Independent finders plus a verifier that tries to refute every finding, with at least three passes over the whole scope. ALWAYS use it when asked to audit, review thoroughly, find bugs, "is everything OK?", "make sure nothing is broken", review before publishing, deploying, migrating or launching, or after a big change. Also for code review, security review or ultrareview of a repository, server, VPS, Docker, configuration, environment variables, docs or website, and for indirect questions like "what's broken?", "what's unnecessary?" or "is this production-ready?".
---

# Giga-audit

An audit means **find, prove and fix**, not read and give opinions. Every finding goes to a second agent that tries to prove it false, and only what survives is reported.

Never promise "zero bugs": a review with no findings does not prove there are none. Deliver three things instead:
- no known bug left unresolved;
- everything important proven;
- an explicit list of what could not be verified.

## Minimum rule

It applies to the audited system and to every fix. Use the smallest amount of code, configuration, variables, files and steps that does the requested job. A fix that adds more than it solves is a new finding.

- **Every piece justifies its function:** line, file, dependency, variable, service or step. If it can't, it goes.
- **Environment variables:**
  - one that repeats the code's default is unnecessary;
  - non-secrets are versioned, not stored in the secrets manager;
  - each secret lives in exactly one place;
  - before removing a flag, check how the code reads it: if its absence changes behavior (`!== 'false'`), it stays.
- **Before writing,** go through these questions in order: is it needed? → does it already exist in the repo? → the standard library? → the platform? → an installed dependency? → one line? → only then, the minimum that works.
- **Never cut:**
  - validation at trust boundaries;
  - error handling that prevents data loss;
  - security;
  - backups;
  - accessibility;
  - anything explicitly requested.

## Phase 0: scope and bar

Write this at the top of the report:

1. **Scope:** what is in and what is out. If the request is vague, propose a concrete scope and proceed.
2. **Project rules:**
   - read `AGENTS.md`, `CLAUDE.md`, `REVIEW.md`, `CONTRIBUTING.md`, the handoff docs and memory;
   - these written rules override this skill.
3. **Spec:** where "what it should do" comes from (issue, plan or request). Without a spec, report that axis as not evaluated.
4. **Brief threat model:** what is protected (money, stock, personal data, secrets, availability), who might attack and how they get in. It sets what counts as a finding and how severe it is.
5. **Irreversible actions:** production, payments, deletions, messages to third parties. On these the audit is read-only unless authorized.
6. **Depth:**
   - **quick**, no agents, for a small change;
   - **full**, with agents, the default when asked for an "audit", "everything" or "nothing left";
   - both run **at least three passes**.

## Phase 1: deterministic sweep

Tools find things fast and without hallucinating. Run what the project already has and record the first failure of each.

- **Code:**
  - tests, types, lint, build and project checks;
  - secrets in code and history (gitleaks or `rg` patterns);
  - vulnerable dependencies (`npm audit`, Trivy);
  - dead code (knip, unused exports).
- **Servers:**
  - containers, networks and volumes;
  - open ports;
  - permissions on secret files;
  - scheduled jobs and backups (does the latest one restore?);
  - certificates and disk;
  - outbound internet for each service that needs it;
  - recent errors in the logs.
- **Websites:** Lighthouse, browser console, broken links, 390 and 1440 px.

## Phase 2: finders by dimension

Use one finder per dimension. On a large scope, also split by subsystem, with no overlap. What to look at in each one is in [references/dimensions.md](references/dimensions.md):

1. Logic, data and spec
2. Security
3. Operations and reliability
4. Performance
5. Tests
6. Configuration and variables
7. Simplification
8. UI and accessibility, if there is a UI
9. Docs and consistency

**Each finder receives:**
- its slice of the scope;
- the threat model and the project rules;
- its dimensions section;
- the format in [references/agent-prompts.md](references/agent-prompts.md).

**Never** give a finder the conclusions of other finders: shared bias makes them all fail the same way.

Finders **don't self-censor**: they report every candidate with its failure scenario, even when unsure. Filtering is the verifier's job.

## Phase 3: adversarial verification

Each candidate goes to an **independent** verifier with fresh context: only the written finding and access to the system, never the finder's reasoning. Its mandate is to **prove it false**: look for the protection that was missed, read how the code is really used and reproduce the failure.

- **Reproduce whenever possible:** a failing test, a command, a query, a `curl`, a restore. Reasoning catches logic errors; only execution catches empirical ones.
- **Verdict, one of three:**
  - `CONFIRMED`, with executed evidence;
  - `REFUTED`, with the quote that contradicts it;
  - `DOUBT`, with the objection. It goes to a second verifier, which confirms it or discards it as unproven.
- **What reaches the report:**
  - `CONFIRMED` findings;
  - `PLAUSIBLE` findings with confidence ≥ 80/100, plus what was missing to prove them.
- **Unanimity is not evidence.** On critical findings (money, security, data), add a critic from **another model** when available: Codex from Claude, or the other way around.

## Phase 4: deduplicate and rank

- **Group by root cause.** These are the same finding:
  - the same cause worded differently;
  - several call sites of the same bug;
  - a missing global guard;
  - a cause and its consequence.

  A useful test: if fixing one breaks the other's reproduction, they are the same.
- **Reasoned severity.** Before assigning it, answer:
  - is it reachable?
  - who controls the input?
  - what preconditions does it need?
  - does it require authentication?
  - does it read or write?
  - how much does it break?

  | Severity | Meaning |
  |---|---|
  | Critical | Loss of money, data or secrets; production outage; unauthorized access |
  | Important | Real failure in normal use, or a main flow broken |
  | Pre-existing | Real bug the audited change didn't introduce; reported separately |
  | Minor | Style, naming, cleanup. At most 5 in the report |

## Phase 5: fix what is confirmed

- **Without asking:** safe, verifiable fixes, such as bugs, simplifications, configuration and stale docs.
- **Ask first** for anything touching production, money, real data, deletions, secrets or a decision that belongs to the user (prices, vendors, design). Offer the recommended option and its risk.
- **How to fix:**
  - at the root cause, where every caller passes, not at the symptom;
  - when logic is involved, first a test that fails with the bug and passes with the fix;
  - minimal diff.
- **After fixing,** rerun the deterministic sweep and the fix's test.

## Phase 6: passes and stopping

- Every pass reviews **the whole scope** with fresh finders that haven't seen earlier passes, and looks closely at what the previous pass changed.
- **Minimum three full passes,** even if the first finds nothing.
- **Stop:** from the third pass on, stop when the **last two** passes found nothing new and confirmed at critical or important severity.
- **Cap:** 5 passes. If findings still appear on the fifth, the problem is design: say so and propose the structural change.
- A recurring bug gets fixed at its origin: a lint rule, a test, a line in `AGENTS.md` or `REVIEW.md`, or a shared guard.

## Phase 7: report

Write it in the user's language, in this format:

```
Scope: <in / out> · Depth: <quick | full> · Passes: <N ≥ 3> (stop: <last 2 clean | cap>)
Deterministic sweep: <tool: result, first failure>

| # | Severity | Verdict | Dimension | Where | What fails | Failure scenario | Executed evidence | Status |

Simplification: `<delete|stdlib|native|yagni|shrink>: <what to cut>. <replacement>. [path]` → net: -<N> lines, -<M> deps, -<K> variables
Pre-existing: …
Minor (max 5): …

Verified and how: …
NOT verified and why: …
Pending on the user: <decisions or access only they can give>
Closing: "No known unresolved bugs in the audited scope" or "N still open".
```

If the project uses its own closing format, add it at the end.

## Running it by tool

- **Claude Code.** Invoking this skill authorizes the agents it describes.
  - With the `Workflow` tool, load `workflow-authoring` and build a `pipeline`: finders → one verifier per finding as soon as each finder ends → dedup → fixes → next pass.
  - Without `Workflow`, use `Agent`: all finders in one message, then the verifiers.
  - Up to 8 agents per pass. Scale up on huge scopes and state the cost.
  - For a large diff before merging, suggest `/code-review ultra`: the user launches it and it costs money.
- **Codex or any tool without subagents:**
  - write each finding to a file;
  - verify it in a new instruction that gets only that file and the code;
  - if `claude -p` is available, use it as the other-model critic.
- **No agents at all:** run the same phases, and state in the report that verification was not independent.

## Traps

| Trap | Fix |
|---|---|
| Only looking for what's unnecessary and calling it an audit | Simplification is 1 of 9 dimensions |
| "I read it and it's fine" | Every "fine" says how it was verified |
| The builder reviews their own work | Independent verifier that tries to refute |
| Ten agents agree, so it's true | Reproduce it and, on critical findings, ask another model |
| A single pass | Three at least |
| Patching the symptom | Root cause, where all callers pass |
