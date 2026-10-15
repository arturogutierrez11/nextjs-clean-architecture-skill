# Ejemplos operativos

## Alcance y adaptación

Estos ejemplos muestran un catálogo genérico. No copiarlos íntegros si la tarea necesita solo una parte. Usan App Router, TypeScript, fetch nativo y Zod 4. Verificar las versiones instaladas antes de adaptar APIs. El alias `@/` apunta a `src/`. Instalar dependencias solo cuando la implementación autorizada las requiera.

El proveedor ilustrativo acepta `GET /products`, `GET /products/:id` y `POST /products`. Devuelve un array de productos, un producto o 404 según la operación. Adaptar sus DTOs al proveedor real; no asumir ese contrato en otra API.

`CATALOG_API_URL` y `CATALOG_API_TOKEN` son configuración privada requerida del servidor; documentar sus nombres, sin credenciales, en `.env.example`. Los ejemplos usan `no-store` por claridad, no como regla universal de caché.

El endpoint de creación es una demostración local del flujo, sin autenticación. **No conectarlo a un catálogo real con escritura privilegiada antes de integrar y verificar autenticación y autorización del proyecto.** No inventar un provider de auth: reutilizar el existente o resolver ese requisito. La Skill no prescribe una librería de autenticación.

Contenido: contratos y configuración; listado servidor; detalle y estado local; formulario validado; API desde cliente; endpoint y service; alternativa action; antes/después.

## 1. Contrato compartido por funcionalidades del mismo dominio

Aquí `Product` tiene consumidores reales en listado, detalle y creación. Por eso vive en el compartido de `products`, no en `src/shared/types`.

`src/domains/products/shared/schemas/product.schema.ts`

```ts
import { z } from "zod";

export const productSchema = z.object({
  id: z.string().min(1),
  name: z.string().min(1),
  description: z.string(),
});

export type Product = z.infer<typeof productSchema>;
```

La configuración tiene consumidores en los tres services del dominio, por eso se comparte a ese nivel.

`src/domains/products/shared/config/catalog.server.ts`

```ts
import "server-only";
import { z } from "zod";

const catalogConfigSchema = z.object({
  baseUrl: z.url(),
  token: z.string().min(1),
});

export function getCatalogConfig() {
  const result = catalogConfigSchema.safeParse({
    baseUrl: process.env.CATALOG_API_URL,
    token: process.env.CATALOG_API_TOKEN,
  });
  if (!result.success) throw new Error("Catalog configuration is invalid");
  return result.data;
}
```

No exponer los detalles de esta validación por HTTP ni mover el token a `NEXT_PUBLIC_`. La validación es diferida hasta usar el service, no necesita ejecutar requests durante imports.

## 2. Listar productos y consumir API desde servidor

`src/domains/products/list-products/services/list-products.server.ts`

```ts
import "server-only";
import { getCatalogConfig } from "../../shared/config/catalog.server";
import { productSchema } from "../../shared/schemas/product.schema";

export async function listProducts() {
  const { baseUrl, token } = getCatalogConfig();
  const response = await fetch(`${baseUrl.replace(/\/$/, "")}/products`, {
    headers: { Authorization: `Bearer ${token}` },
    cache: "no-store",
    signal: AbortSignal.timeout(8_000),
  });
  if (!response.ok) throw new Error("Products could not be loaded");
  const payload: unknown = await response.json();
  return productSchema.array().parse(payload);
}
```

`src/domains/products/list-products/components/ProductList.tsx`

```tsx
import Link from "next/link";
import { listProducts } from "../services/list-products.server";

export async function ProductList() {
  const products = await listProducts();
  if (products.length === 0) return <p>No hay productos disponibles.</p>;

  return (
    <ul>
      {products.map((product) => (
        <li key={product.id}>
          <Link href={`/products/${encodeURIComponent(product.id)}`}>
            {product.name}
          </Link>
        </li>
      ))}
    </ul>
  );
}
```

`src/app/products/page.tsx`

```tsx
import { ProductList } from "@/domains/products/list-products/components/ProductList";

export const metadata = { title: "Productos" };

export default function ProductsPage() {
  return (
    <main>
      <h1>Productos</h1>
      <ProductList />
    </main>
  );
}
```

