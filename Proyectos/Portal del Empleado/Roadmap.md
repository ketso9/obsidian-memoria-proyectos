# Roadmap y Tareas — Portal del Empleado

Estado de desarrollo, tareas prioritarias y backlog de características.

---

## 🎯 En Curso (Alta Prioridad)
- [x] Establecer conexión MCP con Obsidian y estructurar la memoria técnica.
- [ ] Validar el pipeline completo de firma PDF con la versión FPDI 2.1.7 (`ep-signature`).
- [ ] Revisar compatibilidad de endpoints REST y AJAX con la autenticación SSO de Microsoft 365.
- [ ] Retirar el mu-plugin `club1899-portal-sync.php` en producción y en `devpruebas` (duplica el envío de empresas hacia preproducción). Ver [[Proyectos/Portal del Empleado/Despliegue|Despliegue y Operativa]].
- [x] ~~Bot de Teams sin respuesta: diagnóstico~~ → **causa: Cloudflare descartaba el tráfico del canal Teams** (funciona con Cloudflare pausado, 2026-09-04 11:49).
- [ ] **Reactivar Cloudflare sin romper el bot:** (1) Seguridad → Bots → desactivar *Bot Fight Mode* (en plan Free no se puede eximir por regla); (2) Reglas → Reglas de configuración: para `portal.camaracaceres.com` y ruta `/wp-json/employee-portal/v1/teams-bot`, desactivar *Browser Integrity Check* y poner *Security Level* en "Essentially off"; (3) reactivar el proxy y escribir "hola" en Teams; confirmar en `ep_debug.log` una línea `EP Bot IN: POST` desde IP de ASN 8075 con UA `Microsoft-SkypeBotApi`. Si sigue fallando, alternativa: dejar `portal.camaracaceres.com` en *DNS only* (nube gris) y mantener el resto de subdominios proxied.
- [ ] Cloudflare → SSL/TLS: pasar de *Flexible* a *Full (strict)* (el origen sirve HTTPS válido; ahora el proxy habla HTTP con el servidor).

---

- [ ] **Bot de Teams sin respuesta (2026-09-04):** ninguna actividad del canal `msteams` llega al servidor (solo WebChat). Comprobar, en este orden: (1) en el registro del bot (Azure Bot / dev.botframework.com) → *Channels* → *Microsoft Teams*: que exista, esté habilitado y la pestaña *Issues* no muestre errores de entrega (un `403 Forbidden` ahí apunta a Cloudflare); (2) en Cloudflare → *Security* → *Events*, filtrar por ruta `/wp-json/employee-portal/v1/teams-bot` en las últimas 24 h; si hay bloqueos de IPs de Azure, crear una regla WAF de *Skip* para esa ruta; (3) volver a escribir "hola" y confirmar en `ep_debug.log` una línea `EP Bot IN: POST` con `channelid: msteams`.

## 📋 Backlog de Funcionalidades
### Módulo de Firma Electrónica (`ep-signature`)
- [ ] Verificación en frontend de firmas de certificados electrónicos cualificados.
- [ ] Automatización del sellado de tiempo (TSA) en documentos emitidos por la empresa.
- [ ] Interfaz de visualización de historial de firmas para el empleado (`[fds_mis_documentos]`).

### Módulo de Comunicación y Tickets (`ep-buzon` / `ep-avisos`)
- [ ] Notificaciones push o webhooks de Teams cuando se crea o actualiza un ticket.
- [ ] Filtros y búsqueda por departamento en el directorio de empleados (`ep-censo`).

### Integración M365 (`ep-calendar` / Graph)
- [ ] Sincronización bidireccional de ausencias/vacaciones con el calendario personal de Outlook.
- [ ] Mapeo de grupos de seguridad de Azure AD a roles de la suite.

---

## 📌 Decisiones Pendientes (ADR)
* Evaluar si delegar operaciones criptográficas pesadas a un microservicio backend o mantener 100% en Web Crypto API + PHP nativo.
