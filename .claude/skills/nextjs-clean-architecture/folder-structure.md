# Estructura de carpetas

## Objetivo ilustrativo

```text
src/
├── app/
│   ├── layout.tsx
│   ├── products/
│   │   ├── page.tsx
│   │   ├── loading.tsx
│   │   ├── error.tsx
│   │   └── [productId]/page.tsx
│   └── api/products/route.ts
├── domains/
│   └── products/
│       ├── list-products/
│       │   ├── components/
│       │   └── services/
│       ├── product-detail/
│       │   ├── components/
│       │   └── services/
│       ├── create-product/
│       │   ├── components/
│       │   ├── schemas/
│       │   └── services/
│       └── shared/
│           └── schemas/product.schema.ts
└── shared/
    ├── ui/
    ├── layout/
    └── config/
```

No generar este árbol íntegro. Cada carpeta requiere archivos que la tarea necesite. `loading.tsx` y `error.tsx` son ejemplos condicionados al flujo, no archivos obligatorios por ruta.

## Pasos para ubicar código

1. Determiná la funcionalidad por su propósito, no por el nombre de una pantalla ni por una tecnología.
2. Creá la responsabilidad concreta: un componente va en `components/`; un schema compartido cliente/servidor en `schemas/`.
3. Un tipo pequeño de props permanece dentro del componente. No crear `types/` para trasladar tres propiedades sin beneficio.
4. Datos estáticos propios permanecen en `data/` de la funcionalidad cuando separarlos mejora edición o lectura.
5. Tests se colocan junto al código probado, siguiendo el runner existente. Los E2E pueden permanecer en la carpeta convencional del proyecto.
6. Assets públicos permanecen en `public/`; no moverlos a `src` solo por uniformidad. Configuraciones del framework permanecen en la raíz cuando corresponda.

## Proyecto nuevo

Usar App Router y TypeScript cuando ese sea el alcance acordado. Alias recomendado: `@/*` → `./src/*`. Estilos globales pueden vivir en `app/globals.css`; tokens globales pueden extraerse a `shared/styles/` cuando haya una razón real. El CSS local acompaña a su componente.

## Proyecto existente

No mantener dos árboles de routing activos esperados (`app/` y `src/app/`) ni asumir que Next los combina. Si mover rutas excede el alcance, conservá temporalmente el router existente como adaptador hacia `src/domains`, documentá la desviación y evitá migrar framework y estructura en el mismo cambio sin necesidad.

No convertir Pages Router a App Router automáticamente. Preservá rutas y proponé la migración por separado si el usuario no la pidió.
