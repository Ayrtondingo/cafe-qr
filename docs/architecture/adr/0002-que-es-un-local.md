# 0002. Qué es un local

Estado: Aceptado
Fecha: 2026-11-05

## Contexto
La especificación pide que todas las tablas lleven `local_id` (multi-tenant, como Kiosco SaaS), aunque por ahora los puntos valen en un solo café. Hay que definir cuál es la raíz del tenant y de dónde sale el `local_id` en cada request.

## Decisión
- La tabla **`locales`** es la raíz del tenant. Cada fila es un café físico con su configuración (pesos por punto, tiempo de corte de la barra, aprobación en hora pico, etc.).
- Todas las tablas de dominio llevan `local_id` NOT NULL con FK a `locales`. No hay datos de dominio sin local.
- El `local_id` se obtiene **solo** del usuario autenticado (personal) o de la sesión de mesa / código QR (cliente). Nunca del cuerpo, la query ni los parámetros del request.
- Los repositorios reciben el `local_id` del contexto y filtran siempre por él; un recurso de otro local responde como no encontrado.
- Más adelante, una tabla **`marcas`** podrá agrupar varios locales (por ejemplo, para que los puntos valgan en todas las sucursales). No se crea ahora.

## Alternativas descartadas
- Crear ya `marcas` u `organizaciones` como raíz — no lo pide ninguna etapa actual; se agrega sin romper nada cuando haga falta.
- Una base o un esquema por local — complica migraciones y el módulo compartido de puntos.
- Recibir `local_id` en el request y validarlo — abre la puerta a fugas entre locales.

## Consecuencias
- Aislamiento simple y testeable: cada endpoint tiene un test de "otro local no ve esto".
- Cuando aparezca `marcas`, habrá que decidir en un ADR nuevo si `afiliados` y lotes pasan a colgar de la marca en vez del local.
