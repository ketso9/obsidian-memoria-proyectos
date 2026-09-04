# Bitácora de Sesiones - Portal del Empleado

Registro cronológico de cambios, decisiones técnicas, migraciones y resoluciones de incidencias en el proyecto.

---

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
  * Verificado el servidor obsidian-local-rest-api v5.1.0 desde Claude Code: responde en 127.0.0.1:27123, el handshake MCP en /mcp/ negocia protocolVersion 2025-06-18 con capacidades 	ools y esources.
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
* **Rama activa:** ctualizacion/fpdi-pdf-parser-2.1.7
* **Actualización del parser PDF:**
  * Actualización de la librería FPDI a la versión 2.1.7 compatible con versiones recientes de PDF y PHP 8.x.
  * Pruebas de compatibilidad en el pipeline de firma digital y estampado de códigos QR / CSV.