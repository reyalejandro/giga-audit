---
name: giga-audit
description: Auditoría integral con evidencia, revisores independientes y un verificador que intenta refutar cada hallazgo. Revisa todo el alcance como mínimo tres veces y sigue hasta que dos pasadas seguidas no encuentren nada importante. Cubre lógica y datos, seguridad, operación, rendimiento, pruebas, configuración y la regla del mínimo (Ponytail ultra / YAGNI). Úsala SIEMPRE que pidan auditar, revisar a fondo, «buscar errores», «¿está todo bien?», «que no quede ningún error», revisar antes de publicar, desplegar, migrar o abrir ventas, después de un cambio grande, o cuando pidan code review, security review, ponytail audit o ultrareview de un repositorio, servidor, VPS, Docker o Coolify, configuración o variables de entorno, documentación o un sitio. También cuando la pregunta sea indirecta, como «¿qué falla?», «¿qué sobra?» o «¿está listo para producción?».
---

# Giga-audit: auditoría integral

Una auditoría útil no es leer código y opinar: es **encontrar, demostrar y corregir**, con un segundo par de ojos que intenta probar que cada hallazgo es falso. Esta skill junta lo mejor de `ponytail-audit` y `ponytail-review` (qué sobra), `code-review` y `ultrareview` (muchos buscadores, verificación independiente, severidades), la revisión de 5 ejes de addy, la de 2 ejes de Matt Pocock (estándares y especificación), `feature-dev` (umbral de confianza), `doubt-driven-development` (revisor sin contexto previo) y la investigación del 25/11/2026 ([fuentes](references/fuentes.md)).

**Lo que nunca se promete:** «cero errores». Ni Anthropic lo promete con su revisión de varios agentes: una revisión sin hallazgos no prueba que el código esté libre de errores. Lo que sí se entrega es: ningún error conocido sin resolver, todo lo importante probado y la lista explícita de lo que no se pudo probar.

## Regla del mínimo (vale en todo, también al corregir)

El dueño la fijó como regla principal: la mínima cantidad posible y óptima de código, configuración, variables, archivos y pasos. Nada que no se haya pedido y no cumpla una función. Es una dimensión más de la auditoría y también el freno de cada arreglo: un arreglo que agrega más de lo que resuelve es un hallazgo nuevo.

- **Cada línea, archivo, dependencia, variable de entorno, servicio o paso tiene que justificar su función.** Si no la tiene, se borra.
- **Variables de entorno:**
  - una variable que repite el valor por defecto del código no va;
  - lo que no es secreto va versionado (repositorio, Compose), no en el gestor de secretos;
  - cada secreto vive en un solo lugar (una URL que repite una contraseña se arma desde esa contraseña);
  - antes de quitar una bandera, comprobar en el código cómo se lee: si la ausencia cambia el comportamiento (`!== 'false'`), se queda.
- **Escalera de Ponytail** antes de escribir: ¿hace falta? → ¿ya existe en el repositorio? → ¿la biblioteca estándar? → ¿la plataforma? → ¿una dependencia ya instalada? → ¿una línea? → recién ahí, el mínimo que funciona.
- **Lo que nunca se recorta:** validación en límites de confianza, manejo de errores que evita pérdida de datos, seguridad, copias de seguridad, accesibilidad y lo pedido explícitamente.
- **Etiquetas de simplificación:**
  - `delete:` código muerto o especulativo;
  - `stdlib:` reimplementa la biblioteca estándar (nombrar la función);
  - `native:` la plataforma ya lo hace (nombrar la función);
  - `yagni:` abstracción con un solo uso o configuración que nadie toca;
  - `shrink:` misma lógica en menos líneas (mostrar la versión corta).

## Antes de empezar: alcance y vara (fase 0)

Sin alcance cerrado la auditoría no termina nunca, o termina sin haber mirado lo importante. Escribí estas líneas al principio del informe:

