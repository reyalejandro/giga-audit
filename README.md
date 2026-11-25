# giga-audit

Skill para Claude Code y Codex que hace auditorías integrales con evidencia. Hace un barrido automático, usa buscadores por 9 dimensiones y un verificador independiente que intenta refutar cada hallazgo. Recorre todo el alcance **como mínimo tres veces** y termina cuando dos pasadas seguidas salen limpias, con un tope de 5. Aplica la regla del mínimo (Ponytail / YAGNI) a todo, incluidos los arreglos.

## Instalar

```sh
git clone https://github.com/reyalejandro/giga-audit ~/.claude/skills/giga-audit   # Claude Code
git clone https://github.com/reyalejandro/giga-audit ~/.codex/skills/giga-audit    # Codex
```

Se activa sola con pedidos como «auditá esto», «buscá errores» o «¿está listo para producción?». También se invoca con `/giga-audit`.
