# Responsabilidades

| Unidad | Debe hacer | No debe hacer |
|---|---|---|
| `app/**/page.tsx` | Leer parámetros, componer, invocar entrada funcional, metadata | Implementar consultas, reglas o formularios completos |
| Layout | Estructura, providers del alcance y composición | Resolver todas las funcionalidades o convertirse en cliente sin necesidad |
| Route Handler | Parsear/validar entrada, autenticar/autorizar si aplica, delegar, mapear HTTP | Contener SDKs y lógica funcional extensa o devolver excepciones crudas |
| Server Action | Validar entrada no confiable, verificar permisos, delegar, invalidar lo necesario | Duplicar otra implementación de la operación |
| Componente servidor funcional | Coordinar lectura y presentación de su funcionalidad | Exponer secretos o cargar estado de navegador |
| Componente cliente | Interacción, eventos, estado UI y render | Acceso directo a credenciales o infraestructura de servidor |
| Hook | Componer comportamiento React con ciclo de vida/estado | Ser un wrapper obligatorio de cualquier función pura |
| Service | Ejecutar una operación o acceso externo con contrato explícito | Importar componentes, hooks o decidir mensajes visuales |
| Schema | Validar datos en runtime | Leer entorno, hacer HTTP o importar módulos de servidor si lo usa el cliente |
| Tipo | Expresar un contrato estático | Sustituir validación de datos externos |
| Helper puro | Transformar entradas sin efectos | Ocultar requests, navegación o estado global |

Una operación de lectura sencilla puede vivir en un único service que consulta y valida. No exigir `repository`, `use-case`, `mapper` y `service` para el mismo fetch. Separarlos cuando haya reglas independientes, proveedores alternativos reales o una necesidad de pruebas que lo justifique.

Los adaptadores pueden realizar validación de transporte. Las invariantes del negocio deben cumplirse en la operación compartida para todos sus consumidores, usando el mismo schema cuando corresponda.

Si un service del servidor requiere identidad, recibí un contexto autenticado o verificá permisos en un punto servidor confiable. Nunca confiar en `userId`, roles o precios privilegiados enviados por el navegador.

Antes de extraer código, identificá qué cambiaría independientemente y quién lo consumiría. Si no hay respuesta concreta, mantené la implementación simple y local.
