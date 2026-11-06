# 0006. Contramovimientos de pagos

Estado: Aceptado
Fecha: 2026-11-05

## Contexto
El ADR 0004 decidió que `pagos` es solo de inserción y que un pago se anula con un contramovimiento: una fila nueva con monto de signo opuesto que referencia a la original con `anula_a_id`. Ese ADR sigue vigente. Este lo **complementa** con las reglas que el usuario confirmó el 2026-11-05 (especificación, "Reglas confirmadas", regla 2). La regla habla de `anula_pago_id`; en el modelo esa columna es `anula_a_id`.

Reglas confirmadas:
- `anula_a_id` es obligatorio y apunta a un pago del mismo local y de la misma sesión.
- Se permiten anulaciones parciales. La suma de las anulaciones de un pago no supera su monto; la propina se controla por separado.
- Un contramovimiento no se puede anular; si fue un error, se registra un pago nuevo.
- Mismo medio de pago que el original.
- Se imputa al turno abierto actual, no al del pago original.
- Lo registra un cajero o el encargado, siempre con motivo.
- En la base: signo del monto, FK y "el pago anulado no es un contramovimiento". En el servicio: el tope de la suma, con `SELECT … FOR UPDATE` sobre el pago original.

Reglas confirmadas el 2026-11-06:
- **Anular solo la propina:** contramovimiento con monto 0 y propina negativa; al menos uno de los dos distinto de cero. Se refleja en el reporte de propinas del mes de la anulación.
- **Anular un pago de una sesión cerrada:** la sesión no se reabre; el contramovimiento queda ligado a la sesión original y se imputa al turno actual; si el error fue de medio de pago, se registra un pago de corrección en la misma sesión cerrada; si es una devolución al cliente, solo el contramovimiento. Lo hace el encargado; si el turno del pago original ya cerró, solo el dueño. Si estaba facturado, va con nota de crédito y el pago de corrección lleva su propia factura. En la etapa 2, si el pago sumó puntos, se descuentan en proporción.
- **Motivos codificados:** el contramovimiento guarda `codigo_motivo` (`ERROR_DE_COBRO`, `PRODUCTO_MAL_HECHO`, `CLIENTE_ARREPENTIDO`, `OTRO`) y un `comentario` opcional, obligatorio con `OTRO`.

Qué hay hoy en la migración `20261106021213_modelo_etapa_1`:
- CHECK `pagos_normal_o_contramovimiento`: pago normal (`anula_a_id` nulo, `monto > 0`, `propina >= 0`, sin motivo) o contramovimiento (`anula_a_id` no nulo, `monto < 0`, `propina <= 0`, con motivo).
- CHECK `pagos_no_se_anula_a_si_mismo`.
- FK `(local_id, anula_a_id) → pagos(local_id, id)`.
- Único `pagos_local_id_anula_a_id_key` sobre `(local_id, anula_a_id)`: un pago se anula una sola vez.

**El único sobre `anula_a_id` contradice las anulaciones parciales.** Esto reemplaza un solo punto del ADR 0004 ("Una fila solo puede anularse una vez (índice único sobre `anula_a_id`)"). Todo lo demás del ADR 0004 sigue vigente: inmutabilidad, triggers, contramovimiento de signo opuesto y puntos en la misma transacción.

## Decisión

### En la base (migración)

1. **Signo y motivo.** El CHECK `pagos_normal_o_contramovimiento` queda así:
   - Pago normal: `anula_a_id IS NULL AND monto > 0 AND propina >= 0 AND codigo_motivo IS NULL AND comentario IS NULL`.
   - Contramovimiento: `anula_a_id IS NOT NULL AND monto <= 0 AND propina <= 0 AND (monto < 0 OR propina < 0) AND codigo_motivo IS NOT NULL`.

   `motivo_anulacion` se reemplaza por `codigo_motivo` (enum `CodigoMotivoAnulacion`) + `comentario`, con CHECK `pagos_comentario_si_otro`: `codigo_motivo IS DISTINCT FROM 'OTRO' OR (comentario IS NOT NULL AND btrim(comentario) <> '')`.
