# Dimensiones de la auditoría: qué mirar en cada una

Cada buscador recibe sólo su dimensión. Las preguntas no son una lista para tildar: son lugares donde suelen esconderse los errores. Las reglas escritas del proyecto le ganan a cualquier punto de esta lista.

## Contenido
1. Lógica, datos y especificación
2. Seguridad
3. Operación y confiabilidad
4. Rendimiento
5. Pruebas
6. Configuración y variables
7. Simplificación (Ponytail y Fowler)
8. Interfaz y accesibilidad
9. Documentación y coherencia

---

## 1. Lógica, datos y especificación
- **¿Hace lo pedido?** Requisitos de la especificación faltantes o parciales; comportamiento que nadie pidió; requisitos que parecen hechos pero están mal. Citá la línea de la especificación.
- **Bordes:**
  - valores vacíos, nulos o cero;
  - mínimo y máximo;
  - listas vacías y de un elemento;
  - fechas y husos horarios;
  - redondeo de dinero (enteros o decimales exactos, nunca `float` para plata);
  - unidades: gramos o kilos, centímetros o metros, con o sin IVA.
- **Caminos de error,** no sólo el feliz: ¿qué pasa si la API externa tarda, devuelve 500 o devuelve algo inesperado?
- **Estados y transiciones:** ¿se puede llegar a un estado imposible?, ¿un pedido puede pagarse dos veces o entregarse sin pago?
- **Concurrencia:** dos pedidos sobre el último artículo, carreras entre la notificación y el regreso del pago, bloqueos y su liberación.
- **Idempotencia:** los reintentos y las notificaciones repetidas no duplican efectos (cobros, stock, correos, facturas).
- **Dinero y stock:** el servidor decide el precio, el stock y las promociones, nunca el cliente; el regreso del checkout no prueba cobro, el cobro se verifica con el proveedor.
- **Datos:** migraciones reversibles, restricciones en la base (únicos, no nulos, claves foráneas) en lugar de sólo en la aplicación, borrado lógico frente a físico.
- **Lógica nueva metida en un flujo ajeno:** un `if` agregado a un flujo que no le corresponde es un olor de diseño, no un detalle.

## 2. Seguridad
Referencias: OWASP ASVS 5.0, OWASP Top 10 2025, OWASP Secure Code Review Cheat Sheet, Webhook Security Cheat Sheet.
- **Entradas:**
  - validación en el límite de confianza;
  - inyecciones (SQL, NoSQL, comandos, plantillas);
  - XSS (codificación de salida);
  - subida de archivos (tipo, tamaño, contenido);
  - redirecciones sólo a una lista permitida;
  - SSRF en búsquedas a URLs que da el usuario.
- **Autenticación y autorización:**
  - autorización en cada ruta protegida;
  - IDOR (¿puedo ver el pedido de otro cambiando el id?);
  - límite de intentos en el login;
  - sesiones con `httpOnly`, `secure` y `sameSite`;
  - tokens de un solo uso con vencimiento;
  - 2FA donde corresponda.
- **Secretos:**
  - fuera del código, de Git, de los logs, del navegador y de los artefactos;
  - permisos de archivo 600 o carpeta 700;
  - un secreto por integración;
  - rotación posible;
  - el `.env` nunca publicado.
- **Webhooks:**
  - firma HMAC verificada con comparación de tiempo constante antes de procesar;
  - marca de tiempo y ID de evento contra repeticiones;
  - procesamiento idempotente;
  - sólo `POST`;
  - errores genéricos (400 mal formado, 401 firma mala);
  - límite de tamaño.
- **Infraestructura:**
  - cabeceras (CSP, HSTS, X-Frame-Options);
  - CORS sólo a los orígenes reales;
  - mensajes de error sin trazas;
  - mínimo privilegio en usuarios de servicio y contenedores (`cap_drop`, `no-new-privileges`, no root);
  - puertos expuestos sólo los necesarios;
  - el panel de administración no público.
- **Redes:** las bases de datos sin salida a internet ni entrada pública; los servicios que llaman a APIs externas con salida (si no, la integración falla en silencio).
- **Dependencias:** vulnerabilidades conocidas, origen confiable, versiones fijadas.
- **Datos personales:** sólo lo necesario, fuera de los logs, copias cifradas, retención definida.
- **LLM, si hay:**
  - la salida del modelo es dato no confiable (nunca a `eval`, SQL, shell ni `innerHTML`);
  - el system prompt no es una barrera de seguridad;
  - permisos de herramientas acotados;
  - límites de tokens y de recursión.

## 3. Operación y confiabilidad
- **Copias de seguridad:**
  - ¿existen y con qué frecuencia?;
  - ¿hay una fuera del servidor?;
  - ¿están cifradas?;
  - **¿se probó restaurarlas?** Una copia sin restauración probada no cuenta;
  - ¿se borran solas las viejas?;
  - ¿avisan si fallan?
- **Avisos:**
  - ¿alguien se entera si el servicio cae, se llena el disco, falla una copia, se vence un certificado o una tarea programada deja de correr?;
  - ¿el aviso llega a un humano y se probó que llega?
- **Vuelta atrás:** ¿cómo se vuelve a la versión anterior y cuánto tarda?, ¿se ensayó?
- **Salud:**
  - chequeos de salud reales;
  - reinicio automático;
  - límites de memoria;
  - cierre ordenado;
  - tiempos de espera en llamadas externas;
  - reintentos con espera creciente.
