# Roadmap y Tareas — Portal del Empleado

Estado de desarrollo, tareas prioritarias y backlog de características.

---

## 🎯 En Curso (Alta Prioridad)
- [x] Establecer conexión MCP con Obsidian y estructurar la memoria técnica.
- [ ] Validar el pipeline completo de firma PDF con la versión FPDI 2.1.7 (`ep-signature`).
- [ ] Revisar compatibilidad de endpoints REST y AJAX con la autenticación SSO de Microsoft 365.

---

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
