# 0005. Decisiones de modelo de la etapa 1

Estado: Aceptado
Fecha: 2026-11-05

## Contexto
La especificación deja sin definir varias reglas de negocio de la etapa 1 (cuenta dividida, medios de pago, mostrador, opciones, agotado, anulaciones, llamados). El usuario las resolvió al aprobar el plan de la estructura inicial y el modelo de datos de la etapa 1. Este ADR las registra para que las tareas siguientes no las reabran.

## Decisión
- **Cuenta dividida por monto libre (P7).** Cada pago de una sesión es un monto libre; la sesión queda saldada cuando la suma de pagos vigentes cubre el total. No se asignan ítems a pagos. *Por qué:* es lo que hace un café en la práctica y no condiciona el modelo de ítems.
- **Dos medios = dos filas de `pagos` (P8).** Pagar parte en efectivo y parte con tarjeta genera dos pagos, cada uno con su medio. *Por qué:* mantiene un solo medio por fila, simplifica el arqueo por medio y la inmutabilidad (ADR 0004).
- **Pedido de mostrador = sesión sin mesa (P10).** `sesiones_mesa.mesa_id` es nullable; el mostrador abre una sesión sin mesa. *Por qué:* evita mesas virtuales que ensucian el plano y las estadísticas del salón.
- **Opciones de producto en grupos (P11).** Cada producto tiene grupos de opciones con mínimo y máximo de elecciones; cada opción puede tener recargo (centavos, ≥ 0). Las opciones elegidas y su recargo se copian al ítem al pedir. No hay combos. *Por qué:* cubre leche, tamaño y extras sin la complejidad de combos.
- **Agotado = se ve en gris y no se puede pedir (P12).** El producto agotado sigue visible en la carta, deshabilitado. La API rechaza pedidos con productos no disponibles. *Por qué:* el cliente entiende que existe y no lo busca; difiere de "oculta el producto" de la tabla de funciones de la especificación por decisión del usuario.
- **Estado por pedido; ítem VIGENTE/ANULADO (P14).** El estado de preparación (a confirmar → … → entregado) vive en `pedidos`; `pedido_items.estado` solo es `VIGENTE` o `ANULADO`. *Por qué:* hay una sola cola de barra y se prepara por pedido.
- **Llamado pendiente repetido devuelve el mismo (P16).** Índice único parcial en `llamados (sesion_id, tipo) WHERE estado = 'PENDIENTE'`; si el cliente vuelve a llamar con uno pendiente del mismo tipo, la API devuelve el existente sin crear otro ni volver a publicar `llamado.nuevo`. *Por qué:* evita alertas duplicadas y es idempotente sin clave extra. Entra en la migración inicial.

## Pendiente para tareas posteriores
- **P5 — roles N:M** (dueño, encargado, mozo, barra, cajero): tarea de autenticación.
- **P9 — `mozo_id` en `pagos`** para asignar la propina al mozo: tarea de caja.
- **P13 — fondo inicial y movimientos de caja** como Kiosco SaaS, un turno abierto por local, cierre bloqueado con mesas abiertas: tarea de caja.
- **P15 — `hubo_corte` en `turnos_caja`**: tarea de contingencia.
- **P6 — comprobantes ARCA** (`comprobantes`, `puntos_venta_arca`, condición frente al IVA): **pospuesta**, pendiente de confirmar con el contador (CAE diferido o CAEA). La migración inicial no incluye nada fiscal.

## Alternativas descartadas
- Cuenta dividida por ítems o partes iguales — más modelo (asignación ítem↔pago) sin pedido real del local.
- Un pago con varios medios en líneas hijas — duplica la lógica de anulación.
- Mesa virtual de mostrador — aparece en el plano y en reportes de mesas.
- Combos — no lo pide la etapa 1.
- Ocultar el agotado — descartado por el usuario.
- Estado de preparación por ítem — no hay cocina separada hasta la etapa 5.
- Permitir llamados duplicados y deduplicar en la web — la regla quedaría fuera de la base.

## Consecuencias
- El modelo de pagos queda simple y compatible con Mercado Pago (etapa 4): un QR por pago.
- Si se pide cuenta dividida por ítems más adelante, hará falta un ADR nuevo y una tabla de asignación.
- Separar cocina (etapa 5) obligará a revisar P14.
- La facturación (P6) deberá decidir cómo se emite un comprobante cuando una sesión tiene varios pagos.
