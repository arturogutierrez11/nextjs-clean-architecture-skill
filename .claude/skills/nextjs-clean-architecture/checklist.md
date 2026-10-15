# Checklist de implementación y revisión

Usar como control del alcance modificado. Marcar cada punto como pasa, falla o no aplica; no crear archivos solo para completar casillas.

## Arquitectura

- [ ] Dominio y funcionalidad identificados por propósito.
- [ ] Código funcional dentro de `src/domains/<domain>/<feature>`.
- [ ] `app` contiene adaptadores y composición, sin implementación funcional extensa.
- [ ] Ninguna carpeta vacía ni agregado global innecesario de hooks/services/types.
- [ ] Tipos pequeños cercanos y código compartido con consumidores reales.
- [ ] Imports sin ciclos, sin dependencia de shared hacia dominios y sin acceso a internals hermanos.
- [ ] Naming y alias consistentes con la estructura objetivo.

## Entornos y datos

- [ ] Límite `use client` mínimo; servidor no entra en grafo cliente por reexports.
- [ ] Services de servidor protegidos, configuración privada fuera de cliente.
- [ ] Entrada externa validada en servidor; JSON inválido manejado.
- [ ] Permisos aplicados cuando corresponden; DTO público mínimo.
- [ ] HTTP status y datos de respuesta tratados; errores no exponen proveedor/secretos.
- [ ] API Route y Server Action no duplican operación.
- [ ] Política de caché y actualización acorde al dato y versión.
- [ ] Carreras de lecturas cliente y reintentos de mutaciones considerados si aplican.

## UI y estado

- [ ] Estados de carga, error, vacío y éxito relevantes.
- [ ] Labels accesibles, feedback y prevención de doble envío pendiente.
- [ ] Estado en el propietario mínimo, sin duplicación innecesaria de datos derivados/remotos.
- [ ] Rutas, contratos y comportamiento previo preservados en refactors.

## Verificación real

1. Detectar scripts existentes; usar el package manager del lockfile.
2. Ejecutar typecheck, lint y comprobación de formato aplicables. No usar `--fix` fuera del alcance.
3. Ejecutar pruebas existentes relacionadas y añadir pruebas de comportamiento cuando el cambio lo necesite: payload inválido, permisos, errores de proveedor, éxito y estados UI.
4. Para cambios de routing/límites/compilación, ejecutar build si el entorno permite hacerlo; no sustituye pruebas de comportamiento. Documentar requerimientos o bloqueos reales.
5. Inspeccionar diff y consumidores tras movimientos. No borrar archivos ajenos ni correr mutaciones externas como verificación.

No introducir un runner nuevo para una corrección trivial ni tests que solo repitan la estructura del código. No afirmar que TypeScript demuestra el funcionamiento de un endpoint.

## Salida final

Informar cambios, ubicación funcional, verificaciones con resultados y cualquier desviación pendiente. Para auditorías, reportar evidencia, impacto, propuesta y prioridad. Distinguir revisión estática de ejecución real.
