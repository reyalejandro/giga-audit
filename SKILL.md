---
name: giga-audit
description: Auditoría integral con evidencia. Cubre lógica y datos, seguridad, operación, rendimiento, pruebas, configuración y simplificación (YAGNI). Usa buscadores independientes y un verificador que intenta refutar cada hallazgo, y recorre todo el alcance como mínimo tres veces. Usala SIEMPRE que pidan auditar, revisar a fondo, buscar errores, «¿está todo bien?», «que no quede ningún error», revisar antes de publicar, desplegar, migrar o abrir ventas, o después de un cambio grande. También ante code review, security review o ultrareview de un repositorio, servidor, VPS, Docker, configuración, variables de entorno, documentación o sitio, y ante preguntas indirectas como «¿qué falla?», «¿qué sobra?» o «¿está listo para producción?».
---

# Giga-audit

Auditar es **encontrar, demostrar y corregir**, no leer y opinar. Cada hallazgo pasa por un segundo agente que intenta probar que es falso, y sólo queda lo que resiste.

Nunca prometas «cero errores»: una revisión sin hallazgos no prueba que no los haya. Se entregan tres cosas:
- ningún error conocido sin resolver;
- lo importante, probado;
- la lista explícita de lo que no se pudo probar.

## Regla del mínimo

Aplicala a lo auditado y a cada arreglo. Va la mínima cantidad de código, configuración, variables, archivos y pasos que cumple la función pedida. Un arreglo que agrega más de lo que resuelve es un hallazgo nuevo.

- **Toda pieza justifica su función:** línea, archivo, dependencia, variable, servicio o paso. Si no la justifica, se borra.
- **Variables de entorno:**
  - una que repite el valor por defecto del código sobra;
  - lo que no es secreto va versionado, no en el gestor de secretos;
  - cada secreto vive en un solo lugar;
  - antes de quitar una bandera, mirá cómo la lee el código: si la ausencia cambia el comportamiento (`!== 'false'`), se queda.
- **Antes de escribir,** recorré estas preguntas en orden: ¿hace falta? → ¿ya existe en el repositorio? → ¿la biblioteca estándar? → ¿la plataforma? → ¿una dependencia ya instalada? → ¿una línea? → recién ahí, el mínimo que funciona.
- **Nunca se recortan:** validación en límites de confianza, manejo de errores que evita pérdida de datos, seguridad, copias de seguridad, accesibilidad y lo pedido explícitamente.

## Fase 0: alcance y vara

Escribí esto al principio del informe:

1. **Alcance:** qué entra y qué no. Si el pedido es vago, proponé uno concreto y seguí.
2. **Reglas del proyecto:**
   - qué leer: `AGENTS.md`, `CLAUDE.md`, `REVIEW.md`, `CONTRIBUTING.md`, documentación de continuidad y memoria;
   - esas reglas escritas le ganan a esta skill.
3. **Especificación:** de dónde sale «lo que debería hacer» (issue, plan o pedido). Sin especificación, ese eje se informa como no evaluado.
4. **Modelo de amenazas breve:** qué se protege (dinero, stock, datos personales, secretos, disponibilidad), quién podría atacar y por dónde entra. Calibra qué es un hallazgo y su severidad.
5. **Irreversibles:** producción, cobros, borrados y mensajes a terceros. Sobre esto, la auditoría sólo lee, salvo autorización.
6. **Profundidad:**
   - **rápida**, sin agentes, para un cambio chico;
   - **integral**, con agentes, por defecto cuando piden «auditoría», «todo» o «que no quede nada»;
   - las dos hacen **tres pasadas como mínimo**.

## Fase 1: barrido determinista

Las herramientas encuentran rápido y sin alucinar. Corré las que el proyecto ya tiene y guardá el primer fallo de cada una.

- **Código:**
  - pruebas, tipos, lint, build y los checks del proyecto;
  - secretos en código e historial (gitleaks o `rg` de patrones);
  - dependencias vulnerables (`npm audit`, Trivy);
  - código muerto (knip o exportaciones sin uso).
- **Servidores:**
  - contenedores, redes y volúmenes;
  - puertos abiertos;
  - permisos de archivos con secretos;
  - tareas programadas y copias (¿la última restaura?);
  - certificados y disco;
  - salida a internet de cada servicio que la necesita;
  - errores recientes en los logs.
- **Sitios:** Lighthouse, consola del navegador, enlaces rotos, 390 y 1440 px.

## Fase 2: buscadores por dimensión

Un buscador por dimensión. Si el alcance es grande, partí además por subsistema, sin superposiciones. Qué mirar en cada una está en [references/dimensiones.md](references/dimensiones.md):

1. Lógica, datos y especificación
2. Seguridad
3. Operación y confiabilidad
4. Rendimiento
5. Pruebas
6. Configuración y variables
7. Simplificación
8. Interfaz y accesibilidad, si hay interfaz
9. Documentación y coherencia

**Qué recibe cada buscador:**
- su parte del alcance;
- el modelo de amenazas y las reglas del proyecto;
- su sección de dimensiones;
- el formato de [references/prompts-agentes.md](references/prompts-agentes.md).

**Lo que nunca recibe:** las conclusiones de otros buscadores, porque el sesgo compartido hace que todos fallen igual.

**No se autocensura:** reporta todo candidato con su escenario de falla, aunque dude. Filtrar le toca al verificador.

## Fase 3: verificación adversarial

Cada candidato va a un verificador **independiente**: contexto nuevo, sólo el hallazgo escrito y acceso al sistema, nunca el razonamiento del buscador. Su mandato es **demostrar que es falso**: buscar la protección que no se vio, leer cómo se usa de verdad y reproducirlo.

