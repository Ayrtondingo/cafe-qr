# Sistema para café: pedidos por QR y club de puntos

Sistema web para un café: el cliente escanea el QR de su mesa, ve la carta, pide, sigue el pedido, llama al mozo y pide la cuenta; paga en la caja. El local maneja salón, barra y caja desde un panel en vivo. Encima hay un club de puntos con amigos, transferencias, regalos y campañas.

La especificación completa está en `docs/especificacion.md`. Leela antes de diseñar o cambiar cualquier parte del sistema y respetá sus reglas de negocio.

## Decisiones ya tomadas

- Se paga en la caja, no desde la mesa (Mercado Pago integrado en caja llega en la etapa 4).
- Todo sale de la barra: una sola cola de pedidos y una sola pantalla de preparación.
- Pedir no exige cuenta. La cuenta (afiliado) solo hace falta para puntos, favoritos sincronizados, regalos y amigos.
- Los puntos valen en un solo café por ahora, pero todas las tablas llevan `local_id` (multi-tenant, igual que Kiosco SaaS).
- El dueño define cuántos pesos vale un punto, qué productos se canjean y cuántos puntos cuesta cada uno.
- Los puntos se guardan en lotes (`lotes_puntos`) con vencimiento a 12 meses; se consumen primero los que vencen antes. El historial (`movimientos_puntos`) nunca se edita ni se borra.
- Transferir puntos solo entre amigos (amistad mutua), confirmado con PIN, en una única transacción con `SELECT … FOR UPDATE`.
- La caja ve el salón como plano dibujado (mesas con forma y sillas, color por estado).
- No hay menú 3D ni asistente con IA.
- Nunca se acepta un pedido por QR que nadie en el local puede ver: si la barra pierde conexión, la carta pasa sola a modo consulta (ver "Contingencia sin conexión" en la especificación). Nada de modo offline-first por ahora.

## Plan por etapas

1. MVP del salón: carta por QR, sesión de mesa, pedido, barra y salón en vivo, llamar al mozo, pedir la cuenta, cobro en caja, cierre de turno, factura electrónica ARCA, propina, botón de agotado, aprobación de pedidos en hora pico, contingencia sin conexión.
2. Club de puntos, tarjeta de sellos, sugerencias al pedir, rentabilidad por producto, carta deslizable, favoritos, compartir, reseñas, pedidos para llevar, impresora de comandas, app del dueño.
3. Amigos y transferencias, invitaciones con puntos, regalos periódicos, campañas, sorteos.
4. QR de Mercado Pago con el monto exacto en caja.
5. Extras: niveles, promos por horario, stock por receta, varios idiomas.

Trabajá de a una etapa. No adelantes funciones de etapas posteriores salvo que te lo pida.

## Reutilizar Kiosco SaaS

Ya existe Kiosco SaaS (repos `kiosco-saas`, `kiosco-saas-api`, `kiosco-saas-web` del usuario Ayrtondingo en GitHub): multi-local, usuarios y roles, productos, ventas, medios de pago y turnos de caja. Antes de crear algo de eso desde cero, revisá si se puede reutilizar y usá el mismo stack y las mismas convenciones. El club de puntos conviene hacerlo como módulo compartido entre los dos productos.

## Cómo trabajar

- Idioma del código y de la base: español para nombres de dominio (tablas, campos, eventos), como en la especificación.
- Montos en pesos argentinos, guardados como enteros (centavos) o decimal exacto; nunca float.
- El canal en vivo (WebSocket o SSE) solo avisa: la verdad está en la base. Al reconectar, cada pantalla vuelve a pedir el estado completo.
- El código del QR de mesa es aleatorio, nunca el número de mesa.
- Al proponer cambios sobre archivos existentes, mostrá qué buscar y con qué reemplazarlo en vez de reescribir archivos enteros.
- Antes de una migración o de un cambio grande, explicá el plan y esperá el visto bueno.

## Repositorios

El sistema vive en tres repos, igual que Kiosco SaaS:

```
cafe-qr/            ← este repo: CLAUDE.md, docs/ y .claude/ (agentes y comandos)
├── cafe-qr-api/    ← repo del backend (NestJS), clonado adentro e ignorado por este repo
└── cafe-qr-web/    ← repo del frontend (React), clonado adentro e ignorado por este repo
```

- Claude Code se abre en `cafe-qr/`, para ver la especificación y los dos repos a la vez.
- Los comandos de git (ramas, commits, push, PR) se corren **dentro del repo que corresponde**, nunca en `cafe-qr/` salvo que el cambio sea de documentación o de agentes.
- Una tarea puede tocar uno o los dos repos. Si toca los dos, se usa la misma rama (`feature/<nombre>`) en ambos y se abre un PR en cada uno, primero el de la API.
- El contrato entre API y web (endpoints, DTOs, eventos en vivo) se documenta en `docs/api/contratos.md`. Si cambia, primero se actualiza ahí, después la API y por último la web.
- Cada repo de código puede tener su propio `CLAUDE.md` corto con lo específico de ese repo (scripts, estructura de carpetas).

## Stack

- API: NestJS + TypeScript.
- Web (cliente, panel del local, barra y caja): React + TypeScript, como PWA.
- Base: PostgreSQL con Prisma 7 (mismo ORM y convenciones que Kiosco SaaS).
- Docker para el entorno local y para la base de los tests de integración.

## Calidad

- Una tarea no está terminada hasta que pasan lint, tests y build.
- Tests obligatorios para toda regla que toque pedidos, pagos, puntos, sesiones de mesa, permisos o contingencia.
- Lo que dependa de transacciones, bloqueos o concurrencia se testea contra PostgreSQL real, no con mocks.
- Cada endpoint valida el rol y toma el `local_id` del usuario autenticado o de la sesión de mesa, nunca del request.
- `pagos` y `movimientos_puntos` no se editan ni se borran: se anulan con un contramovimiento.
- Si la especificación no define una regla de negocio, preguntá; no la inventes.

## Agentes

La sesión principal implementa y coordina. Los subagentes están en `.claude/agents/`:

- **arquitecto**: planifica la tarea antes de escribir código. Solo escribe en `docs/`.
- **tester**: escribe y corre los tests. Solo toca archivos de test.
- **revisor**: revisa el diff antes del PR. Solo lectura.

Para una tarea completa usá `/tarea <descripción>`, que corre el flujo arquitecto → aprobación → implementación → tester → revisor → resumen. Para cambios chicos (un texto, un estilo) no hace falta el flujo completo, pero sí lint y tests.

`/calidad` configura las reglas de lint de arquitectura (dependency-cruiser) y el CI de GitHub Actions. Se corre una vez, cuando ya existe la estructura del repo; después, cada módulo nuevo se suma a las reglas como parte de su tarea.
