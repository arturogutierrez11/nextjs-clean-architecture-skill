# Datos, servicios y validación

## Lecturas

Preferí ejecutar en servidor las lecturas iniciales que no necesitan una interacción del navegador. La página compone un componente funcional o llama una entrada del dominio; el service contiene el acceso externo. Un Server Component debe llamar al service directamente, no hacer HTTP a su propio `/api` sin una razón de integración real.

Usá fetching cliente cuando la interacción lo requiera: filtros en vivo, polling o datos dependientes del navegador. Si el proyecto ya usa una librería de caché remota, conservá sus convenciones. No instalar una para una sola petición por evento.

Toda lectura externa debe manejar status HTTP y validar datos cuando sean un contrato no confiable. No reemplazar `parse` por `as Product`. Propagar resultados tipados, distinguir ausencia esperada de falla y evitar devolver objetos crudos de SDK.

## Caché

Elegí una política por dato: sin caché compartida para datos privados/personalizados salvo diseño explícito de aislamiento; revalidación para contenido público cuando sea aceptable. Verificá defaults de la versión instalada. No copiar `no-store` de los ejemplos a todas las lecturas ni aplicar caché pública global a todas las rutas. Invalidar únicamente datos afectados después de una mutación exitosa.

## Mutaciones

1. Elegí Route Handler si necesitás contrato HTTP, consumidores externos o una integración basada en endpoints. Elegí Server Action si encaja con la UI y el proyecto.
2. Parseá el formato y validá entrada en servidor; manejar JSON inválido separadamente.
3. Autenticá y autorizá cuando el recurso lo requiera. No asumir que una Server Action es privada por ser de servidor.
4. Delegá en el service funcional. Validar invariantes también allí si tiene múltiples entradas.
5. Mapeá resultados y errores a un contrato público estable. Logs internos sin credenciales ni datos sensibles.
6. Invalidá caché/navegá cuando corresponda y renderizá feedback.

No crear Route Handler y Server Action para la misma UI por costumbre. Si ambos tienen consumidores necesarios, comparten el service; una action no necesita llamar por HTTP al endpoint interno.

## Configuración y transporte

Guardar configuración privada validada en servidor; documentar nombres requeridos en `.env.example` sin valores reales. `NEXT_PUBLIC_` significa exposición al navegador, no un mecanismo para arreglar imports.

Para un proveedor compartido, usar un adaptador pequeño en `shared` solo cuando haya consumidores reales. URLs provienen de configuración confiable, no de una URL arbitraria enviada por el cliente. Codificar segmentos dinámicos.

Definir timeouts razonables donde aplique. No reintentar mutaciones automáticamente sin analizar idempotencia. Cancelar lecturas cliente obsoletas o usar la librería existente para evitar carreras. Nunca lanzar envíos reales como parte de un test sin autorización.

Los ejemplos de [examples.md](examples.md) ilustran fetch nativo y Zod; adaptar a la infraestructura instalada sin introducir esas dependencias automáticamente.