2. **Se permiten varias anulaciones por pago.** Se quita el único `pagos_local_id_anula_a_id_key` y se reemplaza por un índice no único `(local_id, anula_a_id)` para sumar rápido las anulaciones de un pago.
3. **Columnas auxiliares** (columnas comunes con CHECK, no columnas generadas, para que Prisma las modele sin trucos):
   - `es_contramovimiento boolean NOT NULL`, con CHECK `pagos_es_contramovimiento_coherente`: `es_contramovimiento = (anula_a_id IS NOT NULL)`.
   - `anulado_es_contramovimiento boolean NULL`, con CHECK `pagos_anulado_no_contramovimiento`: `(anula_a_id IS NULL AND anulado_es_contramovimiento IS NULL) OR (anula_a_id IS NOT NULL AND anulado_es_contramovimiento = false)`.
4. **Único de destino:** `pagos (local_id, sesion_id, id, medio, es_contramovimiento)`.
5. **FK compuesta** que reemplaza a la FK actual de `anula_a_id`:
   `(local_id, sesion_id, anula_a_id, medio, anulado_es_contramovimiento) → pagos (local_id, sesion_id, id, medio, es_contramovimiento)`, con `ON DELETE RESTRICT ON UPDATE RESTRICT` y `MATCH SIMPLE` (el que viene por defecto). Con una sola FK la base garantiza:
   - **Mismo local y misma sesión:** `local_id` y `sesion_id` del contramovimiento tienen que coincidir con los del pago anulado.
   - **El pago anulado no es un contramovimiento:** el contramovimiento solo puede tener `anulado_es_contramovimiento = false` (CHECK del punto 3), así que solo puede apuntar a una fila con `es_contramovimiento = false`. Sin trigger.
   - **Mismo medio de pago** (*propuesta*, ver abajo).
   - En un pago normal `anula_a_id` y `anulado_es_contramovimiento` son nulos, y con `MATCH SIMPLE` la FK no se controla.
6. Se mantiene `pagos_no_se_anula_a_si_mismo` (sobra con la FK, porque una fila no puede ser a la vez contramovimiento y no contramovimiento, pero es barato y deja clara la intención).
7. **Permiso para el bloqueo.** PostgreSQL exige privilegio `UPDATE` sobre al menos una columna para hacer `SELECT … FOR UPDATE`. Si el rol de la aplicación no es dueño de la tabla y no tiene `UPDATE` en `pagos` (el ADR 0004 dice que no lo necesita), se le da `UPDATE` sobre una sola columna (por ejemplo `GRANT UPDATE (creado_en) ON pagos TO <rol_app>`). El trigger `pagos_inmutables` igual rechaza cualquier `UPDATE` real, así que la inmutabilidad no cambia. Esto matiza la frase "el rol de la aplicación no necesita esos permisos" del ADR 0004.

**Propuesta: el mismo medio va en la base.** El usuario no dijo dónde va esta regla. Se propone meter `medio` en la FK del punto 5 porque no agrega columnas ni triggers: solo suma una columna a una FK que ya hace falta. El servicio igual copia el medio del original (el cliente de la API no lo elige), así que la FK solo frena errores de programación o scripts. Si el usuario prefiere dejarla solo en el servicio, se saca `medio` de la FK y del único de destino, y nada más cambia.

### En el servicio (módulo caja, caso de uso "anular pago")