1. **Qué entra y qué no.** Repositorios, carpetas, servicios, servidor, documentación. Si el pedido es vago, proponé un alcance concreto y seguí; no frenes por preguntar.
2. **Reglas del proyecto.** Leé `AGENTS.md`, `CLAUDE.md`, `REVIEW.md`, `CONTRIBUTING.md`, la guía de continuidad y la memoria. Las reglas escritas del dueño le ganan a esta skill y a cualquier lista genérica.
3. **Especificación.** De dónde sale «lo que debería hacer»: issue, plan, tarjeta o pedido del dueño. Sin especificación, el eje «¿hace lo pedido?» se informa como no evaluado.
4. **Modelo de amenazas breve.** Qué se protege (dinero, stock, datos personales, secretos, disponibilidad), quién podría atacar, por dónde entra y qué es de confianza. Define qué cuenta como hallazgo y calibra la severidad.
5. **Riesgos irreversibles.** Producción, cobros reales, borrado de datos, mensajes a clientes. Para estos, las acciones de la auditoría son de sólo lectura salvo autorización.
6. **Profundidad.** Las dos hacen **tres pasadas como mínimo**: una sola pasada deja errores que la segunda y la tercera encuentran. Rápida, sin agentes, para un cambio chico; **integral** (por defecto cuando se pide «auditoría», «todo» o «que no quede nada»), con agentes y la regla de corte completa.

## El proceso (fases 1 a 7)

### Fase 1: barrido determinista primero

Las herramientas encuentran rápido y sin alucinar lo que un modelo encuentra caro. Corré las que el proyecto ya tenga, sin instalar nada nuevo salvo que haga falta y lo justifiques. Guardá el primer fallo de cada una.

- Pruebas, tipos, lint, build y los checks del proyecto (por ejemplo `npm run check`).
- Secretos en el código y el historial (gitleaks o `rg` de patrones); dependencias vulnerables (`npm audit`, Trivy); código muerto (knip, ts-prune o `rg` de exportaciones sin uso).
- En servidores:
  - estado de contenedores, redes y volúmenes;
  - puertos abiertos;
  - permisos de archivos con secretos;
  - temporizadores y copias (la última, ¿restaura?);
  - certificados;
  - espacio en disco;
  - salida a internet de cada servicio que la necesita;
  - logs de errores recientes.
- En sitios: Lighthouse o el equivalente, consola del navegador, enlaces rotos, 390 y 1440 px.

### Fase 2: buscadores por dimensión (descubrimiento)

Dividí el alcance para que cada buscador mire poco y a fondo. Usá un buscador por dimensión y, si el alcance es grande, partí además por subsistema, sin superposiciones. Las dimensiones y sus listas completas están en [references/dimensiones.md](references/dimensiones.md):

1. Lógica, datos y especificación (dinero, stock, pedidos, estados, concurrencia, idempotencia).
2. Seguridad (OWASP ASVS 5 y Top 10 2025, webhooks, secretos, autorización, LLM si hay).
3. Operación y confiabilidad (copias probadas, avisos, vuelta atrás, salud, recursos, despliegue).
4. Rendimiento (consultas, N+1, paginación, bundles, Core Web Vitals).
5. Pruebas (¿existen?, ¿prueban comportamiento?, ¿detectarían una regresión?).
6. Configuración y variables (regla del mínimo aplicada a entornos, Compose, CI).
7. Simplificación (Ponytail y olores de Fowler).
8. Interfaz y accesibilidad, si hay interfaz.
9. Documentación y coherencia (¿la documentación dice lo que el sistema hace hoy?).

Cada buscador recibe: el alcance de su parte, el modelo de amenazas, las reglas del proyecto, su lista de la dimensión y el formato de hallazgo. **No recibe** las conclusiones de otros buscadores, porque el sesgo compartido hace que todos fallen igual. Los prompts están en [references/prompts-agentes.md](references/prompts-agentes.md).

