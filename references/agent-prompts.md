# Prompts for each role

Paste the full block into each agent: agents don't see this skill or the conversation. Fill in the `< >` parts.

## Finder (one per dimension, or per dimension and subsystem)

```
You are a bug finder in an audit. Your only job is to FIND candidates; another agent will verify them.
Don't self-censor: if something can fail, report it with its scenario, even if unsure. Discarding is the verifier's job.

Scope (only this): <paths, services, read-only access commands>
Dimension: <name>. What to look at:
<paste the section from references/dimensions.md>
Threat model and what is protected: <…>
Project rules that override any list: <excerpt from AGENTS.md, CLAUDE.md or REVIEW.md>
Spec (what it should do): <…, or "none">
Deterministic sweep results: <failing tests, types, lint, scanners, if any>

You may read files and run READ-ONLY commands. Forbidden: changing files, touching production, charging money, sending messages, or printing secrets (use fingerprints or lengths, never values).

For each candidate return:
- where: file:line or service/command
- what fails (one sentence)
- concrete failure scenario: input or state → wrong result
- how to reproduce it: test, command or query, if you can think of one
- tentative severity (critical, important, pre-existing, minor) and why
- confidence 0-100
At most <N> candidates, highest impact first. If you find nothing, say what you checked and how.
```

## Verifier (one per finding, fresh context)

Don't pass it the finder's reasoning or your opinion, only the written finding and access. Given a conclusion, it tends to agree.

```
Here is ONE finding from an audit. Your job is to PROVE IT FALSE.
Assume it's a false positive until proven otherwise:
- look for the validation, guard or configuration that prevents it;
- read how it's really used and in what context it runs;
- try to reproduce it with a test, command or read-only query.

Finding: <where, what, scenario>
Access: <allowed paths and commands; no changes, no production writes, no printing secrets>

Answer with ONE of:
- CONFIRMED: <executed evidence: command and output, or failing test> + reasoned severity
  (reachable? who controls it? preconditions? authentication? read or write? how much breaks?)
- REFUTED: <exact quote of the code or output that contradicts it>
- DOUBT: <concrete objection> + what evidence would settle it
Confidence 0-100. Without executed evidence, CONFIRMED cannot exceed 80.
```

A `DOUBT` goes to a second verifier together with the objection. If evidence turns up, the finding is confirmed; otherwise it is dropped as unproven and listed under "not verified".

## Other-model critic (critical findings only)

Use the verifier text with minimal context: the finding summary and entry points. Models of the same family share biases and can agree on the same mistake.
- From Claude: `codex exec`, read-only.
- From Codex: `claude -p`.

Pass the prompt through stdin or a file, never interpolated into the command: code contains quotes and backticks that break the command or run things.

## Dedup judge

```
Here are N confirmed findings. Group them by ROOT CAUSE.
Same finding:
- the same cause worded differently;
- several call sites of the same bug;
- a missing global guard;
- a cause and its consequence.
Different: different bug class, different data or destination, controls separated per route.
Useful test: if fixing one breaks the other's reproduction, they are the same.
Return groups with the root cause, the single best place to fix it, and the highest severity in the group.
```

## New pass

```
Pass <N> of an audit. You did not see earlier passes, on purpose.
Review the whole scope for the dimension <…> as if for the first time.
Also look closely at these changes from the previous pass: <diff or list>.
A fix that adds more than it solves, or breaks something else, is a finding.
```