1. El controller valida que el usuario sea cajero, encargado o dueño (el permiso final depende del pago, ver 2.2) y que llegue `codigo_motivo`, con comentario si es `OTRO`; el `local_id` y el usuario salen de la autenticación. El request lleva el `id` del pago, el monto a anular y la propina a anular (los dos en centavos, ≥ 0 y al menos uno > 0); **no** lleva medio, sesión ni turno.
2. En una sola transacción (`TransactionManager`, `$queryRaw`):
   1. `SELECT … FROM pagos WHERE local_id = $local AND id = $pago FOR UPDATE`. Si no existe en el local → 404. Si es un contramovimiento → 409 (no se anula un contramovimiento).
   2. **Permisos según el pago** (la sesión y el turno se leen por el servicio público de salon y de caja, sin bloquearlos para cambiarlos):
      - Sesión del pago abierta (`ABIERTA` o `CUENTA_PEDIDA`): cajero o encargado.
      - Sesión cerrada y turno del pago original abierto: encargado.
      - Sesión cerrada y turno del pago original cerrado: solo el dueño.
      - Si no corresponde → 403.
      - La sesión no cambia de estado en ningún caso.
   3. **Después** de tener el bloqueo, en una sentencia aparte: `SELECT COALESCE(SUM(-monto), 0), COALESCE(SUM(-propina), 0) FROM pagos WHERE local_id = $local AND anula_a_id = $pago`. Tiene que ser una sentencia nueva para que, en `READ COMMITTED`, vea las anulaciones que otra transacción confirmó mientras esta esperaba el bloqueo.
   4. Controla `ya_anulado + monto_a_anular ≤ monto_original` y, por separado, `propina_ya_anulada + propina_a_anular ≤ propina_original`. Si no se cumple → 409. Con `monto_a_anular = 0` es una anulación solo de propina.
   5. Busca el turno abierto actual del local; si no hay → 409. El contramovimiento se imputa a ese turno, no al del pago original, aunque la sesión esté cerrada.
   6. Inserta el contramovimiento con `sesion_id` (la sesión original) y `medio` copiados del original, `monto` ≤ 0 y `propina` ≤ 0, `es_contramovimiento = true`, `anulado_es_contramovimiento = false`, `registrado_por_id` = el usuario, `codigo_motivo` y `comentario`.
   7. *Dependencia, etapa 2 (módulo puntos):* en la misma transacción se descuentan los puntos del pago, en proporción a lo anulado (ADR 0004). No se diseña ahora.
   8. *Dependencia, ARCA (pospuesto, ADR 0005 P6):* si el pago estaba facturado, la anulación lleva nota de crédito. No se diseña ahora.
3. Después del commit, tiempo-real publica el aviso (evento a definir en la tarea de caja; ver "Pendiente").

**Pago de corrección (sesión cerrada, error de medio de pago).** Es un pago normal (`monto > 0`) en la misma sesión cerrada, con el medio correcto; la sesión no se reabre. El modelo actual lo permite (la FK a la sesión no mira el estado). Va en un caso de uso propio de caja, porque el cobro común exige sesión abierta, con los mismos permisos que la anulación de esa sesión (encargado; solo el dueño si el turno del pago original cerró). *Propuesta:* contramovimiento y pago de corrección en una sola transacción, para que el turno nunca muestre la mitad de la corrección. *Dependencia ARCA:* el pago de corrección lleva su propia factura. Si es una devolución al cliente, solo se registra el contramovimiento.

**Reporte de propinas.** Las propinas se suman con signo por la fecha (`creado_en`) de cada fila: una anulación de propina resta en el mes en que se anula, no en el del pago original.

**Concurrencia.** Dos anulaciones parciales simultáneas del mismo pago (por ejemplo, dos de $600 sobre un pago de $1.000): las dos piden `FOR UPDATE` sobre el original; la segunda espera. Cuando la primera confirma, la segunda toma el bloqueo, vuelve a sumar (paso 2.3), ve $600 ya anulados y rechaza con 409. Si la primera hace rollback, la segunda sigue y se registra. Si la tarea de caja también bloquea la sesión o el turno en la misma transacción, el orden es fijo para no generar deadlocks: sesión → turno → pago.

**Códigos de error** (propuesta, se fijan en `docs/api/contratos.md` en la tarea de caja): `PAGO_NO_ENCONTRADO` (404), `PAGO_ES_CONTRAMOVIMIENTO` (409), `ANULACION_SUPERA_MONTO` (409), `ANULACION_SUPERA_PROPINA` (409), `TURNO_NO_ABIERTO` (409), `PERMISO_INSUFICIENTE` (403).

