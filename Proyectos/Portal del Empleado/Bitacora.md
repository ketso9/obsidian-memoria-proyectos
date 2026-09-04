# Bitácora de Sesiones - Portal del Empleado

## 📌 2026-09-04 (14:20) - Auditoría del Roadmap y corrección de las versiones del stack PDF

**Qué se hizo.** Revisión del `Roadmap.md` contra el código real del repositorio, a petición
del usuario, que sospechaba que había tareas ya implementadas marcadas como pendientes.
Tenía razón en varias.

**Tareas que ya estaban hechas y se han marcado:**
* *Historial de firmas para el empleado*: existe como pestaña "Mis Documentos" en
  `views/signature-view.php` (tabla `#fds-my-docs-table`, sub-action AJAX `get_my_docs`),
  con borrado, envío por email y ZIP en lote.
* *Filtros y búsqueda por departamento en el directorio*: hecho, pero en `ep-directory`,
  no en `ep-censo` como decía la tarea. `build_department_index()` y
  `normalize_department_key()` agrupan las variantes de escritura que llegan de M365.
* Los dos ítems de Cloudflare sobre Bot Fight Mode, Browser Integrity Check y regla WAF
  de *Skip*: quedaron descartados cuando se vio que la causa era el TLS mínimo.

**Corrección importante de nomenclatura — "FPDI 2.1.7" era un error.** El roadmap,
`CLAUDE.md` y `Arquitectura-PDF-Signature.md` decían que el pipeline era "FPDI 2.1.7".
Leyendo el código, son **tres librerías distintas con numeración propia**:

| Ruta | Librería | Versión | Cómo se comprobó |
| :--- | :--- | :--- | :--- |
| `libs/fpdi/` | FPDI | **2.6.3** | `src/Fpdi.php`, `const VERSION` |
| `libs/pdf-mod/` | FPDI PDF-Parser (add-on comercial `setasign/fpdi_pdf-parser`) | **2.1.7** | `composer.json` + commit `9907a4e` |
| `libs/tcpdf/` | TCPDF | **6.9.4** | fichero `VERSION` y `composer.json` |

El "2.1.7" venía del **PDF-Parser**, no de FPDI. Corregido en los tres sitios.

**Deuda técnica que aparece al desenredarlo:** el `composer.json` del PDF-Parser 2.1.7
declara `setasign/fpdi: ^2.6.6`, o sea pide una FPDI más nueva de la que hay. Funciona
porque la comprobación en tiempo de ejecución sólo exige `PdfString::escape` y la 2.6.3
ofrece todo lo que el parser usa — así se verificó al actualizar el parser (`9907a4e`) —
pero el desajuste sigue ahí. Añadida al roadmap como tarea propia; requiere revisar
`EP_Fpdi_V4`, que depende de las interioridades de FPDI.

**Código muerto detectado:** `public/partials/signature-app.php` invoca los shortcodes
`[firma_documentos]` y `[fds_mis_documentos]`, que **ya no existen** — en todo el
repositorio sólo hay cuatro `add_shortcode`, y ninguno es `fds_*`. La sección 4 de la nota
de arquitectura describía esos shortcodes; se ha reescrito con lo que hay de verdad
(vistas + un único `handle_ajax()` que despacha por `sub_action`).

**Media tarea a un hook de distancia:** `EP_Bot_Mensajeria::notificar_cambio_ticket()` está
escrita pero **no la llama nadie**: no hay ningún `add_action` que la enganche. Anotado en
el roadmap.

**Notas del vault modificadas:** `Roadmap.md` y `Arquitectura/Arquitectura-PDF-Signature.md`.
**Ficheros del repositorio modificados:** `CLAUDE.md` (sección de convenciones, con las
tres versiones y el aviso de que `deploy.ps1` / `deploy-staging.ps1` excluyen
`plugins/ep-signature/libs/*`, que se suben con `deploy-fpdi-parser.ps1`).
## 📌 2026-09-04 (13:35) - Despliegue Integral v2.1.2 (Briefing Teams) a Prod/Staging y Limpieza
* **Responsable:** Antigravity/Gemini + Usuario.
* **Contexto:** Se detectaron y desplegaron los 7 ficheros correspondientes a la versión 2.1.2 (briefing matinal a las 8:00, atajos directos sin IA, corrección de franja horaria en saludos y conmutador en ajustes de empleado).
* **Ficheros desplegados (Producción y Staging):**
  * `employee-portal.php` (versión 2.1.2).
  * `includes/class-ep-bot-briefing.php` (nuevo módulo cron matinal).
  * `includes/class-ep-bot-mensajeria.php` (atajos de respuesta directa y saludo en hora Madrid).
  * `includes/class-ep-graph-service.php` (soporte para eventos de día completo en agenda de hoy).
  * `includes/class-ep-loader.php` (inicialización de briefing matinal).
  * `includes/apps/class-ep-app-settings.php` (registro de la preferencia de usuario).
  * `public/partials/settings-app.php` (conmutador frontend para el empleado).
