# Contrato entre API y web

Fuente de verdad de lo que la API expone y la web consume. Se actualiza **antes** de cambiar código en cualquiera de los dos repos.

## Endpoints

| Método | Ruta | Rol | Request | Response | Etapa |
| --- | --- | --- | --- | --- | --- |
| GET | `/api/v1/health` | Público | — | 200 si la base responde, 503 si no. `{ data: { estado: "ok" \| "degradado", base: "ok" \| "caida", version: string, fecha: string (ISO 8601) } }` | 1 |

## Eventos en vivo

| Evento | Sala | Payload | Lo reciben |
| --- | --- | --- | --- |
| | | | |

## Convenciones

- Todas las rutas llevan el prefijo `/api/v1`.
- Respuestas exitosas: `{ data }` para un recurso; `{ data, meta }` para listas (`meta` lleva paginación o totales).
- Errores con forma `{ codigo, mensaje, detalles, ruta, fecha, requestId }`:
  - `codigo`: string estable (por ejemplo `PRODUCTO_AGOTADO`) para que la web lo traduzca.
  - `mensaje`: texto legible, no se muestra tal cual al cliente.
  - `detalles`: objeto o lista opcional (por ejemplo, errores de validación por campo); `null` si no hay.
  - `ruta`: ruta del request; `fecha`: ISO 8601; `requestId`: el mismo que queda en los logs.
- Montos en **centavos enteros** en todos los payloads (request y response), nunca decimales ni float. La base guarda `Decimal(14,2)` (ADR 0001).
- Fechas en ISO 8601 con zona horaria.
- El `local_id` nunca viaja en el request: sale del usuario autenticado o de la sesión de mesa (ADR 0002).
