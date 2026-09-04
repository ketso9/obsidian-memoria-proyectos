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

## 📅 2026-09-04 — Claude Code se suma al vault: memoria compartida entre agentes
* **Responsable:** Claude Code (Opus 5) + Usuario
* **Contexto:** el vault estaba enlazado únicamente con Gemini/Antigravity. El objetivo de esta sesión era que **ambos agentes compartan la misma memoria**.
* **Hitos:**
  * Verificado el servidor `obsidian-local-rest-api` v5.1.0 desde Claude Code: responde en `127.0.0.1:27123`, el handshake MCP en `/mcp/` negocia `protocolVersion 2025-06-18` con capacidades `tools` y `resources`.
  * Detectado que la configuración vivía solo en `~/.gemini/config/mcp_config.json`; en el lado de Claude, `mcpServers` estaba vacío.
  * Creado `CLAUDE.md` en la raíz del repositorio como espejo de `.agent/rules/obsidian_memory.md`, para que ambos agentes sigan **el mismo protocolo** de consulta y actualización del vault.
  * Migradas al vault las dos notas que Claude mantenía en su memoria local: criterio de **despliegue selectivo por fichero** y la retirada pendiente del mu-plugin `club1899-portal-sync.php`, ahora en [[Proyectos/Portal del Empleado/Despliegue|Despliegue y Operativa]].
* **Decisión:** el vault es la **única fuente de verdad**. La memoria local de cada agente queda reducida a un puntero hacia aquí, para evitar que las dos se desincronicen.
* **Nota operativa:** el plugin solo escucha mientras Obsidian está abierto. Si un agente no alcanza el vault, comprobar eso antes que la configuración.

---

**Ampliación (misma sesión) — Coordinación entre agentes.** Verificada la conexión MCP nativa desde Claude Code tras reiniciar: las 16 herramientas del vault responden. Detectado el riesgo real del montaje compartido: **el vault no tiene bloqueo de escritura**, así que dos agentes editando la misma nota a la vez se pisan sin dejar rastro ni aviso. Como mitigación se crea [[Proyectos/Portal del Empleado/Sesiones|Sesiones activas]], un tablón donde cada agente declara al empezar qué notas va a tocar y lo retira al terminar, y que hay que consultar antes de escribir. La directiva se ha añadido a los dos protocolos espejo (`CLAUDE.md` y `.agent/rules/obsidian_memory.md`), junto con la norma de preferir escrituras por sección frente a reescribir la nota completa. Es una convención voluntaria: no la fuerza nada técnicamente.

## 📅 Hitos Previos (Histórico del Repositorio)
* **Rama activa:** `actualizacion/fpdi-pdf-parser-2.1.7`
* **Actualización del parser PDF:**
  * Actualización de la librería FPDI a la versión 2.1.7 compatible con versiones recientes de PDF y PHP 8.x.
  * Pruebas de compatibilidad en el pipeline de firma digital y estampado de códigos QR / CSV.
