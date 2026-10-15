# Convenciones de nombres

| Elemento | Convención | Ejemplo |
|---|---|---|
| Dominio y funcionalidad | `kebab-case`, propósito claro | `products/create-product` |
| Responsabilidad | Nombre plural convencional | `components`, `schemas`, `services` |
| Componente propio | Archivo y símbolo PascalCase | `CreateProductForm.tsx` |
| Hook | `use` + PascalCase en símbolo/archivo, `.ts` sin JSX | `useProductSearch.ts` |
| Service servidor | Operación + `.server.ts` | `get-product.server.ts` |
| Service cliente | Operación + `.client.ts` | `create-product.client.ts` |
| Schema | Concepto + `.schema.ts`; símbolo camelCase | `create-product.schema.ts`, `createProductSchema` |
| Tipo extraído | Concepto + `.types.ts`; símbolo PascalCase | `product.types.ts`, `Product` |
| Server Action | Operación + `.action.ts` | `create-product.action.ts` |
| Datos | Concepto + `.data.ts` | `product-filters.data.ts` |
| Helper puro | Operación en kebab-case | `format-price.ts` |
| Test | Convención existente del runner | `create-product.server.test.ts` |

Usá `.tsx` solo si hay JSX. Los sufijos de entorno comunican intención; `.server.ts` no reemplaza `import "server-only"`.

Conservá nombres especiales de Next (`page.tsx`, `layout.tsx`, `route.ts`, etc.). Sus componentes llevan nombres descriptivos como `ProductsPage`. Exports nombrados por defecto en módulos propios; default export donde lo exige el framework.

Un componente cliente no requiere sufijo adicional si contiene `"use client"` y su función es clara. Un par homónimo servidor/cliente puede usar nombres semánticos como `ProductDetail` y `QuantitySelector`.

Primitivas generadas por una librería pueden conservar `button.tsx` y sus exports. No renombrar código de terceros únicamente para homogeneizarlo.

Imports con `@/` para cruzar ámbitos; relativos cortos dentro del módulo. No usar recorridos profundos `../../../`. Constantes de configuración inmutables pueden usar UPPER_SNAKE_CASE; valores y colecciones normales, camelCase. No llamar `ProductCards` al tipo de un producto: distinguir entidad, DTO y props cuando tengan contratos diferentes.