* **Limpieza operativa de servidores:**
  * Retirado `wp-content/mu-plugins/club1899-portal-sync.php` (renombrado a `.retirado`) en Producción y Staging, eliminando la duplicidad de tráfico residual hacia preproducción.
* **Verificación:**
  * Backups previos creados en `~/backups_avisos/bot_2_1_2/{prod,staging}/`.
  * `php -l` con 0 errores de sintaxis en ambos entornos.
  * Hashes MD5 locales, de staging y de producción 100% coincidentes.
## 📌 2026-09-04 (13:10) - Despliegue v2.1.1 (Mejora de agenda en Bot de Teams)
* **Responsable:** Claude Code (desarrollo) + Antigravity/Gemini (despliegue selectivo y verificación) + Usuario.
* **Contexto:** Claude Code implementó la versión 2.1.1 para depurar la visualización de citas en las tarjetas de Teams (mostrando la agenda de hoy y evitando solapar eventos de días posteriores). El despliegue automático quedó bloqueado por las directivas de seguridad/permisos de Claude Code para ejecutar SCP hacia servidores remotos.
* **Ficheros desplegados a producción:**
  * `employee-portal.php` (versión 2.1.1).
  * `includes/class-ep-graph-service.php` (método `get_today_events` con caché de 5 minutos).
  * `includes/class-ep-bot-mensajeria.php` (resumen de hoy con conteo + botón "📅 Próxima cita").
* **Procedimiento y Verificación:**
  * Backup previo en el servidor: `~/backups_avisos/bot_2_1_1/prod/`.
  * Despliegue selectivo por SCP hacia `portal.camaracaceres.com/wp-content/plugins/Portal-empleado-1`.
  * Linting remoto: `php -l` OK sin errores en los 3 ficheros.
  * Verificación de integridad MD5 remota confirmada contra copia local (hashes idénticos).

Registro cronológico de cambios, decisiones técnicas, migraciones y resoluciones de incidencias en el proyecto.

---
## 📌 2026-09-04 (tarde) - Bot de Teams: pulido y productividad (v2.1.1 → v2.1.2)
* **Responsable:** Claude Code (Fable 5.1) + Usuario (despliegue vía Gemini/Antigravity)
* **v2.1.1 (desplegada y verificada por el usuario):** el resumen al saludar mostraba la "próxima cita" aunque fuera de otro día. Ahora `EP_Graph_Service::get_today_events()` consulta solo lo que queda de hoy (hora de Madrid, sin eventos de día completo, caché 5 min) y las tarjetas de bienvenida y resumen muestran "📅 Hoy (N): 10:00 Asunto (+2 más)" o "Sin más reuniones". Botón "📅 Próxima cita" en ambas tarjetas.
* **v2.1.2 (pendiente de despliegue):**
  * **Seguridad:** REST, AJAX y endpoint nativo `?ep_bot=1` exigen JWT válido de Bot Framework (`peticion_autenticada()`); se retira la tolerancia a peticiones sin Authorization (el log demuestra que la cabecera llega siempre).
  * **Saludo** con hora de Madrid (antes UTC del servidor).
  * **Atajos sin IA** (`atajo_directo()`): frases fijas ≤ 40 caracteres, normalizadas sin tildes, resueltas sin llamar a Gemini. Las apps se invocan por el mismo filtro `ep_bot_handle_intent_*` que usa la IA (`ejecutar_intent_app()`), con comprobación de permisos. Nueva `tarjeta_proxima_cita()` con el día explícito.
  * **Briefing matinal** (`includes/class-ep-bot-briefing.php`, nuevo): cron cada 15 min que envía a las 8:00 (ventana 8-11, L-V, hora de Madrid, una vez al día vía opción `ep_bot_briefing_last_date`). Opt-in por usuario (`user_meta ep_bot_briefing`, interruptor en Ajustes → Notificaciones, solo si el canal Teams está contratado). Se omite al usuario si `EP_OOF_Sync` dice que está fuera o si hoy tiene un evento de día completo con palabras tipo vacaciones/baja/permiso/libre/festivo. Requiere que el usuario haya escrito antes al bot (usa `ep_bot_conversation_id`).
  * `EP_Bot_Mensajeria` expone `instance()`, `tarjeta_briefing()` y `enviar_tarjeta_a_usuario()`; `enviar_respuesta()` devuelve bool.
