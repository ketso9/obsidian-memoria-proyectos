# Integración: Microsoft 365 y Graph API

**Archivos Clave:**
* `includes/class-ep-auth-o365.php`: Controlador de autenticación SSO OAuth 2.0.
* `includes/class-ep-graph-service.php`: Cliente de conexión y consultas a Microsoft Graph API.
* `includes/class-ep-teams-bot.php` / `class-ep-teams-webhook.php`: Integración con Microsoft Teams.
* `includes/class-ep-oof-sync.php`: Sincronización de estados Fuera de la Oficina (Out-of-Office).

---

## 1. Flujo de Autenticación SSO (Azure AD / Entra ID)
1. **Acceso Exclusivo:** El portal restringe el acceso al formulario tradicional de WordPress. La autenticación se realiza mediante el flujo Authorization Code Grant de OAuth 2.0 contra Azure AD.
2. **Provisionamiento de Usuario:**
   * Al autenticarse satisfactoriamente un usuario de M365, el sistema busca un usuario de WordPress con su `userPrincipalName` / `mail`.
   * Si no existe, se provisiona automáticamente en WordPress asignándole el rol correspondiente según sus grupos en Azure AD.
3. **Persistencia de Tokens:** Almacenamiento seguro del Access Token y Refresh Token para permitir llamadas offline o delegadas a Microsoft Graph en nombre del usuario.

---

## 2. Ámbitos (Scopes) y Servicios de Graph API
* `User.Read` / `User.Read.All`: Obtención de foto de perfil, puesto, departamento, teléfono y jerarquía de empleados para alimentar `ep-censo`.
* `Calendars.ReadWrite`: Consulta y sincronización de eventos de Outlook en la vista de `ep-calendar`.
* `MailboxSettings.Read`: Detección de mensajes de respuesta automática y estados de ausencia programada (`ep-oof-sync`).

---

## 3. Notificaciones y Webhooks (Teams)
* Notificación de creación de tickets de soporte o buzón interno (`ep-buzon`).
* Alertas de nuevos documentos pendientes de firma o avisos urgentes (`ep-avisos`) publicados en los canales correspondientes de Teams.
