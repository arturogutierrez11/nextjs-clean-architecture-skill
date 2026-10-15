# Estado

## Elegir propietario antes de herramienta

| Necesidad | Ubicación inicial |
|---|---|
| Acordeón, cantidad, modal, tab temporal | `useState` del componente cliente |
| Estado compartido entre hermanos | Padre común más cercano |
| Filtro/paginación que debe sobrevivir navegación o compartirse por enlace | URL, con validación de parámetros |
| Campos, errores y envío de formulario | Formulario local o librería ya instalada |
| Datos remotos con caché/refetch | Servidor o librería de estado remoto existente |
| Dependencia contextual de un subárbol | Context/provider de ese alcance |
| Estado cliente transversal complejo | Store, solo con consumidores y ciclo de vida definidos |

No duplicar datos remotos en un store solo para poder leerlos. No almacenar valores que pueden derivarse directamente de otros estados. No sincronizar estado local y URL con efectos bidireccionales sin definir una única fuente de verdad.

Extraer un hook cuando encapsule comportamiento React significativo o reutilizado. Una selección local de cantidad puede permanecer en el componente. Un hook funcional vive en la funcionalidad, no en `shared/hooks` por defecto.

Providers y stores de negocio pertenecen al dominio/funcionalidad. Si son necesarios, crear `context/` o `stores/` allí; no crear esas carpetas vacías ni convertirlas en nuevos niveles obligatorios.

En SSR, no usar un singleton mutable con datos de usuario compartido entre requests. Definir inicialización, hidratación, limpieza al cerrar sesión y persistencia según las necesidades reales. No persistir credenciales en un store de UI.

Durante lecturas disparadas por efectos, manejar cancelación/carreras y desmontaje. Durante mutaciones, manejar pending/error/success y preservar los campos si falla. Usar el estado de envío de la librería de formulario si ya lo resuelve, en lugar de duplicarlo.
