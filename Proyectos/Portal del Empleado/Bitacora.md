# Bitácora de Sesiones — Portal del Empleado

Registro cronológico de cambios, decisiones técnicas, migraciones y resoluciones de incidencias en el proyecto.

---

## 📅 2026-09-04 — Configuración de Memoria y Enlace con Obsidian MCP
* **Responsable:** Antigravity AI + Usuario
* **Hitos:**
  * Configuración del servidor MCP para Obsidian (`obsidian-local-rest-api`) utilizando el puerto HTTP `27123` en `~/.gemini/config/mcp_config.json`.
  * Verificación de la conexión y herramientas MCP (`vault_read`, `vault_write`, `vault_list`).
  * Estructuración del Vault de Obsidian como la memoria viva y documentación técnica permanente del proyecto.
  * Migración y reorganización de notas de proyecto en `Proyectos/Portal del Empleado/`.

---

## 📅 Hitos Previos (Histórico del Repositorio)
* **Rama activa:** `actualizacion/fpdi-pdf-parser-2.1.7`
* **Actualización del parser PDF:**
  * Actualización de la librería FPDI a la versión 2.1.7 compatible con versiones recientes de PDF y PHP 8.x.
  * Pruebas de compatibilidad en el pipeline de firma digital y estampado de códigos QR / CSV.