El buscador **no se autocensura**: reporta todo candidato con su escenario de falla, aunque dude. Filtrar es trabajo del verificador. Un buscador que verifica lo suyo descarta hallazgos reales.

### Fase 3: verificación adversarial (refutar o promover)

Cada candidato pasa por un verificador **independiente**: contexto nuevo, sin el razonamiento del buscador, sólo el hallazgo escrito y acceso al código o al sistema. Su mandato es **intentar demostrar que es falso**:
- buscar la protección que el buscador no vio;
- leer cómo se usa de verdad;
- reproducirlo.

- **Reproducir siempre que se pueda:** una prueba que falla, un comando, una consulta, un `curl` o una restauración. La revisión adversarial atrapa errores de razonamiento, pero no errores empíricos: sólo ejecutar los atrapa.
- **Veredicto, con una de tres respuestas** (evita el falso consenso):
  - `CONFIRMADO` (con la evidencia ejecutada);
  - `REFUTADO: <cita de código o salida que lo contradice>`;
  - `DUDA: <objeción>`. Esta obliga a una segunda mirada: o aparece evidencia que lo confirma o se descarta.
- **Confianza:** sólo entra al informe como hallazgo lo `CONFIRMADO`, o lo plausible con confianza ≥ 80 sobre 100, marcado `PLAUSIBLE` y con lo que faltó para probarlo.
- **Unanimidad no es verdad.** Varios agentes del mismo modelo pueden equivocarse igual. Para lo crítico (dinero, seguridad, datos) sumá un crítico de **otro modelo** si está disponible (por ejemplo Codex desde Claude, o al revés) con contexto mínimo.

### Fase 4: deduplicar y clasificar

- **Agrupá por causa raíz,** no por síntoma. Son el mismo hallazgo: misma causa con distinta redacción, varios lugares que llaman al mismo error, falta de una protección global, o una causa y su consecuencia. Prueba útil: si arreglar uno desarma la reproducción del otro, son el mismo.
- **Severidad,** razonada antes de asignarla. Escribí la respuesta a cada pregunta: ¿se alcanza?, ¿quién lo controla?, ¿qué precondiciones hacen falta?, ¿pide autenticación?, ¿lee o escribe?, ¿cuánto rompe?
  - **Crítico:** pérdida de dinero, datos o secretos; caída de producción; acceso indebido.
  - **Importante:** falla real que se va a dar en uso normal, o un recorrido principal roto.
  - **Preexistente:** error real que no introdujo el cambio auditado; se informa aparte.
  - **Menor:** estilo, nombres, limpieza. Con tope: no más de 5 en el informe, porque el ruido entierra lo importante.

### Fase 5: corregir lo confirmado

- **Se corrige sin preguntar** lo seguro y verificable: errores, simplificaciones, configuración y documentación desactualizada. Es la regla del dueño de aplicar toda mejora sin efectos adversos.
- **Se pregunta antes** lo que toca producción, dinero, datos reales, borrados, secretos o una decisión del dueño (precios, proveedores, diseño). En la pregunta van la opción recomendada y su riesgo.
- **Cómo corregir:**
  - causa raíz, en el lugar por donde pasan todos los llamadores, no en el síntoma;
  - con lógica de por medio, primero una prueba que falla con el error y pasa con el arreglo;
  - cada arreglo pasa por la regla del mínimo: diff chico, sin capas nuevas.
- **Después de corregir:** corré otra vez el barrido determinista y la prueba del arreglo.

### Fase 6: nueva pasada y regla de corte