No crear un `GET /api/products` solo para que este componente servidor lo llame.

`src/app/products/loading.tsx`

```tsx
export default function ProductsLoading() {
  return <p role="status">Cargando productos…</p>;
}
```

`src/app/products/error.tsx`

```tsx
"use client";

export default function ProductsError({ reset }: { reset: () => void }) {
  return (
    <div role="alert">
      <p>No pudimos cargar los productos.</p>
      <button type="button" onClick={reset}>Reintentar</button>
    </div>
  );
}
```

Este error boundary cliente pertenece a `app` por convención del framework. No vuelve cliente a la página. Adaptar su API si la versión instalada difiere.

## 3. Detalle: Server Component + Client Component y estado local

`src/domains/products/product-detail/services/get-product.server.ts`

```ts
import "server-only";
import { getCatalogConfig } from "../../shared/config/catalog.server";
import { productSchema } from "../../shared/schemas/product.schema";

export async function getProduct(productId: string) {
  const { baseUrl, token } = getCatalogConfig();
  const response = await fetch(
    `${baseUrl.replace(/\/$/, "")}/products/${encodeURIComponent(productId)}`,
    {
      headers: { Authorization: `Bearer ${token}` },
      cache: "no-store",
      signal: AbortSignal.timeout(8_000),
    },
  );
  if (response.status === 404) return null;
  if (!response.ok) throw new Error("Product could not be loaded");
  const payload: unknown = await response.json();
  return productSchema.parse(payload);
}
```

`src/domains/products/product-detail/components/QuantitySelector.tsx`

```tsx
"use client";

import { useState } from "react";

export function QuantitySelector({ max }: { max: number }) {
  const [quantity, setQuantity] = useState(1);

  return (
    <div>
      <button
        type="button"
        aria-label="Disminuir cantidad"
        disabled={quantity <= 1}
        onClick={() => setQuantity((value) => Math.max(1, value - 1))}
      >−</button>
      <output aria-live="polite">Cantidad: {quantity}</output>
      <button
        type="button"
        aria-label="Aumentar cantidad"
        disabled={quantity >= max}
        onClick={() => setQuantity((value) => Math.min(max, value + 1))}
      >+</button>
    </div>
  );
}
```

Contrato de este componente: `max` es un entero positivo estable durante su montaje. Si cambia por producto, usar identidad con `key` o definir explícitamente cómo ajustar la selección. Es estado visual: una compra real debe validar cantidad y disponibilidad en servidor. No crear un store ni `useQuantity` para este único consumidor.

`src/domains/products/product-detail/components/ProductDetail.tsx`

```tsx
import { notFound } from "next/navigation";
import { getProduct } from "../services/get-product.server";
import { QuantitySelector } from "./QuantitySelector";

export async function ProductDetail({ productId }: { productId: string }) {
  const product = await getProduct(productId);
  if (!product) notFound();

  return (
    <article>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <QuantitySelector key={product.id} max={10} />
    </article>
  );
}
```

`src/app/products/[productId]/page.tsx`

```tsx
import { ProductDetail } from "@/domains/products/product-detail/components/ProductDetail";

export default async function ProductPage({
  params,
}: {
  params: Promise<{ productId: string }>;
}) {
  const { productId } = await params;
  return <ProductDetail productId={productId} />;
}
```

`params` como Promise corresponde al App Router moderno; verificar la versión al integrar. Se puede agregar un `not-found.tsx` local si se necesita una presentación específica. El service retorna ausencia; la capa de composición decide la navegación.

## 4. Formulario: un schema para cliente y servidor

`src/domains/products/create-product/schemas/create-product.schema.ts`

```ts
import { z } from "zod";

export const createProductSchema = z.object({
  name: z.string().trim().min(2).max(100),
  description: z.string().trim().max(1_000),
}).strict();

export type CreateProductInput = z.infer<typeof createProductSchema>;
```

No duplicar `CreateProductInput` en una carpeta global. El schema es neutral: no importa configuración ni services.

## 5. Service cliente: consumo de API por evento

`src/domains/products/create-product/services/create-product.client.ts`