* **Ficheros a desplegar (v2.1.2):** `employee-portal.php`, `includes/class-ep-bot-mensajeria.php`, `includes/class-ep-graph-service.php`, `includes/class-ep-bot-briefing.php` (nuevo), `includes/class-ep-loader.php`, `includes/apps/class-ep-app-settings.php`, `public/partials/settings-app.php`.
* **Prueba del briefing sin esperar a mañana:** `wp eval '(new EP_Bot_Briefing())->enviar_a_todos(date("Y-m-d"));'` desde la raíz de WordPress, con el interruptor activado en Ajustes del usuario de prueba.

* **v2.1.2 desplegada (13:40, vía Gemini) y verificada por Claude Code:** md5 de los 7 ficheros idénticos a local, `php -l` limpio, versión 2.1.2 en producción, cron `ep_bot_briefing_check` programado (cada 15 min), y un POST al endpoint sin token responde **401** (endurecimiento activo). Primer briefing real: próximo día laborable a las 8:00.
## 📌 2026-09-04 (tarde) - Bot de Teams: los mensajes del canal Teams no llegan al servidor
* **Responsable:** Claude Code (Fable 5.1) + Usuario
* **Contexto:** el bot sigue sin contestar a "hola" / "ayuda" desde Teams. Se revisó el log del plugin en producción (`ep_debug.log`) y el log de acceso de cPanel (`~/access-logs/portal.camaracaceres.com`).
* **Evidencia:**
  * Todas las actividades entrantes de Bot Framework registradas (3 y 4 de septiembre) llevan `channelid: webchat` y User-Agent `BF-DirectLine` (IP 20.43.40.64). Son las pruebas del *Test in Web Chat* del portal de Bot Framework, y esas **sí** se responden con HTTP 200 y tarjeta enviada.
  * **No existe ni una sola petición con canal `msteams`**, ni en el log PHP ni en el log de acceso del servidor web. Lo escrito en Teams nunca llega al origen.
  * El manifiesto (`manifest.json`, v1.4.4 en repo / 1.4.5 publicada) apunta al `botId` correcto y las opciones de producción coinciden: `ep_teams_bot_id` = dfcd7250-…, `ep_o365_client_id` = 954318d3-…, tenant a853cfe5-…. El endpoint responde OK a GET.
  * El código del endpoint (`class-ep-bot-mensajeria.php`) funciona: ya respondió a un "hola" por WebChat el 03-09 a las 11:22 (usuario no reconocido porque WebChat no envía `aadObjectId`, pero la tarjeta se entregó).
* **Conclusión:** el problema **no está en el plugin**, está antes: o el canal *Microsoft Teams* no está realmente habilitado/sano en el registro del bot, o las peticiones del canal Teams (IPs de Azure, UA `Microsoft-BotFramework/3.1`) se bloquean en Cloudflare antes de llegar al origen. El "check azul" en Teams solo indica que Teams aceptó el mensaje, no que Bot Framework lo entregara al endpoint.
* **Siguiente paso (usuario):** ver [[Proyectos/Portal del Empleado/Roadmap|Roadmap]], apartado "En Curso".

* **Resolución (11:49):** el usuario **pausó Cloudflare** en la zona y el bot respondió al "hola" desde Teams. En `ep_debug.log` entró por fin una actividad `message` con canal `msteams`, desde la IP 52.123.136.131 (ASN 8075 Microsoft) y User-Agent `Microsoft-SkypeBotApi (Microsoft-BotFramework/3.0)`; usuario reconocido (Jorge Polo Cortés) y respuesta entregada con HTTP 201 en `smba.trafficmanager.net/emea/{tenant}`. **El plugin y el registro del bot están bien.**
* **Causa confirmada:** Cloudflare descartaba las peticiones del canal Teams **sin dejar evento** en Seguridad → Eventos (la regla personalizada "Autorizar Bot Microsoft", de tipo *Omitir*, solo exime de WAF/reglas administradas; no exime del *Bot Fight Mode* ni del *Browser Integrity Check* en el plan Free). El tráfico del *Test in Web Chat* (IP 20.43.40.64) sí pasaba, por eso engañaba.
* **Detalle colateral:** con Cloudflare activo las peticiones llegan al origen por HTTP (aparecen en `access-logs/portal.camaracaceres.com`, no en el `-ssl_log`), es decir, el modo SSL de Cloudflare está en *Flexible*. Conviene pasarlo a *Full (strict)*, el origen ya tiene certificado válido.
* **Pendiente:** no dejar Cloudflare pausado. Ver tarea en [[Proyectos/Portal del Empleado/Roadmap|Roadmap]].

