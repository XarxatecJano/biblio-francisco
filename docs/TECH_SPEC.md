# Especificación Técnica - App Activa Biblioteca

## Tecnologías

### Frontend
- **HTML5**: Estructura y marcado semántico
- **CSS3**: Estilos y diseño responsivo
- **TypeScript**: Lógica de cliente con tipado estático

### Backend
- **TypeScript**: Implementación del servidor y lógica de negocio

### Base de Datos
- **PostgreSQL**: Sistema de gestión de datos relacional

---

## Arquitectura

Se sigue el patrón **MVC (Model-View-Controller)**:

- **Model**: Capas de acceso a datos y lógica de negocio (PostgreSQL)
- **View**: Interfaz de usuario (HTML + CSS)
- **Controller**: Lógica de coordinación entre vistas y modelos (TypeScript)

Esta arquitectura separa responsabilidades y facilita el mantenimiento y escalabilidad de la aplicación.

---

## Base de Datos

### Esquema Relacional

#### Tablas Principales

**User**
- Gestiona información de usuarios (miembros, empleados)
- Campos: id, nombre, apellidos, email, DNI, dirección completa, teléfono, rol, estado, fecha nacimiento

**Item**
- Catálogo de libros y materiales de la biblioteca
- Campos: id, título, editorial, ISBN, edición, fecha publicación, sinopsis, ubicación, estado, calidad, idioma, extensión, país, tipo

**Loan**
- Registro de préstamos realizados
- Campos: id, id_user, id_item, fecha inicio, fecha fin, fecha devolución, observaciones, gestionado por

**Reservation**
- Reservas de ítems por usuarios
- Campos: id, id_item, id_user, observaciones, estado

**Request**
- Solicitudes de usuarios (adquisición, mantenimiento, etc.)
- Campos: id, id_user, descripción, estado, creado por, actualizado por, gestionado por

**Sanction**
- Sanciones aplicadas a usuarios (multas, restricciones)
- Campos: id, id_user, id_loan, días penalización, tipo, estado, observaciones

**Audit_log**
- Registro de auditoría de cambios en la BD
- Campos: id, id_user, tabla afectada, id registro, operación, datos anteriores, datos nuevos, timestamp

**Writer**
- Autores de los ítems
- Campos: id, nombre, apellidos, país, fecha nacimiento

**Genre**
- Géneros literarios disponibles
- Campos: id, nombre

#### Tablas Relacionales (Many-to-Many)

**Item_writer_rel**
- Relación entre ítems y autores

**Item_genre_rel**
- Relación entre ítems y géneros

### Relaciones Clave

- Un **Usuario** puede tener múltiples **Préstamos**, **Reservas**, **Solicitudes** y **Sanciones**
- Un **Ítem** puede tener múltiples **Préstamos**, **Reservas**, varios **Autores** y **Géneros**
- Los **Préstamos** registran quién realiza la gestión (employee/admin)
- El **Audit Log** rastrea cambios en cualquier tabla

---

### Configuración de Base de Datos

1. Inicializar base de datos PostgreSQL
2. Ejecutar seed: `psql -U <usuario> -d <base_datos> -f docs/seedsql.sql`
3. Verificar esquema con: `psql -U <usuario> -d <base_datos> -c "\dt"`

## Descripción General

App Activa Biblioteca es un sistema de gestión integral para bibliotecas que permite:

- Administrar catálogo de libros y materiales
- Gestionar préstamos y devoluciones
- Manejar reservas de ítems
- Registrar sanciones a usuarios
- Procesar solicitudes de usuarios
- Mantener auditoría de cambios
- Gestionar información de usuarios y autores

---

## Metodología de trabajo (normas para Claude Code)

Estas reglas son **vinculantes** para cualquier asistente de IA (Claude Code) que programe en
este proyecto. Tienen prioridad sobre cualquier preferencia por defecto del asistente.

### 1. Test-Driven Development (TDD) — obligatorio

Todo el desarrollo de lógica se hace siguiendo el ciclo **Red → Green → Refactor**:

1. **Red** — Escribe primero un test que describa el comportamiento deseado. Ejecútalo y
   confirma que **falla** por el motivo esperado (la funcionalidad aún no existe). Un test que
   pasa a la primera no ha probado nada.
2. **Green** — Escribe el **mínimo** código de producción necesario para que el test pase. Nada de
   funcionalidad extra «por si acaso».
3. **Refactor** — Con los tests en verde, mejora el diseño del código sin cambiar su
   comportamiento. Los tests deben seguir pasando tras cada refactor.

Normas de aplicación:

- **Ningún código de producción se escribe sin un test que lo exija** primero. Si te pido una
  funcionalidad, el primer entregable es el test (o la lista de tests), no la implementación.
- Cada **caso de uso** de [`casos-de-uso.md`](./casos-de-uso.md) se traduce en tests: el camino
  feliz **y cada excepción** listada (p. ej. «el socio tiene multas activas», «el libro ya está
  registrado», «el usuario no tiene permisos»).
- Ejecuta la suite completa antes de dar una tarea por terminada. No se marca como hecha una
  tarea con tests en rojo; si algo falla, se dice claramente con la salida del test.
- Los tests se organizan por capa del MVC: lógica de negocio y modelos (unitarios), controladores
  (integración), y acceso a datos contra PostgreSQL donde aplique.

### 2. Regla de oro: no se tocan los tests ya verificados

> **Nunca modifiques, reescribas, deshabilites, borres ni relajes (`skip`, `only`, cambiar una
> aserción, ampliar un margen, comentar) un test que ya haya sido probado y esté en verde, sin
> consultármelo antes y obtener mi permiso explícito.**

Un test que ya ha pasado es un **contrato de comportamiento acordado**. Cambiarlo para que el
código nuevo encaje es invertir el TDD: en lugar de que el test valide el código, se amaña el
test para tapar un fallo. Eso destruye la red de seguridad.

Qué **sí** puedes hacer sin preguntar:
- Añadir **nuevos** tests.
- Modificar un test que **tú mismo acabas de escribir en esta misma tarea** y que aún no ha
  pasado una vez en verde (todavía está en la fase Red).

Qué requiere **mi permiso explícito** antes de tocar nada:
- Cualquier cambio a un test que ya estaba en verde antes de empezar la tarea.
- Si para implementar algo «necesitas» cambiar un test existente: **detente y pregunta**. No lo
  cambies y me avises después.

### 3. Cómo debes consultarme un cambio de test

Cuando creas que un test ya verificado debe cambiar, **párate** y preséntame una consulta
**didáctica**, pensada para que yo entienda y decida. Incluye, de forma clara:

1. **Qué test** es y qué comportamiento protege actualmente (en lenguaje llano).
2. **Por qué** crees que debe cambiar. Distingue explícitamente entre las dos causas posibles:
   - El **requisito ha cambiado** (el comportamiento esperado ahora es otro) → el test está
     desactualizado y procede actualizarlo.
   - El **código nuevo no cumple** el contrato (hay un conflicto real) → quizá el fallo está en el
     código, no en el test.
3. **El cambio concreto** que propones: aserción actual → aserción nueva, con el diff.
4. **Consecuencias**: qué deja de estar garantizado si cambiamos el test, y qué casos de uso o
   excepciones se ven afectados.
5. **Alternativas** que has considerado sin tocar el test (p. ej. ajustar el código de producción).

Explícamelo como si me enseñaras: el objetivo es que yo comprenda el *motivo* y las implicaciones,
no solo que apruebe. **Espera mi confirmación explícita** antes de modificar el test. Un «sí»
genérico no vale si no he visto el porqué.
