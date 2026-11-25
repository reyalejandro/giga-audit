# giga-audit (español)

Skill de auditoría integral para Claude Code, Codex y ChatGPT. Esta rama es la versión en español; la versión en inglés está en `main`.

Hace un barrido automático y lanza buscadores independientes por 9 dimensiones. Un verificador intenta refutar cada hallazgo. Recorre todo el alcance **como mínimo tres veces** y termina cuando las dos últimas pasadas salen limpias, con un tope de 5. Aplica la regla del mínimo (YAGNI) también a los arreglos.

## Instalar

```sh
git clone -b es https://github.com/reyalejandro/giga-audit ~/.claude/skills/giga-audit   # Claude Code
git clone -b es https://github.com/reyalejandro/giga-audit ~/.codex/skills/giga-audit    # Codex
```

Se activa sola con pedidos como «auditá esto», «buscá errores» o «¿está listo para producción?». También se puede invocar con `/giga-audit`.

## Fuentes

- `ponytail-audit` y `ponytail-review`: etiquetas de simplificación.
- `/code-review` y ultrareview de Claude Code: buscadores en paralelo, verificación y severidades.
- addy:
  - `code-review-and-quality`: 5 ejes;
  - `doubt-driven-development`: revisor con contexto nuevo;
  - `security-auditor`.
- Matt Pocock, `code-review`: estándares y especificación por separado.
- `feature-dev`: umbral de confianza de 80.
- `verification-before-completion`: nada está listo sin evidencia.
- [Code review en Anthropic](https://claude.dev/blog/solving-code-review-at-anthropic).
- [Using LLMs to secure source code](https://claude.com/resources/articles/using-llms-to-secure-source-code).
- [Adversarial Review](https://arxiv.org/html/2608.18167) y [Refute-or-Promote](https://arxiv.org/pdf/2604.19049v1).
- [Google eng-practices](https://google.github.io/eng-practices/review/reviewer/looking-for.html), OWASP ASVS 5.0 y [OWASP Webhook Security](https://cheatsheetseries.owasp.org/cheatsheets/Webhook_Security_Cheat_Sheet.html).
