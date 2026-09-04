# Roadmap y Tareas — Portal del Empleado

Estado de desarrollo, tareas prioritarias y backlog de características.

---

## 🎯 En Curso (Alta Prioridad)
- [x] Establecer conexión MCP con Obsidian y estructurar la memoria técnica.
- [ ] Validar el pipeline completo de firma PDF. **Versiones corregidas el 2026-09-04** (estaban mal en el roadmap, en `CLAUDE.md` y en la nota de arquitectura, que decían "FPDI 2.1.7"): son **tres** librerías con numeración propia — **FPDI 2.6.3** (`libs/fpdi/`), **FPDI PDF-Parser 2.1.7** (`libs/pdf-mod/`, el add-on comercial, de donde venía el 2.1.7) y **TCPDF 6.9.4** (`libs/tcpdf/`). Ya corregidos los tres sitios. Ver [[Proyectos/Portal del Empleado/Arquitectura/Arquitectura-PDF-Signature|Arquitectura PDF/Firma]].
- [ ] **Subir FPDI de 2.6.3 a 2.6.6+**: el `composer.json` del PDF-Parser 2.1.7 declara `setasign/fpdi: ^2.6.6`. Hoy funciona igualmente (en ejecución sólo exige `PdfString::escape`, y se verificó que la 2.6.3 ofrece todo lo que el parser usa), pero el desajuste está ahí. Requiere revisar `EP_Fpdi_V4`, que depende de las interioridades de FPDI.
- [ ] Revisar compatibilidad de endpoints REST y AJAX con la autenticación SSO de Microsoft 365.
- [ ] Retirar el mu-plugin `club1899-portal-sync.php` en producción y en `devpruebas` (duplica el envío de empresas hacia preproducción). Ver [[Proyectos/Portal del Empleado/Despliegue|Despliegue y Operativa]]. *(No verificable desde el repositorio: el mu-plugin vive en el servidor; en el código sólo aparece `club1899.es` como destino por defecto del connector en `plugins/ep-empresas/class-ep-empresas.php`.)*
- [x] ~~Bot de Teams sin respuesta: diagnóstico~~ → **causa real: TLS mínimo 1.3 en Cloudflare** (el conector de Teams sólo negocia TLS 1.2). Resuelto bajando el mínimo a 1.2; el bot responde con el proxy activo (2026-09-04 12:49).
- [x] ~~Reactivar Cloudflare sin romper el bot (Bot Fight Mode / Browser Integrity Check / Security Level)~~ → **descartado**: no era la causa. Superado por el arreglo de TLS 1.2.
- [x] ~~Bot de Teams sin respuesta (2026-09-04): revisar canal msteams, Security Events y regla WAF de Skip~~ → **descartado**: misma causa TLS. Superado por el arreglo de TLS 1.2.
- [ ] Cloudflare → SSL/TLS: pasar de *Flexible* a *Full (strict)* (el origen sirve HTTPS válido; ahora el proxy habla HTTP con el servidor).
- [ ] SSL Full (strict) **solo para portal**: Cloudflare → Reglas → Configuration Rules → expresión `(http.host eq "portal.camaracaceres.com")` → ajuste SSL = *Full (strict)*. No cambiar el modo de toda la zona: diez subdominios (deovs, erp, inv, magicinfo, old, preccc, prensa, titulos, espace-admin, sistemas) no tienen certificado válido en el origen y darían error 526. Alternativa a largo plazo: emitirles certificado con AutoSSL de cPanel o retirar los DNS que ya no se usen.

