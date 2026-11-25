# giga-audit

An evidence-based full-audit skill for Claude Code, Codex and ChatGPT (Agent Skills format). A Spanish version lives on the [`es`](https://github.com/reyalejandro/giga-audit/tree/es) branch.

What it does:
1. A deterministic sweep with tests, linters, scanners and server checks.
2. Independent finders across 9 dimensions.
3. A verifier that tries to refute every finding and reproduce it.
4. Deduplication by root cause and minimal fixes.
5. At least **three passes** over the whole scope. It stops when the last two come out clean, with a cap of 5.

## Install

```sh
git clone https://github.com/reyalejandro/giga-audit ~/.claude/skills/giga-audit   # Claude Code
git clone https://github.com/reyalejandro/giga-audit ~/.codex/skills/giga-audit    # Codex
```

- **ChatGPT:** download the repo as a ZIP, then go to Plugins → Skills → Create → Upload from your computer.
- **Claude.ai and Claude Code cloud sessions:** upload the ZIP in your claude.ai skills settings. Cloud sessions load the skills enabled on your account.

It triggers on requests like "audit this", "find bugs" or "is this production-ready?". It can also be invoked as `/giga-audit`.

## Sources

- `ponytail-audit` and `ponytail-review`: simplification tags.
- Claude Code `/code-review` and ultrareview: parallel finders, verification and severities.
- addy:
  - `code-review-and-quality`: 5 axes;
  - `doubt-driven-development`: fresh-context reviewer;
  - `security-auditor`.
- Matt Pocock's `code-review`: standards and spec as separate axes.
- `feature-dev`: confidence threshold of 80.
- `verification-before-completion`: nothing is done without evidence.
- [Code review at Anthropic](https://claude.dev/blog/solving-code-review-at-anthropic).
- [Using LLMs to secure source code](https://claude.com/resources/articles/using-llms-to-secure-source-code).
- [Adversarial Review](https://arxiv.org/html/2608.18167) and [Refute-or-Promote](https://arxiv.org/pdf/2604.19049v1).
- [Google eng-practices](https://google.github.io/eng-practices/review/reviewer/looking-for.html), OWASP ASVS 5.0 and the [OWASP Webhook Security cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/Webhook_Security_Cheat_Sheet.html).
