# Arquitectura y dependencias

## Regla de ubicación

Antes de crear un archivo, respondé: ¿qué dominio lo posee?, ¿qué funcionalidad lo necesita?, ¿qué responsabilidad cumple? Ubicalo en `src/domains/<domain>/<feature>/<responsibility>/` cuando esa responsabilidad amerite carpeta. Un tipo o helper pequeño de un solo consumidor puede permanecer en ese archivo.

Dominio: área estable del producto, como `products`. Funcionalidad: resultado reconocible, como `list-products`, `product-detail` o `create-product`. Responsabilidad: `components`, `hooks`, `services`, `schemas`, `types`, `actions` o `data`, solo cuando exista código correspondiente.

## Direcciones permitidas

```text
app → domains → shared
app ──────────→ shared
feature → su propio código
feature → shared de su dominio → shared global
```

- `shared` global nunca importa `domains` ni `app`.
- `domains` nunca importa implementación de `app`.
- El compartido del dominio nunca importa funcionalidades concretas.
- No importar detalles internos de una funcionalidad hermana. Compartir un contrato estable a nivel dominio cuando haya consumidores reales, o componer desde `app`.
- Una dependencia entre dominios debe ser explícita, acíclica y mediante un contrato pequeño; no acceder a componentes internos, stores o infraestructura privada de otro dominio. No mover negocio a `shared` para ocultar esa dependencia.
- No crear un `index.ts` que reexporte módulos cliente y servidor juntos. Preferir imports explícitos; una API pública pequeña puede exponer contratos neutrales por separado.

## Compartir con evidencia

Antes de mover algo, buscá consumidores. Dos componentes visualmente parecidos no necesariamente tienen el mismo contrato. Extraé a nivel funcionalidad primero, luego al dominio si se reutiliza allí, y al global si sirve a ámbitos distintos sin conocer sus reglas de negocio. Las primitivas existentes de un design system pueden mantenerse como compartidas.

## Decisión operativa

Para cada cambio no trivial, registrá en el resumen: propietario funcional, entrada desde `app`, entorno de ejecución y dependencias nuevas. Evitá documentación adicional si el código y el resumen ya hacen evidente la decisión.

Si una necesidad exige romper una regla, explicá la excepción concreta y su alcance. No rehagas la aplicación para alcanzar una estructura ideal fuera del pedido.
