# Next.js Clean Architecture Skill

Skill para Claude Code que organiza proyectos Next.js + TypeScript por **dominio → funcionalidad → responsabilidad**.

La arquitectura objetivo es `src/app/`, `src/domains/` y `src/shared/`. Sirve para proyectos nuevos, nuevas funcionalidades, diagnósticos y refactors graduales.

## Instalación en un proyecto

Cloná este repositorio en una carpeta de herramientas:

```sh
git clone https://github.com/arturogutierrez11/nextjs-clean-architecture-skill.git
```

Desde la raíz del proyecto destino, copiá la carpeta completa (reemplazá `/ruta/` por la ubicación del clon):

```sh
mkdir -p .claude/skills
cp -R /ruta/nextjs-clean-architecture-skill/.claude/skills/nextjs-clean-architecture .claude/skills/
```

Si ya existe una Skill con ese nombre, revisá las diferencias antes de reemplazarla.

## Uso en Claude Code

```text
/nextjs-clean-architecture Implementá el listado de productos.
```

```text
/nextjs-clean-architecture Analizá este repositorio sin modificar archivos.
```

```text
/nextjs-clean-architecture Refactorizá la creación de productos preservando su comportamiento.
```

## Contenido

La entrada es [SKILL.md](.claude/skills/nextjs-clean-architecture/SKILL.md). Sus referencias cubren arquitectura, carpetas, responsabilidades, naming, componentes, límites servidor/cliente, datos, estado, ejemplos, checklist y migración de inconsistencias.

Los [ejemplos](.claude/skills/nextjs-clean-architecture/examples.md) incluyen listado y detalle de productos, formulario validado, services cliente/servidor, endpoint, estado local y una corrección de código que incumple la arquitectura. Son ejemplos adaptables: la escritura sobre un catálogo real requiere integrar los permisos del proyecto.

## Reglas principales

- `app/` integra routing, layouts, metadata, endpoints y composición.
- `domains/` contiene las funcionalidades y sus responsabilidades.
- `shared/` contiene código realmente compartido.
- Mantener los límites cliente mínimos y proteger el código privado del servidor.
- Validar entradas en servidor y delegar desde endpoints/actions.
- Evitar duplicaciones, abstracciones innecesarias y carpetas vacías.
- Migrar proyectos existentes gradualmente, dentro del alcance autorizado.