* **Causa raíz definitiva (12:25):** al reanudar Cloudflare el bot volvió a fallar aunque *Bot Fight Mode* estaba apagado y existía una Configuration Rule para la ruta. Una petición simulada del canal Teams (mismo UA `Microsoft-SkypeBotApi`, misma ruta, cabecera Authorization) **sí** atravesaba Cloudflare desde una IP doméstica. La diferencia estaba en la capa TLS: el borde de Cloudflare de la zona `camaracaceres.com` **solo acepta TLS 1.3** (`curl --tls-max 1.2` → fallo de handshake; `--tlsv1.3` → 200; contra el origen directo TLS 1.2 → 200). El conector de Teams negocia TLS 1.2, así que la conexión moría en el handshake, antes de HTTP: por eso no dejaba evento en Seguridad ni en el servidor. DirectLine/Web Chat usa TLS 1.3 y por eso pasaba.
* **Corrección:** Cloudflare → SSL/TLS → Edge Certificates → *Minimum TLS Version* = **TLS 1.2** (estaba en 1.3). La Configuration Rule de la ruta del bot y la regla WAF "Autorizar Bot Microsoft" pueden quedarse, no estorban. Ver [[Proyectos/Portal del Empleado/Arquitectura/Integracion-Microsoft-Graph|Integración Microsoft Graph]] para el pipeline entrante.

