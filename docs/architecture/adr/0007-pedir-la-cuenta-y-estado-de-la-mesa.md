# 0007. Pedir la cuenta y estado de la mesa

Estado: Aceptado
Fecha: 2026-11-05

## Contexto
La especificación tiene dos cosas que pueden reflejar que se pidió la cuenta: el estado de la sesión (`sesiones_mesa.estado`: `ABIERTA`, `CUENTA_PEDIDA`, `CERRADA`) y el llamado de tipo `CUENTA` (`llamados`, `PENDIENTE` / `ATENDIDO`). No decía cuál manda, qué pasa al atender el llamado ni qué pasa si la mesa pide algo más después de pedir la cuenta. Los estados de la mesa en la especificación son "libre → ocupada → cuenta pedida → libre (al registrar el pago)". Los colores del plano son: libre (sin relleno), ocupada (azul), pidió la cuenta (amarillo) y llamó al mozo (rojo).

El usuario confirmó la regla el 2026-11-05 (especificación, "Reglas confirmadas", regla 6):
- El estado de la sesión es la fuente de verdad; el llamado `CUENTA` es solo el aviso para la caja.
- Pedir la cuenta es una transacción que pone la sesión en `CUENTA_PEDIDA` y crea el llamado.
- Atender el llamado no cambia el estado de la sesión; solo el pago la cierra.
- Color: rojo (llamado al mozo pendiente) > amarillo (`CUENTA_PEDIDA`) > ocupada > libre.
- Un pedido nuevo después de pedir la cuenta se acepta, la sesión vuelve a `ABIERTA` y la caja recibe un aviso de que la cuenta cambió.

Ampliación confirmada el 2026-11-06 (especificación, regla 6): si la sesión vuelve a `ABIERTA` por un pedido nuevo, el llamado de cuenta pendiente pasa automáticamente a `CANCELADO`. Si el cliente vuelve a pedir la cuenta, se crea un llamado nuevo.

Ya existe el índice único parcial `llamados_uno_pendiente_por_tipo` (ADR 0005, P16): un solo llamado pendiente por sesión y tipo; si se repite, se devuelve el existente sin volver a publicar `llamado.nuevo`.

## Decisión

### Transiciones de la sesión

| Desde | Hacia | Cuándo | Quién |
| --- | --- | --- | --- |
| (sin sesión) | `ABIERTA` | Primer escaneo del QR o el mozo abre la mesa | Cliente / mozo |
| `ABIERTA` | `CUENTA_PEDIDA` | Pedir la cuenta | Cliente desde el QR |
| `CUENTA_PEDIDA` | `ABIERTA` | Se crea un pedido nuevo en la sesión (cualquier origen) | Automático, dentro de la transacción del pedido |
| `ABIERTA` o `CUENTA_PEDIDA` | `CERRADA` | Pago final (`PAGO`), `ABANDONO`, `MANUAL` o `AUTOMATICO` (regla 5) | Caja / encargado / sistema |

Atender el llamado `CUENTA` **no** es una transición: la sesión sigue en `CUENTA_PEDIDA` hasta que se cierra o llega un pedido nuevo.

### Estados del llamado

`PENDIENTE` → `ATENDIDO` (alguien del personal lo atiende) o `PENDIENTE` → `CANCELADO` (solo automático, solo tipo `CUENTA`, cuando la sesión vuelve a `ABIERTA` por un pedido nuevo). `ATENDIDO` y `CANCELADO` son finales.

### En la base
- El enum `EstadoSesionMesa` ya tiene `CUENTA_PEDIDA`, y `llamados_uno_pendiente_por_tipo` ya evita llamados `CUENTA` duplicados mientras hay uno pendiente.
- **Nuevo valor `CANCELADO` en `EstadoLlamado`** y columna `cancelado_en timestamptz` nullable, con CHECK `llamados_cancelacion_completa`: `(estado = 'CANCELADO') = (cancelado_en IS NOT NULL)`. No lleva `cancelado_por`, porque lo hace el sistema.
  - `llamados_atencion_completa` (`(estado = 'ATENDIDO') = (atendido_por_id IS NOT NULL AND atendido_en IS NOT NULL)`) no cambia: en un llamado cancelado los dos lados dan falso.
  - `llamados_uno_pendiente_por_tipo` (`WHERE estado = 'PENDIENTE'`) no cambia: el cancelado sale del índice, así que el próximo "pedir la cuenta" puede crear un llamado nuevo.
  - *Propuesta:* CHECK `llamados_cancelado_solo_cuenta`: `estado <> 'CANCELADO' OR tipo = 'CUENTA'`.
- El color de la mesa **no se guarda**: se calcula.

### En el servicio
- **Pedir la cuenta** (módulo salon, que es dueño de sesiones y llamados). Es una sola transacción:
  1. `SELECT … FROM sesiones_mesa WHERE local_id = $local AND id = $sesion FOR UPDATE` (la sesión sale del token de la sesión de mesa, no del body).
  2. Si está `CERRADA` → 409. Si el local está en modo consulta → se rechaza, igual que cualquier acción por QR ("Contingencia sin conexión").
  3. Si está `ABIERTA` → pasa a `CUENTA_PEDIDA`.
  4. Crea el llamado `CUENTA`, o devuelve el pendiente que ya exista (ADR 0005, P16).
  5. Después del commit: `mesa.estado` (salón, caja) si cambió el estado, y `llamado.nuevo` (salón, caja) solo si se creó un llamado nuevo.
  - Si la sesión ya estaba en `CUENTA_PEDIDA` y el llamado sigue pendiente, la operación no cambia nada y no publica nada.
