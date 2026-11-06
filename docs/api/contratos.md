# Contrato entre API y web

Fuente de verdad de lo que la API expone y la web consume. Se actualiza **antes** de cambiar código en cualquiera de los dos repos.

## Endpoints

| Método | Ruta | Rol | Request | Response | Etapa |
| --- | --- | --- | --- | --- | --- |
| | | | | | |

## Eventos en vivo

| Evento | Sala | Payload | Lo reciben |
| --- | --- | --- | --- |
| | | | |

## Convenciones

- Montos en centavos (enteros) en todos los payloads.
- Fechas en ISO 8601 con zona horaria.
- Errores con forma `{ codigo, mensaje }`; `codigo` estable para que la web lo traduzca.
