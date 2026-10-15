# Componentes

## Al implementar

1. Partí de un componente servidor para contenido y lecturas.
2. Identificá exactamente qué interacción necesita estado, eventos o APIs del navegador.
3. Extraé esa interacción a un componente cliente local.
4. Pasá props serializables y mínimas. No enviar objetos completos de proveedor o datos privados por comodidad.
5. Separá una pieza visual cuando tenga responsabilidad clara, reutilización real o vuelva comprensible al padre; no por un límite arbitrario de líneas.

Primitivas visuales sin conocimiento del dominio pueden estar en `shared/ui`. Componentes como `ProductCard` pertenecen a una funcionalidad o al compartido del dominio si hay varios consumidores. Un componente usado una sola vez no va a `shared` por ser pequeño.

Formularios deben tener labels asociados, mensajes accesibles, estados pending/success/error y prevención de envíos duplicados mientras están pendientes. La validación de cliente no sustituye la del servidor. Mantener clases y estilos cerca de la UI; no introducir reglas funcionales dentro de tokens o variantes visuales.

Estados vacíos deben distinguirse de errores. Una lista de cero productos no es una falla HTTP. Usar keys estables de identidad cuando existan; no índices para listas reordenables.

Wrappers animados pueden recibir `children` producido desde un Server Component. Importar un wrapper cliente no exige hacer cliente al padre. Ver [server-client-boundaries.md](server-client-boundaries.md).

Evitá HTML inyectado cuando JSX sea suficiente. Para HTML externo, usar una estrategia explícita de sanitización apropiada al origen; no copiar un ejemplo de texto controlado a contenido de usuarios.

No extraer un componente universal lleno de flags para unificar pantallas con reglas distintas. Preferí composición con props pequeñas y elementos hijos.