## Alternativas descartadas
- **Mantener el único sobre `anula_a_id`** (una sola anulación, por el total): contradice las anulaciones parciales confirmadas.
- **Trigger que rechace anular un contramovimiento:** no hace falta; la FK compuesta con `anulado_es_contramovimiento = false` lo garantiza de forma declarativa y sin carreras.
- **Columnas auxiliares generadas** (`GENERATED ALWAYS AS … STORED`): Prisma no modela bien las columnas generadas y aparecerían como drift. Las columnas comunes con CHECK dan la misma garantía.
- **Tope de la suma en la base** (trigger con `SUM`, o un acumulado "anulado hasta ahora" en el pago original): el original es inmutable y no se puede actualizar, y un trigger con `SUM` necesita el mismo bloqueo para no tener carreras. El usuario lo ubicó en el servicio.
- **Bloquear la sesión en vez del pago:** también serializa, pero el usuario pidió `FOR UPDATE` sobre el pago original, y además bloquearía cobros no relacionados de la misma mesa.
- **Mismo medio solo en el servicio:** es válido (ver la propuesta); se prefiere la base porque no cuesta nada extra.
- **Imputar al turno del pago original:** reabriría el arqueo de turnos ya cerrados; el usuario decidió el turno abierto actual.

## Consecuencias
- El estado "anulado" de un pago se calcula: total, parcial o sin anular, según la suma de sus contramovimientos.
- El arqueo de un turno suma con signo los pagos y contramovimientos de ese turno. Una anulación de un pago de un turno ya cerrado aparece en el turno actual, por su medio.
- Los totales por sesión siguen siendo suma con signo. Una sesión cerrada puede quedar con saldo después de una devolución; no se reabre ni genera pérdida (las pérdidas solo salen de un cierre por `ABANDONO` o `MANUAL`).
- Tests obligatorios contra PostgreSQL real:
  - El CHECK acepta un contramovimiento solo de propina (`monto = 0`, `propina < 0`) y rechaza `monto = 0` con `propina = 0`, o cualquiera de los dos positivo.
  - Contramovimiento sin `codigo_motivo`, o con `OTRO` y sin comentario → rechazado.
  - Sesión cerrada: el cajero no puede anular; el encargado sí si el turno del original está abierto; con el turno original cerrado, solo el dueño. La sesión sigue `CERRADA` y el contramovimiento queda en el turno actual.
  - Pago de corrección en sesión cerrada: se registra junto con el contramovimiento, o no se registra ninguno de los dos.
  - La FK rechaza: un contramovimiento de otra sesión, de otro local, con otro medio, o que apunte a otro contramovimiento.
  - Los CHECKs rechazan `es_contramovimiento` o `anulado_es_contramovimiento` incoherentes.
  - Dos anulaciones parciales concurrentes cuya suma supera el monto: una sola se registra.
  - Varias anulaciones parciales que suman exactamente el monto: todas pasan; una más falla.
  - Propina: el tope se controla aparte del monto.
  - `SELECT … FOR UPDATE` funciona con el rol de la aplicación, y `UPDATE` sigue rechazado por el trigger.
- Cambios concretos al schema y a la migración: especificación, "Reglas confirmadas" → "Cambios pendientes en el modelo", filas 7, 8, 12, 13 y 16. Entran en la tarea de caja.

### Pendiente (para la tarea de caja)
- **Evento en vivo de la anulación:** ¿se reutiliza `pago.registrado` (Mesa, caja) con el monto negativo, o hace falta uno propio?
- **Sesión abierta con el turno del pago original cerrado** (por ejemplo, un pago de contingencia imputado a un turno anterior): la regla del dueño está escrita para sesiones cerradas. Si la tarea de caja bloquea el cierre de turno con mesas abiertas (ADR 0005, P13), el caso casi no aparece; si aparece, hay que preguntar.

---
Ampliado el 2026-11-06 con las reglas confirmadas "anular solo la propina", "anular un pago de una sesión cerrada" (especificación, regla 2) y "motivos codificados" (regla 8).
