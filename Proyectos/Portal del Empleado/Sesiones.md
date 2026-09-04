# Sesiones Activas - Tablón de Coordinación entre Agentes

Este vault lo comparten varios agentes (Claude Code y Gemini/Antigravity) **sin ningún
bloqueo técnico**: si dos escriben la misma nota a la vez, el último pisa al primero
sin avisar y sin dejar rastro.

Esta nota es la convención que lo evita. No la fuerza nada: depende de que todos los
agentes la respeten.

---

## Sesiones en curso

| Agente | Inicio | Trabajando en | Notas del vault que va a escribir |
| :--- | :--- | :--- | :--- |
| _(vacío)_ | | | |

## Protocolo

**Al empezar** una sesión de trabajo sobre el proyecto, añadir una fila a la tabla con:
el nombre del agente, la fecha y hora de inicio (`AAAA-MM-DD HH:MM`), una frase sobre
qué se está haciendo, y **qué notas del vault se piensan modificar**.

**Antes de escribir** en cualquier nota, leer esta tabla. Si otro agente tiene esa
misma nota declarada, **no escribir**: avisar al usuario de que hay un conflicto y
esperar su decisión. Coordinar es tarea del usuario, no de los agentes entre sí.

**Al terminar**, borrar la propia fila. Una sesión cerrada no deja rastro aquí; el
registro de lo que se hizo va en [[Proyectos/Portal del Empleado/Bitacora|la Bitácora]],
que es lo permanente.

**Filas caducadas.** Si una fila lleva más de 8 horas abierta, lo normal es que sea una
sesión que terminó de golpe sin cerrarse. Antes de retirarla, preguntar al usuario.

---

## Buenas prácticas de escritura

Aunque el tablón esté al día, hay una regla que reduce el daño de cualquier colisión:
**escribir por secciones, no reescribiendo el fichero entero.** Añadir al final o
parchear un encabezado concreto (`vault_append`, `vault_patch`) en lugar de un
`vault_write` completo. Así, dos escrituras simultáneas sobre partes distintas de una
misma nota pueden convivir en vez de destruirse.

Reservar la reescritura completa para cuando se sea el único agente activo.

---

## Limitaciones honestas

* Es una convención **voluntaria**. Un agente que no lea esta nota escribirá igual.
* No es tiempo real: entre que un agente empieza a trabajar y apunta su fila hay un hueco.
* No protege el código del repositorio, solo las notas del vault. Para los ficheros del
  plugin, la coordinación sigue siendo git y el criterio de
  [[Proyectos/Portal del Empleado/Despliegue|despliegue selectivo]].

---

Ver también: [[Proyectos/Portal del Empleado/Index|Índice del proyecto]]