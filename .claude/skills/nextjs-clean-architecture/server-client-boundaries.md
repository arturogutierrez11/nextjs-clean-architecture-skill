# Límites servidor y cliente

## Árbol de decisión

- ¿Necesita secreto, filesystem, DB o SDK privado? Módulo servidor con `import "server-only"`.
- ¿Necesita eventos, hooks de estado/efecto, DOM o APIs del navegador? Componente cliente con `"use client"` al principio del archivo.
- ¿Solo transforma datos? Módulo neutral, sin directiva.
- ¿Es una función invocable como Server Action? Usar `"use server"` siguiendo la versión de Next. No aplicar esa directiva a todos los services.

El grafo de imports de una entrada cliente también se incorpora al ámbito cliente. Un archivo sin directiva no es automáticamente seguro para importar allí. No importar un service servidor desde un hook cliente ni reexportarlo mediante un barrel compartido.

Un Client Component puede participar en el render inicial del servidor: no leer `window` en el nivel superior ni asumir que `"use client"` desactiva SSR. Acceder a APIs de navegador en eventos o efectos según corresponda.

## Composición permitida

Un padre servidor puede renderizar un hijo cliente y pasar contenido servidor como `children`. El hijo cliente no debe importar el componente servidor para ejecutarlo. Pasar DTOs mínimos serializables; callbacks ordinarios no cruzan ese límite. Las Server Actions son un mecanismo específico, no una excepción para funciones arbitrarias.

Colocar providers cliente tan abajo como su alcance lo permita. No marcar el root layout cliente solo para alojar un provider: componerlo desde el layout servidor.

Los services `.client.ts` usados desde componentes cliente deben ser seguros para navegador y no necesitan `"use client"` si no son entradas React. Usar `client-only` cuando realmente dependan de APIs exclusivas del navegador. Un fetch relativo dentro de una función llamada por un evento no requiere leer `window`.

## Revisión obligatoria

- Seguir imports desde cada nueva entrada cliente, incluidos reexports.
- Confirmar que schemas compartidos no importan configuración privada.
- Confirmar que props y respuestas no incluyen tokens, sesiones completas o campos privados.
- Verificar la API de parámetros, caché, actions y error boundaries según la versión instalada.

Referencia oficial: [Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components).
