# 0004. Inmutabilidad de pagos y movimientos de puntos

Estado: Aceptado. Reemplazado en parte por 0006 (anulaciones parciales: un pago puede tener varios contramovimientos)
Fecha: 2026-11-05

## Contexto
La especificación y CLAUDE.md dicen que `pagos` y `movimientos_puntos` no se editan ni se borran: se anulan con un contramovimiento. Los puntos se acreditan en la misma transacción que registra el pago. El arqueo del turno, la carga posterior de contingencia y (más adelante) la facturación dependen de que ese historial sea confiable.

## Decisión
- `pagos` y `movimientos_puntos` son **solo de inserción** (append-only).
- Una anulación es una fila nueva con monto/cantidad de signo opuesto que referencia a la original (`anula_a_id`), con motivo, usuario y fecha. Una fila solo puede anularse una vez (índice único sobre `anula_a_id`). _Reemplazado por ADR 0006: se permiten anulaciones parciales._
- Estas tablas no tienen `actualizado_en`, ni borrado lógico, ni columna de estado editable; el estado "anulado" se deduce de la existencia del contramovimiento.
- Se refuerza en la base: la migración agrega triggers que rechazan `UPDATE`, `DELETE` y `TRUNCATE` sobre esas tablas (los tests de integración deshabilitan solo el de `TRUNCATE` un momento para vaciar la base). El rol de la aplicación no necesita esos permisos.
- La acreditación de puntos de un pago y su anulación ocurren en la misma transacción que el pago o el contrapago.
- `movimientos_puntos` se crea en la etapa 2; la regla y el trigger aplican desde que exista.

## Alternativas descartadas
- Columna `anulado` + `UPDATE` — pierde trazabilidad y rompe el arqueo de turnos ya cerrados.
- Borrado lógico (`eliminado_en`) — mismo problema, y es fácil olvidar el filtro.
- Confiar solo en la disciplina del código — un script o una migración mal hecha podría editar historial.

## Consecuencias
- Totales y saldos se calculan sumando filas (con signo); conviene indexar por `sesion_id`, `turno_id` y `afiliado_id`.
- Corregir un error de carga siempre deja rastro (pago + contrapago).
- Los tests de integración deben verificar que `UPDATE`, `DELETE` y `TRUNCATE` fallan contra PostgreSQL real.
- `pagos` se crea en la migración inicial de la etapa 1 (`20261106021213_modelo_etapa_1`), ya con el trigger; la lógica de cobro llega en la tarea de caja.