```ts
import { createProductSchema, type CreateProductInput } from "../schemas/create-product.schema";
import { productSchema } from "../../shared/schemas/product.schema";

export async function submitProduct(input: CreateProductInput) {
  const response = await fetch("/api/products", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(createProductSchema.parse(input)),
  });
  if (!response.ok) throw new Error("No se pudo crear el producto.");
  const payload: unknown = await response.json();
  return productSchema.parse(payload);
}
```

No importa el service servidor y no expone el token del proveedor. Este ejemplo produce un error público genérico; si el producto necesita errores por campo o conflictos, definir y validar un contrato discriminado, sin mostrar excepciones del proveedor.

`src/domains/products/create-product/components/CreateProductForm.tsx`

```tsx
"use client";

import { useState, type FormEvent } from "react";
import { createProductSchema } from "../schemas/create-product.schema";
import { submitProduct } from "../services/create-product.client";

export function CreateProductForm() {
  const [status, setStatus] = useState<"idle" | "pending" | "success" | "error">("idle");
  const [error, setError] = useState<string | null>(null);

  async function handleSubmit(event: FormEvent<HTMLFormElement>) {
    event.preventDefault();
    if (status === "pending") return;
    const form = event.currentTarget;
    const fields = new FormData(form);
    const result = createProductSchema.safeParse({
      name: fields.get("name"),
      description: fields.get("description"),
    });

    if (!result.success) {
      setError("Revisá los campos: nombre de 2 a 100 caracteres y descripción de hasta 1000.");
      setStatus("error");
      return;
    }

    setError(null);
    setStatus("pending");
    try {
      await submitProduct(result.data);
      form.reset();
      setStatus("success");
    } catch {
      setError("No se pudo crear el producto. Intentá nuevamente.");
      setStatus("error");
    }
  }

  return (
    <form onSubmit={handleSubmit} aria-busy={status === "pending"}>
      <fieldset disabled={status === "pending"}>
        <legend>Nuevo producto</legend>
        <label htmlFor="product-name">Nombre</label>
        <input id="product-name" name="name" required minLength={2} maxLength={100} />
        <label htmlFor="product-description">Descripción</label>
        <textarea id="product-description" name="description" maxLength={1_000} />
        <button type="submit">
          {status === "pending" ? "Guardando…" : "Crear producto"}
        </button>
      </fieldset>
      {error && <p role="alert">{error}</p>}
      {status === "success" && <p role="status">Producto creado.</p>}
    </form>
  );
}
```

Este formulario sencillo usa HTML y estado local. Para formularios complejos o si el proyecto usa React Hook Form, usar su resolver y `formState.isSubmitting`; no duplicar estado de envío. Añadir errores por campo cuando el flujo lo requiera.

`src/app/products/new/page.tsx`

```tsx
import { CreateProductForm } from "@/domains/products/create-product/components/CreateProductForm";

export default function NewProductPage() {
  return <CreateProductForm />;
}
```

## 6. Service servidor y endpoint que valida y delega

`src/domains/products/create-product/services/create-product.server.ts`

```ts
import "server-only";
import { createProductSchema, type CreateProductInput } from "../schemas/create-product.schema";
import { getCatalogConfig } from "../../shared/config/catalog.server";
import { productSchema } from "../../shared/schemas/product.schema";

export async function createProduct(input: CreateProductInput) {
  const validated = createProductSchema.parse(input);
  const { baseUrl, token } = getCatalogConfig();
  const response = await fetch(`${baseUrl.replace(/\/$/, "")}/products`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${token}`,
    },
    body: JSON.stringify(validated),
    signal: AbortSignal.timeout(8_000),
  });
  if (!response.ok) throw new Error("Product creation failed");
  const payload: unknown = await response.json();
  return productSchema.parse(payload);
}
```

`src/app/api/products/route.ts`

```ts
import { NextResponse } from "next/server";
import { createProductSchema } from "@/domains/products/create-product/schemas/create-product.schema";
import { createProduct } from "@/domains/products/create-product/services/create-product.server";