- Cada pasada nueva revisa **todo el alcance** con buscadores frescos, que no vieron las pasadas anteriores, y además mira en especial lo que se cambió en la pasada anterior.
- **Mínimo: tres pasadas completas,** siempre, aunque la primera no encuentre nada.
- **Corte:** después de la tercera, terminás cuando **las dos últimas pasadas** no encontraron ningún hallazgo crítico ni importante nuevo y confirmado. Si la tercera encuentra algo, seguís.
- **Tope:** 5 pasadas. Si a la quinta siguen apareciendo, el problema es de diseño, no de revisión: decilo y proponé el cambio de fondo en vez de seguir parchando.
- Un mismo tipo de error que reaparece se corrige en su origen: una regla de lint, una prueba, una línea en `AGENTS.md` o `REVIEW.md`, o una guarda compartida. Así la auditoría converge y no gira en círculos.

### Fase 7: informe

```
Alcance: <qué entró y qué no> · Profundidad: <rápida | integral> · Pasadas: <N, mínimo 3> (corte: <2 últimas limpias | tope>)
Barrido determinista: <herramienta: resultado> (primer fallo guardado)

| # | Severidad | Veredicto | Dimensión | Dónde (archivo:línea / servicio) | Qué falla | Escenario de falla | Evidencia ejecutada | Estado |
|---|---|---|---|---|---|---|---|---|

Simplificación (Ponytail): `<etiqueta> <qué se corta>. <reemplazo>. [ruta]` … → net: -<N> líneas, -<M> dependencias, -<K> variables
Preexistentes (no los introdujo el cambio): …
Menores (máximo 5): …

Comprobado y cómo: <lista: qué se probó y con qué comando o prueba>
NO comprobado y por qué: <lista: qué no se pudo probar y qué haría falta>
Pendiente del dueño: <decisiones o accesos que sólo él puede dar>
Frase final: «Sin errores conocidos sin resolver en el alcance auditado» o «Quedan N abiertos», nunca «sin errores».
```

En español rioplatense, con la lista leí / cambié / probé / pendiente al cierre si el proyecto lo usa.

## Cómo ejecutarla según la herramienta

- **Claude Code.** Invocar esta skill autoriza la orquestación de varios agentes que describe.
  - Si está la herramienta `Workflow`, cargá la skill `workflow-authoring` y armá un `pipeline`: buscadores por dimensión → verificador por hallazgo, apenas termina cada buscador → deduplicación → arreglos → nueva pasada.
  - Si no, usá el tool `Agent` con subagentes en paralelo: un mensaje con todos los buscadores, después los verificadores.
  - Tamaño por defecto: hasta 8 agentes por pasada. Si el usuario pide más profundidad o el alcance es enorme, escalá y avisá el costo aproximado.
  - Para un diff grande antes de fusionar, sugerí además `/code-review ultra`. La lanza el usuario, tiene costo y verifica en la nube.
- **Codex.** Si no podés lanzar subagentes, hacé los roles en secuencia con separación explícita.
  - Escribí cada hallazgo en un archivo.
  - Verificalo en una instrucción nueva que reciba sólo ese archivo y el código, con el mandato de refutarlo.
  - Si está disponible `claude -p`, usalo como crítico de otro modelo.
- **Sin agentes (último recurso):** hacé las fases igual, pero marcá en el informe que la verificación no fue independiente. Es menos confiable.

## Errores comunes que esta skill evita

| Error | Corrección |
|---|---|
| Usar sólo la lente de Ponytail y llamarlo auditoría | Ponytail es una de las 9 dimensiones |
| Revisar sólo lo que uno tocó | Alcance cerrado y escrito; lo de afuera se nombra como no auditado |
| «Lo leí y está bien» | Cada «está bien» dice cómo se comprobó |
| El que construyó revisa lo suyo | Verificador independiente con mandato de refutar |
| Diez agentes coinciden, entonces es verdad | La unanimidad no es evidencia; reproducir y, en lo crítico, sumar otro modelo |
| Informe lleno de detalles de estilo | Tope de 5 menores; primero lo crítico |
| Arreglar el síntoma donde se vio | Causa raíz donde pasan todos los llamadores |
| El arreglo agrega una capa | La regla del mínimo vale para el arreglo |
| Prometer «sin errores» | Decir qué se probó y qué no |