- **Reproducí siempre que se pueda:** una prueba que falla, un comando, una consulta, un `curl`, una restauración. Razonar atrapa errores de lógica; sólo ejecutar atrapa los empíricos.
- **Veredicto, uno de tres:**
  - `CONFIRMADO`, con evidencia ejecutada;
  - `REFUTADO`, con la cita que lo contradice;
  - `DUDA`, con la objeción. Va a un segundo verificador: confirma o se descarta como no probado.
- **Qué entra al informe:**
  - lo `CONFIRMADO`;
  - lo `PLAUSIBLE` con confianza ≥ 80 sobre 100, junto con lo que faltó para probarlo.
- **La unanimidad no es evidencia.** En lo crítico (dinero, seguridad, datos), sumá un crítico de **otro modelo** si está disponible: Codex desde Claude, o al revés.

## Fase 4: deduplicar y clasificar

- **Agrupá por causa raíz.** Son el mismo hallazgo:
  - la misma causa contada distinto;
  - varios lugares que llaman al mismo error;
  - la falta de una protección global;
  - una causa y su consecuencia.

  Prueba útil: si arreglar uno desarma la reproducción del otro, son el mismo.
- **Severidad razonada.** Antes de asignarla, respondé:
  - ¿se alcanza?;
  - ¿quién lo controla?;
  - ¿qué precondiciones necesita?;
  - ¿exige autenticación?;
  - ¿lee o escribe?;
  - ¿cuánto rompe?

  | Severidad | Qué es |
  |---|---|
  | Crítico | Pérdida de dinero, datos o secretos; caída de producción; acceso indebido |
  | Importante | Falla real en uso normal o recorrido principal roto |
  | Preexistente | Error real que no introdujo el cambio auditado; va aparte |
  | Menor | Estilo, nombres, limpieza. Máximo 5 en el informe |

## Fase 5: corregir lo confirmado

- **Sin preguntar:** lo seguro y verificable, como errores, simplificaciones, configuración y documentación desactualizada.
- **Preguntando antes:** lo que toca producción, dinero, datos reales, borrados, secretos o una decisión del usuario (precios, proveedores, diseño). Ofrecé la opción recomendada y su riesgo.
- **Cómo corregir:**
  - en la causa raíz, donde pasan todos los llamadores, no en el síntoma;
  - si hay lógica de por medio, primero una prueba que falle con el error y pase con el arreglo;
  - diff mínimo.
- **Al terminar:** repetí el barrido determinista y la prueba del arreglo.

## Fase 6: pasadas y corte

- Cada pasada revisa **todo el alcance** con buscadores frescos, que no vieron las anteriores, y mira en especial lo que cambió la pasada previa.
- **Mínimo: tres pasadas completas**, aunque la primera no encuentre nada.
- **Corte:** desde la tercera, terminás cuando las **dos últimas** pasadas no encontraron nada crítico ni importante, nuevo y confirmado.
- **Tope:** 5 pasadas. Si a la quinta siguen apareciendo hallazgos, el problema es de diseño: decilo y proponé el cambio de fondo.
- Un error que reaparece se corrige en su origen: una regla de lint, una prueba, una línea en `AGENTS.md` o `REVIEW.md`, o una guarda compartida.

## Fase 7: informe

En el idioma del usuario, con este formato:

```
Alcance: <entra / no entra> · Profundidad: <rápida | integral> · Pasadas: <N ≥ 3> (corte: <2 últimas limpias | tope>)
Barrido determinista: <herramienta: resultado, primer fallo>

| # | Severidad | Veredicto | Dimensión | Dónde | Qué falla | Escenario de falla | Evidencia ejecutada | Estado |

Simplificación: `<delete|stdlib|native|yagni|shrink>: <qué se corta>. <reemplazo>. [ruta]` → net: -<N> líneas, -<M> dependencias, -<K> variables
Preexistentes: …
Menores (máximo 5): …

Comprobado y cómo: …
NO comprobado y por qué: …
Pendiente del usuario: <decisiones o accesos que sólo él puede dar>
Cierre: «Sin errores conocidos sin resolver en el alcance auditado» o «Quedan N abiertos».
```

Si el proyecto usa otro formato de cierre, se agrega al final.

## Ejecución según la herramienta

- **Claude Code.** Invocar esta skill autoriza los agentes que describe.
  - Con la herramienta `Workflow`, cargá `workflow-authoring` y armá un `pipeline`: buscadores → un verificador por hallazgo, apenas termina cada buscador → deduplicación → arreglos → nueva pasada.
  - Sin `Workflow`, usá `Agent`: todos los buscadores en un mensaje y después los verificadores.
  - Hasta 8 agentes por pasada. Escalá si el alcance es enorme, y avisá el costo.
  - Para un diff grande antes de fusionar, sugerí `/code-review ultra`: lo lanza el usuario y tiene costo.
- **Codex u otra herramienta sin subagentes:**
  - escribí cada hallazgo en un archivo;
  - verificalo en una instrucción nueva que reciba sólo ese archivo y el código;
  - si está `claude -p`, usalo como crítico de otro modelo.
- **Sin agentes:** hacé las mismas fases, pero indicá en el informe que la verificación no fue independiente.

## Trampas

| Trampa | Corrección |
|---|---|
| Sólo buscar qué sobra y llamarlo auditoría | La simplificación es 1 de 9 dimensiones |
| «Lo leí y está bien» | Cada «está bien» dice cómo se comprobó |
| El que construyó revisa lo suyo | Verificador independiente que intenta refutar |
| Diez agentes coinciden, entonces es verdad | Reproducir y, en lo crítico, otro modelo |
| Una sola pasada | Mínimo tres |
| Parchar el síntoma | Causa raíz, donde pasan todos los llamadores |
