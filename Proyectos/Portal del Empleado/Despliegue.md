# Despliegue y Operativa — Portal del Empleado

Criterios acordados con el usuario para subir cambios a los entornos. Aplican a
**todos los agentes** que trabajen en el proyecto.

---

## 1. Regla principal: despliegue selectivo por fichero

Para arreglos puntuales se suben **solo los ficheros modificados**, mediante `scp`.
No se usan `deploy.ps1` ni `deploy-staging.ps1`.

**Motivo:** esos scripts empaquetan con `tar` todo el árbol de trabajo, así que
arrastran a producción cambios sin commitear de otras apps y toda la diferencia de
la rama activa respecto a `main` (han llegado a ser más de 20.000 líneas). El
usuario eligió explícitamente el despliegue selectivo cuando se le plantearon
ambas opciones.

## 2. Procedimiento

1. Comparar el `md5` del fichero remoto con el de la versión base local, para
   confirmar que no se pisa trabajo ajeno.
2. Copia de seguridad remota en `~/backups_avisos/<timestamp>/{prod,staging}/`.
3. Subir con `scp`. Ojo: el `scp` nuevo usa SFTP, y en rutas con espacios **no**
   hay que poner comillas internas.
4. Verificar después con `md5` y `php -l` remoto.

## 3. Rutas de los entornos

| Entorno | Ruta |
| :--- | :--- |
| Producción | `portal.camaracaceres.com/wp-content/plugins/Portal-empleado-1` |
| Staging | `portal.camaracaceres.com/devpruebas/wp-content/plugins/portal del empleado` |

---

## 4. Tarea pendiente: retirar el mu-plugin `club1899-portal-sync.php`

**Completado el 2026-09-04.** Se retiró `wp-content/mu-plugins/club1899-portal-sync.php` renombrándolo a `.retirado` tanto en producción como en `devpruebas`.

**Motivo histórico:** el conector nuevo de `ep-empresas` (desplegado el 2026-09-02) ya hace el push al club vía `sync_to_club()` hacia `club1899.es`. El mu-plugin antiguo empujaba duplicadamente en `shutdown` hacia `camaracaceres.com/prepro`. Su desactivación limpia el pipeline.
