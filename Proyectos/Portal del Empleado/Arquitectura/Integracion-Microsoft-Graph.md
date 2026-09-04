# Integración: Microsoft 365, Teams Bot y Graph API

**Archivos Clave:**
* `includes/class-ep-auth-o365.php`: Controlador de autenticación SSO OAuth 2.0.
* `includes/class-ep-graph-service.php`: Cliente de conexión y consultas a Microsoft Graph API.
* `includes/class-ep-teams-bot.php`: Envío proactivo de notificaciones y gestión de canales (Graph y Bot Framework).
* `includes/class-ep-bot-mensajeria.php`: Endpoint webhook para mensajes entrantes de Teams y generador de Adaptive Cards.
* `includes/class-ep-ai-service.php`: Integración con Google Gemini API (`gemini-3.1-flash-lite-preview`) para comprensión del lenguaje natural (NLU).
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
* `TeamsActivity.Send`: Envío de notificaciones al feed/campana de Teams (requiere manifest ID del catálogo organizativo).

---

## 3. Arquitectura del Bot de Microsoft Teams y Bot Framework

### 3.1. Identificadores y Credenciales
* **Bot ID (Microsoft App ID):** `dfcd7250-abfd-4689-8ad4-a5163898de14`
* **Tenant ID:** `a853cfe5-f96e-4342-99af-95f72c72dea7` (Configurado como Single-Tenant en Azure App Registration y `dev.botframework.com`).
* **App de Teams (Manifiesto):** ID externo `954318d3-5750-426e-b365-74ea5c53fcf1`, versión actual `1.4.5`.
* **Catálogo Organizativo Teams:** ID interno `e77c3d1b-f248-4e83-b2ec-454fb9b45f6c`.
* **Messaging Endpoint:** `https://portal.camaracaceres.com/wp-json/employee-portal/v1/teams-bot`

### 3.2. Pipeline de Envío Proactivo (Servidor → Teams)
1. Obtención de token de aplicación vía `https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token` con scope `https://api.botframework.com/.default`.
2. El bot intenta el envío al feed (`sendActivityNotification`) y conmuta de forma resiliente a **Chat 1:1 vía Bot Framework** si el feed responde 403.
3. Enrutamiento regional automático: Si la llamada al endpoint global `smba.trafficmanager.net/v3` responde 404 (*Conversation not found*), el sistema reintenta y entrega con éxito en el endpoint de Europa **EMEA** (`https://smba.trafficmanager.net/emea/v3/conversations/...`).

### 3.3. Pipeline de Mensajes Entrantes (Teams → Servidor → Gemini)
1. Teams entrega el payload al conector de Bot Framework.
2. Bot Framework despacha un HTTP `POST` firmado con JWT a `/wp-json/employee-portal/v1/teams-bot`.
3. **Validación Criptográfica:** `EP_Bot_Mensajeria::validar_token_microsoft` valida:
   * Emisor (`iss`): `https://api.botframework.com` o endpoints del tenant.
   * Audiencia (`aud`): Bot ID o Client ID de Teams.
   * Caducidad y activación (`exp`, `nbf`).
   * Firma digital RSA (RS256/RS512) contra los JWKS oficiales de Microsoft cacheando claves por `kid`.
4. **Resolución de Intención con Gemini:**
   * Si el mensaje es un saludo directo (`hola`, `buenos días`), genera la tarjeta adaptativa de bienvenida con datos del día (firmas pendientes, citas de Outlook, tickets y tareas).
   * En cualquier otra consulta en lenguaje natural, delega en `EP_AI_Service::get_intent()` con el modelo `gemini-3.1-flash-lite-preview` para clasificar intenciones (`AGENDA`, `DOCUMENTS`, `TICKETS`, `INVENTORY`, `DIRECTORY`, `CENSO`, etc.) y responde con Adaptive Cards interactivas v1.4.