- **Atender el llamado** (salon): pasa el llamado a `ATENDIDO` con `atendido_por` y `atendido_en`; no toca la sesión. Publica `llamado.atendido` (mesa, salón).
- **Pedido nuevo con la cuenta pedida** (pedidos → salon). En la misma transacción que crea el pedido, pedidos llama al servicio público de salon (no a la tabla), que bloquea la sesión con `FOR UPDATE` y, si está en `CUENTA_PEDIDA`, la pasa a `ABIERTA` y pasa a `CANCELADO` el llamado `CUENTA` pendiente, si hay (con `cancelado_en`). Orden de bloqueo fijo: primero la sesión, después lo demás del pedido. Después del commit: `pedido.nuevo` (salón, barra) y `mesa.estado` (salón, caja). No hace falta un evento `llamado.cancelado`: la caja, al recibir `mesa.estado` con `cuentaCambiada`, saca el aviso de cuenta de esa mesa (*propuesta*, junto con la del campo).
  - Si el cliente vuelve a pedir la cuenta, no hay pendiente: se crea un llamado nuevo y se publica `llamado.nuevo`.
  - El cambio ocurre cuando el pedido **se crea**, también si entra `A_CONFIRMAR`.
- **Aviso a la caja de que la cuenta cambió** (*propuesta*). No se crea un evento nuevo: `mesa.estado` ya llega a la caja. Se le agrega al payload un campo `cuentaCambiada: true` cuando la sesión vuelve de `CUENTA_PEDIDA` a `ABIERTA` por un pedido nuevo, y la caja muestra el aviso con eso. Como el canal solo avisa, la caja que se reconecta no recibe el aviso: ve la mesa en azul y el total nuevo, que es el estado verdadero. El payload exacto se fija en `docs/api/contratos.md` en la tarea de salón.
- **Color de la mesa** (salon, servicio del plano). Se toma la sesión no cerrada de la mesa (hay como máximo una, por `sesiones_mesa_una_abierta_por_mesa`) y se aplica la primera que corresponda:
  1. **Rojo:** la sesión tiene un llamado `MOZO` en `PENDIENTE`.
  2. **Amarillo:** la sesión está en `CUENTA_PEDIDA`.
  3. **Azul (ocupada):** la sesión está en `ABIERTA`.
  4. **Libre:** no hay sesión no cerrada.

  El amarillo sale del estado de la sesión, no del llamado `CUENTA`: si la caja atiende el llamado, la mesa sigue amarilla. Los llamados de una sesión cerrada no pintan la mesa. El total consumido que muestra el plano se arma con los servicios de pedidos y caja (ADR 0003), no con joins.

## Alternativas descartadas
- **El llamado `CUENTA` pendiente como fuente de verdad** (sin `CUENTA_PEDIDA`): atender el aviso "despediría" la cuenta, y quedarían dos fuentes de verdad.
- **Que atender el llamado vuelva la sesión a `ABIERTA`:** contradice la regla; solo el pago cierra.
- **Rechazar pedidos con la cuenta pedida:** la regla dice que se aceptan.
- **Guardar el color en la base:** sería un dato derivado que se puede desincronizar.
- **Evento nuevo `cuenta.cambiada`:** también funciona. Se prefiere el campo en `mesa.estado` para no sumar eventos fuera de la especificación; si la caja necesita un sonido o un tratamiento aparte, se reconsidera en la tarea de salón.

## Consecuencias
- El plano de caja y el de salón usan la misma función de color, en salon, con tests unitarios de la prioridad.
- La transacción del pedido toca dos módulos (pedidos y salon). Se hace con el `TransactionManager` compartido y el servicio público de salon, sin acceder a tablas ajenas.
- Tests obligatorios:
  - Pedir la cuenta pone `CUENTA_PEDIDA` y crea el llamado en la misma transacción: si falla una parte, no queda ninguna.
  - Pedir la cuenta dos veces: un solo llamado pendiente, sin segundo `llamado.nuevo`.
  - Atender el llamado `CUENTA` deja la sesión en `CUENTA_PEDIDA`.
  - Pedido nuevo en `CUENTA_PEDIDA`: se acepta, la sesión vuelve a `ABIERTA`, el llamado `CUENTA` pendiente queda `CANCELADO` con `cancelado_en`, y `mesa.estado` lleva `cuentaCambiada`.
  - Volver a pedir la cuenta después de eso: se crea un llamado nuevo (el cancelado no bloquea el índice parcial) y se publica `llamado.nuevo`.
  - CHECKs: `CANCELADO` sin `cancelado_en`, o con `atendido_por` → rechazado; `CANCELADO` en un llamado `MOZO` → rechazado (si se acepta la propuesta).
  - Prioridad de colores: llamado al mozo pendiente + `CUENTA_PEDIDA` → rojo; atendido → amarillo; etc.
  - Pedir la cuenta en modo consulta o con la sesión cerrada → rechazado.
  - Pedir la cuenta y crear un pedido a la vez sobre la misma sesión (contra PostgreSQL real): el resultado final es coherente con el orden en que se tomaron los bloqueos.

- Cambios concretos al schema: especificación, "Reglas confirmadas" → "Cambios pendientes en el modelo", filas 11 y 14. Entran en la tarea de salón.

---
Ampliado el 2026-11-06 con la regla confirmada "llamado de cuenta cancelado al volver a `ABIERTA`" (especificación, regla 6).
