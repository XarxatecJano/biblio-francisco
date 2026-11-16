# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Estado del proyecto

Proyecto **greenfield**: por ahora el repositorio contiene **solo especificación y diseño**, aún
no hay código de aplicación ni toolchain (no existe `package.json`, `tsconfig.json`, ni suite de
tests). La primera fase de desarrollo consiste en montar el andamiaje del proyecto a partir de los
documentos de esta carpeta.

> El git root es esta carpeta `docs/`. Todo lo que se cree (código, tests, configuración) vive
> aquí salvo que se decida otra estructura.

## Documentos fuente (leer antes de programar)

Son la **fuente de verdad**. Ante cualquier duda de alcance o comportamiento, mandan estos docs:

- **`TECH_SPEC.md`** — stack, arquitectura MVC, esquema de base de datos y, de forma vinculante,
  la sección **«Metodología de trabajo (normas para Claude Code)»** (TDD y protección de tests).
- **`casos-de-uso.md`** — los 33 casos de uso con sus actores, objetivos y **excepciones**. Cada
  excepción es un test a escribir.
- **`sistema-de-diseno.md`** — sistema de diseño: tokens en variables CSS, componentes y reglas
  **mobile-first / responsive**. La capa *View* se construye consumiendo estos tokens.
- **`seedsql.sql`** — esquema PostgreSQL ejecutable (tablas, claves foráneas). Es el contrato de
  datos actual.

## Metodología de trabajo — OBLIGATORIA

Resumen operativo; el detalle completo y vinculante está en `TECH_SPEC.md` §«Metodología de trabajo».

- **TDD estricto (Red → Green → Refactor).** No se escribe código de producción sin un test que
  lo exija primero. Confirma que el test falla por el motivo esperado antes de implementar. Cada
  caso de uso se traduce en tests para el camino feliz **y cada excepción** listada.
- **🔴 Regla de oro — no tocar tests ya verificados.** **Nunca** modifiques, borres, deshabilites
  (`skip`/`only`), ni relajes una aserción de un test que ya pasó en verde **sin pedir permiso
  explícito al usuario antes**. Un test en verde es un contrato acordado.
  - *Sí* sin preguntar: añadir tests nuevos; editar un test que acabas de escribir en esta tarea y
    que todavía no ha pasado nunca (sigue en fase Red).
  - *Requiere permiso*: cualquier cambio a un test que ya estaba verde. Si crees que «necesitas»
    cambiarlo para que el código encaje, **detente y consulta** — no lo cambies y avises después.
  - Al consultar, sé **didáctico**: explica qué test es y qué protege, por qué debe cambiar
    (distingue «cambió el requisito» de «el código nuevo no cumple»), el diff propuesto, las
    consecuencias y las alternativas sin tocar el test. Espera confirmación explícita.

## Comandos

Aún no hay toolchain de build/lint/test. El único comando operativo documentado es la carga del
esquema en PostgreSQL:

```bash
# Inicializar el esquema (desde la raíz del proyecto, que es docs/)
psql -U <usuario> -d <base_datos> -f seedsql.sql

# Verificar tablas
psql -U <usuario> -d <base_datos> -c "\dt"
```

> **Cuando se inicialice el proyecto** (gestor de paquetes, runner de tests, build de TypeScript),
> **actualiza esta sección** con los comandos reales de build, lint, test y «ejecutar un solo
> test». No los inventes antes de que existan.

## Arquitectura

**Stack:** HTML5 + CSS3 + TypeScript en frontend y backend, PostgreSQL como base de datos.
Patrón **MVC**:

- **Model** — acceso a datos y lógica de negocio sobre PostgreSQL.
- **View** — HTML + CSS consumiendo los tokens de `sistema-de-diseno.md`.
- **Controller** — TypeScript que coordina vistas y modelos.

**Dos superficies de UI** (misma identidad, distinto layout y permisos):
- **Público (OPAC)** — catálogo consultable sin identificarse. Actores: *Usuario no registrado*, *Socio*.
- **Gestión (back-office)** — panel con sidebar. Actores: *Bibliotecario*, *Admin*.

El **rol** (`User.role`) condiciona qué se ve y qué se puede hacer; la excepción transversal «el
usuario no tiene permisos» aparece en casi todos los casos de uso y debe reflejarse tanto en la
lógica (autorización) como en la UI (acción bloqueada visible, no error silencioso).

**Modelo de dominio** (ver `seedsql.sql`): `User`, `Item` (catálogo), `Loan` (préstamos),
`Reservation`, `Request` (solicitudes/adquisiciones), `Sanction` (multas/suspensiones),
`Writer`, `Genre`, las relaciones N:M `Item_writer_rel` / `Item_genre_rel`, y `Audit_log`
(trazabilidad de cambios — requisito del CU 33). `Loan`, `Request` y otras tablas registran
`managed_by` / `created_by` / `updated_by` para auditar quién gestiona cada operación.

**Desajustes conocidos esquema ↔ diseño** (resolver con el usuario antes de dar por cerrada la
capa de datos; no asumir en silencio):
- Los mockups modelan **ejemplares por sede** (Central / Norte / Playa) con códigos de barras
  (`BM-00129x`), pero el esquema actual no tiene tabla `Sede`/sucursal, ni entidad «ejemplar»
  separada del título, ni campo de código de barras. `Item` tiene una sola `ubication`.
- `role`, `status` y `ubication` son `bigint` (ids de catálogos/enumeraciones) cuyas tablas de
  referencia no están en `seedsql.sql`.