- **Despliegue:**
  - reproducible;
  - migraciones antes de arrancar las réplicas;
  - secretos inyectados en ejecución;
  - el mismo artefacto en prueba y en producción;
  - sin pasos manuales escondidos.
- **Recursos:** disco, memoria, CPU, límites de planes gratuitos, vencimientos de dominios, certificados y planes.
- **Observabilidad:** errores enviados a Sentry o un equivalente (¿llegan?), logs con contexto y sin secretos.
- **Restos:** procesos, contenedores, volúmenes, imágenes, archivos y paquetes que ya no sirven (también es regla del mínimo).

## 4. Rendimiento
- Consultas N+1, falta de índices, traer de más, falta de paginación.
- Trabajo pesado en la ruta caliente o en el hilo del pedido; lo que puede ir en segundo plano.
- Front:
  - peso del bundle;
  - imágenes sin tamaño ni formato moderno;
  - LCP, CLS e INP;
  - re-renders innecesarios;
  - fuentes bloqueantes.
- Cachés: ¿se invalidan bien? Una caché vieja es un error de lógica.
- Medí antes de proponer: sin números, una optimización es una opinión.

## 5. Pruebas
- ¿Existe una prueba para cada comportamiento crítico (dinero, stock, autorización, webhooks)?
- ¿Prueba el comportamiento o la implementación? ¿Fallaría si el código se rompe? Rompé el código a propósito un segundo y miralo fallar.
- ¿Cubre los bordes y los caminos de error?
- ¿Las pruebas usan datos o proveedores ficticios cuando corresponde y nunca tocan producción?
- ¿Hay pruebas que siempre pasan, saltadas u omitidas en silencio?

## 6. Configuración y variables (regla del mínimo)
- **Cada variable de entorno justificada:**
  - ¿qué código la lee?;
  - ¿qué pasa si falta?;
  - si su valor es el mismo que el código asume cuando falta, sobra;
  - si el código trata la ausencia distinto (`!== 'false'`), se queda.
- **No secretos fuera del gestor de secretos:** URLs, IDs públicos y constantes van versionados.
- **Un secreto en un solo lugar:** sin URLs que repitan contraseñas, sin copias en varios archivos.
- **Banderas:** las apagadas por defecto no se declaran; cada bandera encendida tiene responsable y motivo.
- **Compose y CI:**
  - imágenes fijadas;
  - nada que se baje sin verificar;
  - permisos;
  - redes y puertos mínimos;
  - perfiles o servicios que nadie usa.
- **Nombres engañosos:** algo llamado «staging» en producción, «temporal» que lleva meses, «pruebas» por defecto en un archivo de producción.

## 7. Simplificación (Ponytail y Fowler)
- **Etiquetas de Ponytail:** `delete:` · `stdlib:` · `native:` · `yagni:` · `shrink:`. Una línea por hallazgo: `<etiqueta> <qué se corta>. <reemplazo>. [ruta]`, con el cierre `net: -N líneas, -M dependencias`.
- **Qué buscar:**
  - dependencias que la plataforma ya trae;
  - interfaces con una sola implementación;
  - fábricas de un solo producto;
  - envoltorios que sólo delegan;
  - archivos que exportan una sola cosa;
  - banderas y configuración muertas;
  - stdlib reimplementada;
  - código «para más adelante».
- **Olores de Fowler** (siempre son criterio, nunca una violación dura; el estándar del repositorio gana):
  - nombre misterioso;
  - código duplicado;
  - envidia de datos;
  - grupos de datos que viajan juntos;
  - obsesión por primitivos;
  - `switch` repetidos;
  - cirugía de escopeta;
  - cambio divergente;
  - generalidad especulativa;
  - cadenas de mensajes;
  - intermediario;
  - herencia rechazada.
- **¿El refactor reduce complejidad o sólo la mueve?** Contá los conceptos que hay que tener en la cabeza; si no baja, no es más simple.
- No marcar para borrar la única prueba o autocomprobación mínima: es el mínimo de Ponytail, no grasa.
- `ponytail:` sin disparador de mejora es deuda que se pudre: listarla (como `ponytail-debt`).

## 8. Interfaz y accesibilidad (si hay interfaz)
- Recorridos principales en 390 y 1440 px, sin desborde horizontal.
- Teclado: foco visible, orden lógico, todo alcanzable.
- Contraste, textos alternativos, etiquetas de formularios, mensajes de error claros.
- Estados: vacío, cargando, error y sin stock.
- Datos reales largos (nombres y precios grandes) que no rompan el diseño.
- Consola del navegador sin errores; pedidos de red sin 4xx ni 5xx inesperados.
- Para juzgar diseño o «slop», derivar a la skill de diseño del proyecto (por ejemplo `design-unslopify`).

## 9. Documentación y coherencia
- ¿La documentación de continuidad, operación y despliegue describe lo que el sistema hace **hoy**? Cada diferencia es un hallazgo: la próxima persona o agente va a actuar sobre la versión vieja.
- ¿Hay instrucciones que mandan a rutas, comandos o servicios que ya no existen?
- ¿Se cumplen las reglas de `AGENTS.md` y `CLAUDE.md` en lo auditado?
- ¿Hay decisiones escritas del responsable contradichas por el código o la configuración?
