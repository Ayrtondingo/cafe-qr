# 0001. ORM y convenciones de base

Estado: Aceptado
Fecha: 2026-11-05

## Contexto
La API es NestJS + TypeScript sobre PostgreSQL y CLAUDE.md pide usar el mismo ORM y las mismas convenciones que Kiosco SaaS. La especificación exige `local_id` en todas las tablas, montos sin float, transacciones con `SELECT … FOR UPDATE` (lotes de puntos, transferencias) y tests de concurrencia contra PostgreSQL real.

## Decisión
- ORM: **Prisma 7** con el driver adapter **`@prisma/adapter-pg`** (cliente `pg` nativo), igual que Kiosco SaaS. Las consultas con bloqueo (`FOR UPDATE`, orden fijo por `id`) se escriben con `$queryRaw` dentro de `$transaction` interactiva.
- Migraciones: `prisma migrate`, versionadas en `cafe-qr-api/prisma/migrations`. Lo que Prisma no expresa (`CHECK` y triggers) se agrega a mano en el SQL de la migración.
- Nombres en español y `snake_case` en la base (tablas en plural: `mesas`, `sesiones_mesa`, `pedido_items`); en el modelo Prisma se mapean con `@@map` / `@map`.
- Claves primarias: UUID v7 (`@default(uuid(7)) @db.Uuid`), generados por Prisma al insertar, como en Kiosco SaaS. Los valores de los enums van en MAYÚSCULAS (`ABIERTA`, `CUENTA_PEDIDA`).
- Índices únicos parciales: con la función preview `partialIndexes` de Prisma 7 (`where: raw("...")`), así quedan en el schema y `npm run db:check-drift` no los pierde.
- Toda tabla de dominio lleva `local_id` (FK a `locales`, ver ADR 0002), con índice que empieza por `local_id`.
- Montos: `Decimal(14,2)` en la base (pesos con centavos exactos); el dominio y la API trabajan con **centavos enteros** (`number` entero). La conversión vive solo en la capa de infraestructura (repositorios/mapeadores).
- Fechas: `timestamptz`, siempre en UTC en la base; ISO 8601 con zona en la API.
- Columnas de auditoría `creado_en` / `actualizado_en` en las tablas editables; las tablas inmutables (ADR 0004) solo `creado_en`.
- Tests de integración contra PostgreSQL en Docker, con base propia por corrida.

## Alternativas descartadas
- TypeORM — se aleja de Kiosco SaaS y complica compartir el módulo de puntos.
- Drizzle / Knex — mismo motivo; además, sin generador de migraciones equivalente al del otro producto.
- Guardar montos como centavos `integer` en la base — Kiosco SaaS usa decimal; mantener la misma representación facilita reutilizar ventas, medios de pago y turnos.
- Montos `float` / `numeric` sin escala — prohibido por la especificación.

## Consecuencias
- Se puede reutilizar esquema y código de Kiosco SaaS casi sin traducción.
- Los bloqueos con `FOR UPDATE` quedan en SQL crudo: hay que testearlos siempre contra PostgreSQL real.
- Cada repositorio debe convertir Decimal ↔ centavos; un test por mapeador evita errores de redondeo.
- Los `CHECK` y triggers escritos a mano no los compara Prisma: se cubren con tests de integración contra PostgreSQL real.