## 📋 Backlog de Funcionalidades
##### Bot de Teams (`class-ep-bot-mensajeria.php`, `class-ep-bot-briefing.php`)
- [x] Resumen del saludo muestra la agenda de **hoy**, no la próxima cita de otro día; botón "📅 Próxima cita" (v2.1.1, desplegada 2026-09-04).
- [x] Endpoint endurecido: se exige JWT de Bot Framework en REST, AJAX y endpoint nativo `?ep_bot=1`; ya no se tolera la ausencia de cabecera Authorization (v2.1.2).
- [x] Saludo "buenos días/tardes" con hora de Madrid en vez de UTC del servidor (v2.1.2).
- [x] Atajos sin IA: "mis tareas", "notificaciones", "resumen", "firmas", "tickets", "inventario", "agenda"/"hoy"/"mañana", "próxima cita" y los botones de las tarjetas se resuelven por coincidencia directa, sin llamar a Gemini (v2.1.2).
- [x] Briefing matinal proactivo a las **8:00** (L-V), opt-in en Ajustes → Notificaciones. Se omite si el usuario tiene respuestas automáticas de Outlook activas o un evento de día completo tipo vacaciones/baja/permiso en su calendario (v2.1.2). Pendiente de desplegar y verificar el primer envío real.
- [ ] Botones que actúan: "Firmar ahora" (deep link al documento), "Marcar tarea hecha" en To-Do, "Cerrar ticket" / "Responder" desde la tarjeta. *(Confirmado pendiente 2026-09-04: el `invoke` sí llega y se procesa, pero el manejador se limita a leer `value.action.data.m` y reenviarlo a `generar_respuesta()` como si fuera texto — o sea, los botones sólo saben **consultar**, no ejecutar. Falta un despachador por acción antes de esa llamada.)*
- [ ] Más notificaciones proactivas. *(Estado real: ya existen dos — nueva solicitud de firma (`ep_signature_request_created` → `notificar_nueva_firma()`, con canal Graph y fallback por `conversation_id`) y recordatorio diario de firmas pendientes a las 48 h (`ep_signature_pending_reminders_cron`). Siguen sin hacerse: aviso interno nuevo, ticket asignado, firma completada por la otra parte y recordatorio 15 min antes de una reunión con sala.)*
- [ ] Seguimiento conversacional: extender `EP_Bot_Context` a la agenda para que "¿y mañana?" / "¿y el viernes?" funcionen tras una consulta. *(Confirmado pendiente: `EP_Bot_Context` guarda intent, params y resultados 10 min, pero `get_prompt_context()` sólo sabe resumir empresa (`RAZON`/`NIF`) y persona (`display_name`); no hay nada de fechas ni de agenda.)*
- [ ] Panel de "no entendí": pantalla de administración sobre `log_uncertain_intent`. *(Confirmado pendiente: `EP_AI_Service::log_uncertain_intent()` existe y se llama desde el bot, pero no hay ninguna pantalla que lea ese registro — en `admin/` sólo están `class-ep-admin.php` y `class-ep-deployer.php`.)*

### Módulo de Firma Electrónica (`ep-signature`)
- [ ] Verificación en frontend de firmas de certificados electrónicos cualificados. *(Parcial: existe verificación pública del documento por CSV — `render_verification_view()` + `views/verification-view.php` — y se parsea el X.509 del firmante con `openssl_x509_parse()` al guardar. Falta lo pendiente de verdad: validar la cadena de confianza / cualificación del certificado y mostrarla al verificar.)*
- [ ] Automatización del sellado de tiempo (TSA) en documentos emitidos por la empresa. *(Confirmado pendiente: sin rastro de TSA/RFC 3161 en el código.)*
- [x] Interfaz de visualización de historial de firmas para el empleado. **Ya implementada** (2026-09-04): pestaña "Mis Documentos" en `views/signature-view.php` sobre la tabla `#fds-my-docs-table`, servida por `handle_get_my_docs()`, con acciones en lote (borrar, enviar por email, ZIP). Ojo: el shortcode `[fds_mis_documentos]` que citaba esta tarea **ya no existe**; sigue invocándose en `public/partials/signature-app.php`, que es código muerto y habría que retirar.

### Módulo de Comunicación y Tickets (`ep-buzon` / `ep-avisos`)
- [ ] Notificaciones push o webhooks de Teams cuando se crea o actualiza un ticket. *(Medio hecho: `EP_Bot_Mensajeria::notificar_cambio_ticket()` ya está escrito, pero **no está enganchado a ningún hook** — no lo llama nadie en todo el repositorio. Lo que falta es disparar la acción desde `ep-tickets` al crear/actualizar, y usar el canal proactivo de Graph como en firmas, no sólo la `conversation_id` guardada.)*
- [x] Filtros y búsqueda por departamento en el directorio de empleados. **Ya implementado** (2026-09-04), aunque en `ep-directory`, no en `ep-censo`: `build_department_index()` + `normalize_department_key()` agrupan las variantes de escritura que llegan de M365, y `views/directory-view.php` pinta el selector "Todos los departamentos" y el filtro por texto (`data-search-text`, `data-department-key`).

### Integración M365 (`ep-calendar` / Graph)
- [ ] Sincronización bidireccional de ausencias/vacaciones con el calendario personal de Outlook. *(Parcial: `EP_OOF_Sync` ya sincroniza en los dos sentidos el estado "Fuera de la oficina" — portal → M365 con `handle_oof_update_ajax()` sobre `/me/mailboxSettings`, y M365 → portal por token de aplicación + `$batch` cada 15 min hacia el user_meta `ep_oof_info`. Lo que **no** existe es gestión de vacaciones como tal ni escritura de eventos en el calendario: no hay ninguna llamada a `/events` en el código.)*
- [ ] Mapeo de grupos de seguridad de Azure AD a roles de la suite. *(Confirmado pendiente: no hay ninguna consulta a `memberOf` ni a `/groups` en el repositorio.)*

## 📌 Decisiones Pendientes (ADR)
* Evaluar si delegar operaciones criptográficas pesadas a un microservicio backend o mantener 100% en Web Crypto API + PHP nativo.
