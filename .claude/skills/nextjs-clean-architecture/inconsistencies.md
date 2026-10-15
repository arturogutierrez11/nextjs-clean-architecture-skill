# Detección y migración de inconsistencias

## Inspección operativa

Buscar rutas, directivas, fetch/SDKs, schemas, providers y consumidores en el alcance. Comandos orientativos, adaptados al repo:

```sh
rg --files src app pages
rg -n 'use client|use server|server-only' src
rg -n 'fetch\(|axios|createContext|useState|process\.env' src
```

Omitir directorios ausentes; no interpretar un exit code de búsqueda sin coincidencias como error de arquitectura. Leer el contexto de cada resultado antes de clasificarlo.

| Señal | Acción |
|---|---|
| Página con formulario, fetch y schema | Extraer a funcionalidad; conservar página como composición |
| `shared/services/product-service.ts` usado por una sola feature | Mover al dominio/feature y actualizar consumidores |
| Cliente importando SDK privado | Corregir límite inmediatamente; crear entrada servidor adecuada |
| Carpetas globales gigantes | Migrar archivos por propietario funcional, no por extensión |
| API y action con lógica idéntica | Extraer operación compartida; conservar entradas solo si tienen consumidores |
| Tipos duplicados | Inferir del schema o compartir contrato en el ámbito mínimo |
| Abstracción con muchos flags y un solo uso | Simplificar localmente, preservando comportamiento |
| Nombres iguales para entidad y componente | Separar significado con nombres explícitos |
| Todo marcado cliente | Identificar interacciones y bajar límites sin romper props |
| Documentación que contradice código | Actualizar solo la sección afectada si está en alcance |

## Priorizar

Primero fallos funcionales y límites de seguridad; luego contratos y dependencias; luego ubicación/naming. No convertir una preferencia cosmética en bloqueo de producción. Distinguir ausencia de evidencia de ausencia comprobada: revisar consumidores indirectos antes de eliminar una dependencia.

## Migración por funcionalidad

1. Registrar entrada pública, consumidores y comportamiento esperado.
2. Elegir destino y mover una porción coherente, actualizando imports.
3. Conservar temporalmente un adaptador de compatibilidad solo si hay consumidores que lo necesitan; indicar qué permite retirarlo.
4. Verificar tipos, lint y comportamiento afectado.
5. Eliminar duplicación anterior solo cuando no tenga consumidores; revisar el diff.

No mezclar actualización de Next, cambio de router, rediseño visual y reorganización total sin autorización de ese alcance. La estructura objetivo es obligatoria para el diseño nuevo, pero la deuda existente se migra gradualmente y se informa, no se oculta ni se arregla toda incidentalmente.

Formato útil de hallazgo: `archivo → responsabilidad actual → problema concreto → destino/corrección → verificación`. No inventar fallos por no tener auth, stores o loading si el producto no los necesita.
