# Portal del Empleado — Memoria del Proyecto

**Estado:** En desarrollo activo  
**Entorno:** WordPress Plugin Monolítico Modular / PHP / MySQL / JS  
**Integración:** Microsoft 365 (Graph API, SSO Azure AD, Teams Webhook)  
**Bóveda de Conocimiento:** [[Proyectos/Portal del Empleado/Index|Portal del Empleado]]

---

## 📌 Navegación Rápida
* 📓 **Diario de Desarrollo:** [[Proyectos/Portal del Empleado/Bitacora|Bitácora de Sesiones]]
* 🗺️ **Planificación y Estado:** [[Proyectos/Portal del Empleado/Roadmap|Roadmap y Tareas]]
* 📐 **Arquitectura Técnica:**
  * [[Proyectos/Portal del Empleado/Arquitectura/Arquitectura-PDF-Signature|Módulo de Firma PDF (ep-signature)]]
  * [[Proyectos/Portal del Empleado/Arquitectura/Integracion-Microsoft-Graph|Integración con Microsoft 365 / Graph API]]
  * [[Proyectos/Portal del Empleado/Arquitectura/Estructura-Submodulos|Arquitectura de Submódulos e Interfaz]]

---

## 1. Stack Técnico y Arquitectura Base
* **Core:** Clases controladoras orientadas a objetos con namespace/prefijo `EP_` (`EP_Admin`, `EP_Deployer`, `EP_Security`, `EP_Hardening`).
* **Plataforma:** WordPress (Plugin monolítico extensible mediante submódulos internos desacoplados en `plugins/`).
* **Frontend:** JavaScript Vanilla, vistas parciales (`partials/`) y CSS modular por aplicación. Interacción 100% desde Frontend (sin acceso a `wp-admin` por parte del empleado).
* **Autenticación e Identidad:** Single Sign-On (SSO) exclusivo mediante Azure AD / OAuth 2.0 (`class-ep-auth-o365.php`) y Microsoft Graph API (`class-ep-graph-service.php`).
* **Notificaciones & Bots:** Microsoft Teams Webhook y Bot (`class-ep-teams-bot.php`, `class-ep-teams-webhook.php`), sincronización Out-of-Office (`class-ep-oof-sync.php`).
* **Asistencia IA:** Servicio interno (`class-ep-ai-service.php`) y contexto de mensajería conversacional.

---

## 2. Submódulos Activos (`plugins/`)
Todos los módulos implementan `interface-ep-app.php` y se registran dinámicamente mediante `EP_App_Manager`:

| Módulo | Responsabilidad | Componentes Clave |
| :--- | :--- | :--- |
| **`ep-signature`** | Firma digital, CSV, manipulación y estampado en PDFs | FPDI 2.1.7, TCPDF, QR Code, Web Crypto API |
| **`ep-censo`** | Directorio / censo de empleados y jerarquías | Importador XLSX (`SimpleXLSX`), CPT, sincronización |
| **`ep-expenses`** | Liquidaciones y control de gastos de personal | DB personalizada, adjuntos y exportación XLSX |
| **`ep-calendar`** | Turnos, eventos y sincronización con Outlook | Microsoft Graph Calendar, vistas de calendario |
| **`ep-contratos`** | Visualización y gestión documental de contratos | Templates frontend, repositorio seguro |
| **`ep-inventory`** | Hardware y activos asignados a empleados | CPT, generación de actas de entrega/devolución PDF |
| **`ep-buzon`** | Sistema de tickets internos y solicitudes | Formularios frontend, notificaciones por email |
| **`ep-avisos`** | Comunicados internos y noticias corporativas | Templates, avisos urgentes destacados |
| **`ep-empresas`** | Gestión multi-empresa y branding | Entidades, logotipos, asignación de sedes |
| **`ep-downloads`** | Repositorio documental y descarga de nóminas | Control de acceso por roles y tareas programadas |
| **`ep-gdpr`** | Cumplimiento normativo y gestión de bajas | Anonimización y auditoría de datos |
| **`ep-links`** | Hub de herramientas y accesos rápidos corporativos | Enlaces dinámicos según permisos del empleado |

---

## 3. Directivas Esenciales para el Agente (Antigravity)
1. **Frontend-Only:** Cualquier funcionalidad para el empleado final debe resolverse en frontend (shortcodes, endpoints AJAX o REST).
2. **Contrato de Submódulos:** Todo nuevo módulo debe colocarse en `plugins/<nombre>` e implementar obligatoriamente `interface-ep-app.php`.
3. **Manejo de PDFs:** Se debe mantener la compatibilidad del pipeline **FPDI 2.1.7 + TCPDF** en `ep-signature/libs/`.
4. **Seguridad:** Sanitización estricta, verificación de nonces y comprobación de permisos mediante `class-ep-security.php` y `class-ep-roles.php`.
