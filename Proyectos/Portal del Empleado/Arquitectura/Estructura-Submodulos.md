# Arquitectura: Estructura de Submódulos

**Ubicación:** `plugins/`  
**Interfaz Obligatoria:** `includes/interfaces/interface-ep-app.php`  
**Gestor Central:** `includes/class-ep-app-manager.php`

---

## 1. Patrón de Diseño Modular
La suite "Portal del Empleado" funciona como un plugin núcleo (Core) que actúa como orquestador de micro-aplicaciones internas. Cada subdirectorio dentro de `plugins/` (`ep-signature`, `ep-expenses`, `ep-censo`, etc.) es una unidad autónoma con su propio ciclo de vida.

---

## 2. Contrato de la Aplicación (`interface-ep-app.php`)
Cualquier nuevo módulo debe implementar los métodos del contrato básico:

```php
interface EP_App_Interface {
    /**
     * Identificador único del módulo (slug)
     */
    public function get_id(): string;

    /**
     * Nombre legible del módulo para menús y paneles
     */
    public function get_name(): string;

    /**
     * Versión actual del módulo
     */
    public function get_version(): string;

    /**
     * Inicialización de hooks, shortcodes y endpoints
     */
    public function init(): void;

    /**
     * Renderizado de la vista principal en frontend
     */
    public function render_frontend(): string;
}
```

---

## 3. Registro y Carga en `EP_App_Manager`
* El gestor central escanea el directorio `plugins/` y registra las instancias de cada módulo.
* Controla la activación condicional de submódulos según licencias, roles de usuario o flags de configuración en el panel de administración.
* Encola únicamente los assets (CSS/JS) del módulo en aquellas páginas donde se renderiza su shortcode correspondiente, evitando sobrecargar el frontend.