* **Cerrado (12:49):** con *Minimum TLS Version* = 1.2 y Cloudflare **activo**, el "hola" desde Teams entró (IP 52.123.134.93, canal `msteams`), usuario reconocido y respuesta 201. **Incidencia resuelta.**
* **Sobre pasar SSL a Full (strict):** es un ajuste de toda la zona. Probado el origen (178.211.133.52) con verificación estricta para cada subdominio: tienen certificado válido `camaracaceres.com`, `www`, `portal`, `analitica`, `bonificada`, `camarajobs.es`, `campus`, `campuspotencialcdc`, `cjobs.es`, `club`, `club1899`, `comunicacion`, `empleo`, `firmasegura`, `liderafp.com`, `liderafp.es`, `videopotencialcdc`. **Sin certificado válido en el origen** (fallarían con error 526 en Full strict si su origen es este servidor): `deovs`, `erp`, `inv`, `magicinfo`, `old`, `preccc`, `prensa`, `titulos`, `espace-admin`, `sistemas`. Recomendación: no cambiar la zona; crear una *Configuration Rule* en Cloudflare solo para `portal.camaracaceres.com` con SSL = Full (strict).
## 📌 2026-09-04 - Diagnóstico Integral del Bot de Teams, Bot Framework y Gemini API
* **Responsable:** Antigravity AI + Usuario
* **Contexto:** Incidencia en la mensajería del Bot de Microsoft Teams (mensajes entrantes sin respuesta en el chat de Teams).
* **Diagnóstico y Pruebas Realizadas:**
  1. **Envío Proactivo (Servidor → Teams / Chat 1:1):**
     * Verificado y **100% funcional**.
     * El servidor descifra p_teams_bot_secret y obtiene token en login.microsoftonline.com/a853cfe5-f96e-4342-99af-95f72c72dea7.
     * Entrega confirmada de Adaptive Cards al usuario Jorge en la región EMEA (https://smba.trafficmanager.net/emea/v3/conversations/...).
  2. **Notificaciones de Actividad en Feed (sendActivityNotification):**
     * Fallan con código 403 Forbidden (TeamsActivity.Send concedido, pero Graph requiere instalación de la app vía catálogo organizativo exacto). El fallback automático conmuta a Chat 1:1 con éxito.
  3. **Motor de Inteligencia Artificial (Google Gemini API):**
     * Verificada la conexión de EP_AI_Service con gemini-3.1-flash-lite-preview (	est_connection: OK).
     * Comprobada la clasificación de intenciones (get_intent): clasifica correctamente saludos (CONVERSATIONAL), reuniones (AGENDA) y búsqueda de normativas internas (DOCUMENTS).
  4. **Mensajería Entrante (Teams → Servidor):**
     * El endpoint REST POST /wp-json/employee-portal/v1/teams-bot está activo y procesa peticiones en <1s.
     * Identificado que el bot está configurado en dev.botframework.com (App ID dfcd7250-abfd-4689-8ad4-a5163898de14, Tenant 853cfe5-f96e-4342-99af-95f72c72dea7, Single-Tenant).
     * En Teams Admin Center, la app Portal Empleado v1.4.5 está restringida por directiva a 1 grupo corporativo (donde el usuario pertenece).
     * Tras resolver el bloqueo inicial en el cliente de Teams (cambió de *Error al enviar* a *Enviado/check azul*), se verificó que las alertas de Conversation not found en Bot Framework correspondían al intento previo en la región global antes del fallback a EMEA.
     * Documentados los pasos finales para la sincronización del canal Microsoft Teams en el portal de Bot Framework.

---

## 📌 2026-09-04 - Claude Code se suma al vault: memoria compartida entre agentes
* **Responsable:** Claude Code (Opus 5) + Usuario
* **Contexto:** el vault estaba enlazado únicamente con Gemini/Antigravity. El objetivo de esta sesión era que **ambos agentes compartan la misma memoria**.
* **Hitos:**
  * Verificado el servidor obsidian-local-rest-api v5.1.0 desde Claude Code: responde en 127.0.0.1:27123, el handshake MCP en /mcp/ negocia protocolVersion 2025-06-18 con capacidades 	ools y 
esources.
  * Detectado que la configuración vivía solo en ~/.gemini/config/mcp_config.json; en el lado de Claude, mcpServers estaba vacío.
  * Creado CLAUDE.md en la raíz del repositorio como espejo de .agent/rules/obsidian_memory.md, para que ambos agentes sigan **el mismo protocolo** de consulta y actualización del vault.
  * Migradas al vault las dos notas que Claude mantenía en su memoria local: criterio de **despliegue selectivo por fichero** y la retirada pendiente del mu-plugin club1899-portal-sync.php, ahora en [[Proyectos/Portal del Empleado/Despliegue|Despliegue y Operativa]].
* **Decisión:** el vault es la **única fuente de verdad**. La memoria local de cada agente queda reducida a un puntero hacia aquí, para evitar que las dos se desincronicen.
* **Nota operativa:** el plugin solo escucha mientras Obsidian esté abierto. Si un agente no alcanza el vault, comprobar eso antes que la configuración.

---

## 📌 2026-09-04 - Configuración de Memoria y Enlace con Obsidian MCP
* **Responsable:** Antigravity AI + Usuario
* **Hitos:**
  * Configuración del servidor MCP para Obsidian (obsidian-local-rest-api) utilizando el puerto HTTP 27123 en ~/.gemini/config/mcp_config.json.
  * Verificación de la conexión y herramientas MCP (ault_read, ault_write, ault_list).
  * Estructuración del Vault de Obsidian como la memoria viva y documentación técnica permanente del proyecto.
  * Migración y reorganización de notas de proyecto en Proyectos/Portal del Empleado/.

---

## 📌 Hitos Previos (Histórico del Repositorio)
* **Rama activa:** `actualizacion/fpdi-pdf-parser-2.1.7`
* **Actualización del parser PDF** (commit `9907a4e`, 2026-08-13):
  * Actualización del **FPDI PDF-Parser** (`libs/pdf-mod/`, add-on comercial de setasign) de
    **2.1.6 a 2.1.7**. ⚠️ *Corregido el 2026-09-04: aquí ponía "la librería FPDI a la versión
    2.1.7", y no es así — FPDI es otra librería y está en la 2.6.3.*
  * Motivo real de la actualización: la 2.1.6 aceptaba sin validar los parámetros que vienen
    dentro del propio PDF, y el portal parsea ficheros que suben los empleados. Se corrigieron
    `Predictor` (`/Colors` y `/Columns` sin validar → `array_fill` agotaba la memoria del
    worker), `CompressedReader` (`/N` y `/W` sin comprobar), `SecHandler` (`/Length 0` →
    división por cero) y `SaslPrep` (un `|` literal dentro de una clase de caracteres borraba
    la barra vertical de las contraseñas, así que un PDF AES-256 con ese carácter en la clave
    no se podía abrir).
  * En el plugin: se retiró el fallback que cargaba el autoloader del parser desde
    `wp-content/uploads` (directorio escribible: cargar PHP desde ahí convierte cualquier
    subida en ejecución de código), y se añadió un tope de 25 MB antes de entregar el PDF al
    parser (`ep_signature_max_pdf_bytes`).
  * Probado con 40 PDF reales, 25 de ellos con xref comprimido: 40 correctos, 0 fallos.
  * Se añadió `deploy-fpdi-parser.ps1` porque `deploy.ps1` y `deploy-staging.ps1` excluyen
    `plugins/ep-signature/libs/*`: estas librerías no viajan en el despliegue normal.
