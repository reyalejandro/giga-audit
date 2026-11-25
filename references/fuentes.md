# Fuentes (relevadas el 25/11/2026)

Qué aporta cada una a la skill. Reconsultar si pasan meses: los métodos de revisión con agentes cambian rápido.

## Skills y agentes instalados (contenido leído)
- **ponytail-audit y ponytail-review** (Ponytail 4.9.0): etiquetas `delete`/`stdlib`/`native`/`yagni`/`shrink`, una línea por hallazgo, cierre `net: -N`. Excluye por diseño los errores, la seguridad y el rendimiento, por eso no alcanza solo.
- **ponytail-debt:** juntar los comentarios `ponytail:` en un registro y marcar los que no tienen disparador.
- **code-review y /code-review ultra** (Claude Code): niveles de esfuerzo, hallazgos con escenario de falla, severidades (importante, preexistente, menor) y veredicto confirmado o plausible. Ultra: flota de revisores en la nube y cada hallazgo reproducido; hasta 500 archivos u 8.000 líneas; entre 5 y 25 USD después de las corridas gratis. https://code.claude.com/docs/en/ultrareview
- **addy code-review-and-quality:** cinco ejes (corrección, legibilidad y simplicidad, arquitectura, seguridad, rendimiento), «¿el refactor reduce complejidad o sólo la mueve?», tamaño de archivo, primero las pruebas.
- **addy doubt-driven-development:** revisor con contexto nuevo al que se le pasa el artefacto y el contrato, nunca la conclusión; escalar a otro modelo.
- **addy security-auditor:** entradas, autenticación y autorización, datos, infraestructura, terceros y LLM.
- **Matt Pocock, code-review:** dos ejes separados (estándares y especificación) y olores de Fowler como criterio, donde el estándar del repositorio gana.
- **feature-dev, code-reviewer:** confianza de 0 a 100 y reportar sólo desde 80.
- **superpowers, verification-before-completion:** ninguna afirmación de «listo» sin evidencia recién ejecutada.

## Web
- **Cómo revisa código el equipo de Claude Code** (claude.dev, 15/10/2026): varios buscadores en paralelo y un grupo aparte que vota cada hallazgo; severidades importante, menor y preexistente; se descarta lo que es sólo estilo; tope de rondas; corregir lo recurrente en su origen (`CLAUDE.md`, `REVIEW.md`, lint). https://claude.dev/blog/solving-code-review-at-anthropic
- **Using LLMs to secure source code** (Anthropic, 27/05/2026): modelo de amenazas antes de buscar; partir el alcance; buscadores con contexto rico y prompts cortos; verificador independiente que intenta refutar y reproduce; deduplicar por causa raíz; severidad razonada antes de asignarla; errores típicos, como el buscador que se autocensura o el verificador que comparte historia. https://claude.com/resources/articles/using-llms-to-secure-source-code
- **Adversarial Review: Structured Disagreement** (arXiv 2608.18167): crítico obligado a responder AGREE, DISAGREE_EVIDENCE o DISAGREE_CONCERN contra el falso consenso; mejor F1 con tres agentes que con cinco sin restricciones. https://arxiv.org/html/2608.18167
- **Refute-or-Promote** (arXiv 2604.19049): etapas con mandato de destruir, no de converger; descarte acumulado de alrededor del 79 %; crítico de otra familia de modelos; «la unanimidad no es corrección»; la validación empírica es obligatoria. https://arxiv.org/pdf/2604.19049v1
- **Google eng-practices, qué mirar:** diseño, funcionalidad, complejidad, sobreingeniería, pruebas, nombres, comentarios, estilo, coherencia, documentación, cada línea y contexto. https://google.github.io/eng-practices/review/reviewer/looking-for.html
- **OWASP:** Secure Code Review Cheat Sheet, Webhook Security Cheat Sheet (HMAC en tiempo constante, marca de tiempo e ID, idempotencia, 400 y 401 genéricos) y ASVS 5.0. https://cheatsheetseries.owasp.org/cheatsheets/Webhook_Security_Cheat_Sheet.html
- **Production readiness** (Google SRE y listas de Cortex y Harper): cada punto con evidencia (una alerta configurada, un simulacro hecho), copias con restauración probada, vuelta atrás ensayada.