// Demostración local: integrar permisos reales antes de habilitar escritura real.
export async function POST(request: Request) {
  let body: unknown;
  try {
    body = await request.json();
  } catch {
    return NextResponse.json({ error: "INVALID_JSON" }, { status: 400 });
  }

  const result = createProductSchema.safeParse(body);
  if (!result.success) {
    return NextResponse.json({ error: "INVALID_INPUT" }, { status: 422 });
  }

  try {
    const product = await createProduct(result.data);
    return NextResponse.json(product, { status: 201 });
  } catch {
    return NextResponse.json({ error: "CREATE_FAILED" }, { status: 500 });
  }
}
```

La validación en la entrada permite mapear errores de transporte; la del service mantiene su contrato para otros consumidores. Es el mismo schema, no dos definiciones de reglas. En producción, distinguir errores esperados, permisos y fallos internos con un contrato deliberado y observabilidad segura. Un timeout no demuestra que una mutación no ocurrió: no añadir reintento automático; considerar idempotencia en operaciones que lo necesiten.

Comprobaciones de comportamiento al integrar: JSON inválido → 400; datos inválidos → 422 sin llamar al proveedor; permisos insuficientes → 401/403 según contrato; éxito → 201 y DTO válido; fallo externo → error público controlado. Usar mocks/adaptadores de prueba, no crear productos reales para comprobarlo.

## 7. Alternativa con Server Action, no un segundo flujo obligatorio

Si se elige Server Action para esta UI, reemplazar el transporte cliente y el endpoint cuando no tengan otros consumidores. Mantener `createProduct` y el schema. La action debe seguir este orden:

```text
autenticar y autorizar con la integración existente
→ convertir FormData a campos explícitos
→ safeParse(createProductSchema)
→ createProduct(datos validados)
→ invalidar datos afectados si estaban cacheados
→ devolver resultado serializable o redirigir
```

No envolver `redirect()` en un catch genérico que intercepte su señal de control. No crear un endpoint que simplemente llame a la action. Si ambas entradas son necesarias, ambas delegan al service y aplican sus controles de acceso. Este esquema no es código para copiar: requiere elegir la integración y versión reales.

## 8. Código que no cumple y cómo corregirlo

**Incorrecto:** `src/app/products/page.tsx` concentra estado, acceso HTTP y transformación, y obliga a descargar toda la pantalla como cliente.

```tsx
"use client";
import { useEffect, useState } from "react";

export default function ProductsPage() {
  const [products, setProducts] = useState<{ id: string; name: string }[]>([]);
  const [quantity, setQuantity] = useState(1);

  useEffect(() => {
    fetch("/api/products")
      .then((response) => response.json())
      .then(setProducts);
  }, []);

  return (
    <div>
      {products.map((product) => <p key={product.id}>{product.name}</p>)}
      <button onClick={() => setQuantity(quantity + 1)}>{quantity}</button>
    </div>
  );
}
```

Además de ubicación incorrecta, no comprueba HTTP, no valida respuesta y confunde carga con lista vacía. Mover el fetch a un hook global no resuelve el diseño.

**Corrección concreta:** usar la página y `ProductList` de la sección 2 para el listado inicial; el service valida la API en servidor. Colocar la interacción local en `QuantitySelector` de la sección 3 dentro de su detalle. La página compone; las funcionalidades poseen su comportamiento; el service es independiente de React.

Si el listado necesita actualización por interacción cliente, mantener el componente y su hook/service dentro de `list-products`, con cancelación/estado remoto según el stack. No forzar fetching servidor cuando el requisito es una interacción continua.

## 9. Cuándo extraer un hook o compartir

- Cantidad local de un detalle: conservar `useState` en el componente.
- Búsqueda con debounce, cancelación y varios eventos: un `useProductSearch.ts` dentro de `list-products/hooks` puede aportar claridad.
- `Product` usado por tres funcionalidades: compartir contrato en `products/shared` como en el ejemplo.
- Un `Button` genérico reutilizado en dominios distintos: `src/shared/ui`.
- Un componente solo parecido visualmente a otro: comprobar contrato antes de extraerlo.

Los ejemplos no necesitan carpetas globales de hooks, services o types ni una capa de repositorios abstractos. Un módulo de 20 líneas puede respetar todos los límites sin multiplicar archivos innecesariamente.
