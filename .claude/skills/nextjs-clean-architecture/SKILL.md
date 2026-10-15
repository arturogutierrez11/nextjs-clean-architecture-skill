---
name: nextjs-clean-architecture
description: Organiza e implementa proyectos Next.js con TypeScript bajo src/app, src/domains y src/shared. Usar para crear funcionalidades, iniciar proyectos, revisar arquitectura o refactorizar módulos por dominio, funcionalidad y responsabilidad.
---

# Next.js Clean Architecture

Aplicá esta Skill al pedido concreto del usuario. Producí código y verificaciones cuando solicite implementar; entregá hallazgos con evidencia cuando solicite diagnóstico. No modifiques código en un pedido de solo análisis.

## Contrato obligatorio

- Estructura objetivo: `src/app/`, `src/domains/`, `src/shared/`.
- Organización funcional: **dominio → funcionalidad → responsabilidad**.
- `app/` integra routing, layouts, metadata, endpoints, estados de ruta y composición. No contiene la implementación del negocio.
- `domains/` contiene funcionalidades y sus componentes, lógica, acceso a datos y validación.
- `shared/` contiene solo código realmente compartido. No crear depósitos globales de hooks, services o tipos de negocio.
- Crear únicamente carpetas con contenido necesario. Los tipos pequeños pueden permanecer junto al consumidor.
- Mantener Server Components por defecto y límites cliente mínimos. No importar código exclusivo de servidor desde cliente.
- Validar entradas en servidor. Endpoints y actions delegan; no duplicar operaciones entre ambos.
- No introducir abstracciones ni dependencias para satisfacer un árbol ilustrativo.
- No ampliar el alcance de un refactor ni reemplazar decisiones explícitas del usuario. Explicar excepciones y migraciones pendientes.

## Procedimiento de trabajo

1. Inspeccioná instrucciones aplicables, `package.json`, lockfile, configuración de Next/TS, rutas y módulos afectados. Usá `rg --files` y búsquedas dirigidas. No leas secretos ni vuelques archivos `.env`.
2. Identificá la versión instalada, package manager, contratos públicos, consumidores y checks disponibles. No supongas defaults de caché o APIs del framework: verificá la versión y documentación oficial cuando haga falta.
3. Elegí modo: proyecto nuevo, funcionalidad nueva, refactor o diagnóstico. Definí dominio, funcionalidad, rutas afectadas y límite servidor/cliente antes de editar.
4. Leé [architecture.md](architecture.md), [folder-structure.md](folder-structure.md) y [responsibilities.md](responsibilities.md). Consultá las referencias específicas de la tabla siguiente.
5. Implementá una porción funcional completa: entrada, validación, operación, UI y estados relevantes. Reutilizá capacidades existentes que respeten los límites. No generes un esqueleto vacío.
6. Revisá imports y contratos; ejecutá los controles disponibles adecuados al cambio según [checklist.md](checklist.md). No afirmes que pasó un check que no ejecutaste.
7. Entregá qué cambió, árbol del alcance, decisiones relevantes, checks y limitaciones. En diagnóstico, indicá evidencia, impacto y corrección propuesta; no escribas archivos salvo pedido.

## Referencias por tarea

| Necesidad | Leer |
|---|---|
| Crear/mover archivos y exports | [naming-conventions.md](naming-conventions.md) |
| Diseñar UI o extraer componentes | [components.md](components.md) |
| Hooks, browser APIs, secretos o props entre entornos | [server-client-boundaries.md](server-client-boundaries.md) |
| Consultas, mutaciones, API routes, actions, schemas | [data-fetching.md](data-fetching.md) |
| Formularios, estado local, URL, context o stores | [state-management.md](state-management.md) |
| Implementación de referencia | [examples.md](examples.md) |
| Repo existente o refactor | [inconsistencies.md](inconsistencies.md) |

## Aplicación por modo

- **Nuevo:** creá los tres ámbitos a medida que tengan archivos; configurá el alias hacia `src`. No instales un stack de estado/formularios por anticipado.
- **Nueva funcionalidad:** ubicá un caso de uso en el dominio correcto, implementá solo las responsabilidades necesarias y conectalo desde `app`.
- **Existente/refactor:** inventariá consumidores y preservá URLs, contratos, comportamiento y cambios ajenos. Migrá por funcionalidad. No muevas un `app/` raíz a `src/app` parcialmente: planificá esa transición como una unidad cuando esté autorizada.
- **Diagnóstico:** diferenciá hechos observados, riesgos inferidos y propuestas. Usá la arquitectura objetivo para evaluar sin describirla como si ya existiera.

El nombre de la Skill describe modularidad y límites explícitos; no exige repositorios abstractos, inyección de dependencias, entidades o capas hexagonales para cada pantalla.
