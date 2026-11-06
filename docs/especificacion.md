# Sistema para café: pedidos por QR y club de puntos

Especificación funcional y técnica. Versión del 5 de noviembre de 2026.

## Resumen

Un solo sistema web maneja todo el café: el cliente escanea el QR de su mesa, ve la carta, pide, sigue su pedido, llama al mozo, pide la cuenta y paga en la caja. El local ve en un panel qué pidió cada mesa, la barra prepara desde su pantalla y la caja cierra el turno por medio de pago.

Encima se suma un club de afiliados: quien tiene cuenta suma puntos con cada consumo, los canjea por productos y puede pasarle puntos a otro usuario que tenga agregado como amigo.

Tres decisiones de base:

- **Sin app para descargar.** Todo corre en el navegador (web instalable como PWA).
- **Pedir no exige registrarse.** La cuenta solo hace falta para sumar, canjear o transferir puntos, así nadie abandona el pedido por un formulario.
- **Todo queda registrado.** Cada pedido y cada pago se guarda aunque el cliente no tenga cuenta, igual que en Kiosco SaaS.

## Roles y pantallas

Hay cinco roles y tres pantallas: la del cliente (celular), la del personal (celular o tablet) y la de barra (tablet fija).

| Rol | Dónde lo usa | Qué hace |
| --- | --- | --- |
| Cliente | Su celular, desde el QR | Ve la carta, pide, sigue el pedido, llama al mozo, pide la cuenta, suma, canjea y transfiere puntos |
| Mozo | Celular o tablet | Recibe llamados y pedidos de cuenta, confirma pedidos, toma pedidos a mano, abre y cierra mesas |
| Barra | Tablet fija | Ve la cola de pedidos por mesa y marca cada uno como listo |
| Cajero | PC o tablet en caja | Cobra en el mostrador, registra el medio de pago, identifica al afiliado y aplica canjes, abre y cierra el turno |
| Dueño / admin | Panel web | Carga la carta y los precios, define mesas y QR, configura el club de puntos (pesos por punto, productos canjeables y su costo en puntos), ve reportes |

Una persona puede tener varios roles: en un café chico, el mozo suele cobrar también.

## Experiencia del cliente en la mesa

El cliente hace todo desde un link que abre el QR de su mesa. Todos los celulares que escanean la misma mesa comparten una **sesión de mesa**: un solo pedido abierto y una sola cuenta.

1. **Escanea el QR.** El QR lleva el código de la mesa (por ejemplo `cafe.com/m/A7K2`). Se abre la carta con "Mesa 4" arriba y, si no hay sesión abierta, se crea una.
2. **Ve la carta.** Categorías (cafés, desayunos, tortas, salados), foto, precio, descripción y opciones del producto (tamaño, tipo de leche, extra shot). Lo agotado aparece gris. Si el cliente inició sesión, cada producto muestra cuántos puntos suma.
3. **Arma el pedido y lo envía.** Agrega productos al carrito, con nota opcional ("sin azúcar"), y toca "Enviar pedido". Puede mandar varios pedidos en la misma visita; todos se suman a la cuenta de la mesa.
4. **Sigue el estado.** Recibido → en preparación → listo → entregado, actualizado en vivo.
5. **Llama al mozo.** Botón siempre visible. Al personal le llega la alerta con el número de mesa y el cliente ve "Ya avisamos al mozo". Se bloquea 60 segundos para evitar repeticiones.
6. **Pide la cuenta.** Ve el detalle de lo consumido en la mesa y el total. Si tiene cuenta, puede elegir un producto para canjear con sus puntos. La caja recibe el aviso y tiene la cuenta lista cuando el cliente se acerca.
7. **Paga en la caja y suma puntos.** Dice su número de mesa y paga con cualquier medio. Para sumar puntos se identifica con su QR personal o su teléfono. Al registrarse el pago, la sesión se cierra, la mesa queda libre y se acreditan los puntos.

**Que nadie pida desde su casa.** El primer pedido de cada sesión entra como "a confirmar" y el mozo lo acepta con un toque; los siguientes entran directo. Alternativa: el mozo abre la mesa al sentar al cliente.

## Favoritos, compartir y reseñas

Ninguna de estas funciones obliga a tener cuenta para pedir; la cuenta solo suma beneficios.

**Carta deslizable.** Al tocar un producto se abre a pantalla completa con su foto vertical; deslizando aparece el siguiente de la misma categoría. Desde ahí se agrega al pedido, se marca como favorito o se comparte. Sin foto vertical se muestra la tarjeta común.

**Favoritos.** El corazón guarda el producto en "Mis favoritos". Sin cuenta se guarda en el celular; al registrarse pasa a la cuenta. Con favoritos e historial aparece el botón "Lo de siempre", que repite el último pedido. El contador de corazones es opcional (lo activa el dueño). El dueño ve el ranking de favoritos y puede usarlo en campañas.

**Compartir un producto.** Genera un link (por ejemplo `cafe.com/p/cortado`) con vista previa de foto, nombre y precio. Quien lo abre ve la carta en modo consulta, sin mesa, y no puede pedir. Si quien comparte es afiliado, el link lleva su código de invitación: cuando el invitado se registra y paga su primera compra, los dos reciben puntos (por ejemplo 200) y quedan agregados como amigos. El premio se da recién con la primera compra pagada y hay un tope de invitaciones premiadas por mes.

**Reseñas.** Después de pagar, el celular pregunta cómo estuvo la visita: 1 a 5 estrellas, comentario opcional y nota opcional por producto. Solo opinan quienes pagaron. El dueño las ve y puede responderlas. Con 3 estrellas o menos le llega un aviso para contactar al cliente. La puntuación de un producto se muestra en la carta desde 5 opiniones.

El botón para dejar reseña en Google se ofrece a todos por igual y sin premio a cambio: Google prohíbe pedir reseñas solo a clientes contentos y dar incentivos.

## Panel del local

El personal usa un panel con cinco pestañas, todas en vivo. Cada rol ve solo las que le corresponden.

| Pestaña | Quién la usa | Qué muestra | Acciones |
| --- | --- | --- | --- |
| Resumen | Dueño, encargado | Ventas del turno, pedidos activos, mesas ocupadas, pedidos por estado | Ir a una mesa o a un pedido |
| Salón | Mozo | Plano de mesas por color: libre, ocupada, pidió, llamó al mozo, pidió la cuenta | Atender un llamado, confirmar el primer pedido, abrir o cerrar la mesa, cargar un pedido a mano |
| Mesa (detalle) | Mozo, cajero | Todo lo pedido en esa mesa, con hora y estado, y el total parcial | Agregar o anular ítems (la anulación pide motivo), cambiar de mesa, dividir la cuenta |
| Barra | Barista | Tarjetas por pedido: mesa, ítems, notas y minutos de espera. Las más viejas primero, en rojo pasados los 10 minutos | Marcar "en preparación" y "listo" |
| Caja | Cajero | Plano del salón con cada mesa dibujada, su estado y su total; cobros del turno por medio de pago; canjes de puntos | Buscar la cuenta por número de mesa, identificar al afiliado (QR o teléfono), cobrar total o en partes, registrar el medio de pago, cerrar el turno |

**Todo sale de la barra.** Una sola cola y una sola pantalla. Si más adelante se suma una cocina, alcanza con asignar cada producto a una estación.

**Cierre de turno.** El cajero declara el efectivo y el sistema lo compara con lo registrado (arqueo). El resumen muestra totales por medio de pago, productos más vendidos, puntos entregados y canjeados.

### Pantalla de caja: plano del salón

El cajero ve el salón dibujado: cada mesa en su lugar, con su forma (redonda, cuadrada, larga) y sus sillas, pintada según su estado y con lo consumido. Colores: libre (sin relleno), ocupada (azul), pidió la cuenta (amarillo), llamó al mozo (rojo). Al tocar una mesa se abre a la derecha el detalle: ítems, total, afiliado identificado, botones de medio de pago (efectivo, Mercado Pago, débito, crédito) y "Cobrar". El dueño arma el plano una vez desde su panel arrastrando mesas. El mozo ve el mismo plano en la pestaña Salón.

## Club de puntos

El afiliado suma puntos por cada peso pagado y los canjea por productos. El dueño define cuántos pesos hay que gastar para ganar un punto, qué productos se canjean y cuántos puntos cuesta cada uno. Las cifras de esta sección son ejemplos.

**Alta.** Desde el celular, con email o Google y teléfono verificado por código. En la caja el cliente se identifica con su QR personal o su teléfono. Bono de bienvenida opcional (por ejemplo 100 puntos).

**Cómo se suman.**

- 1 punto cada $X pagados; el dueño fija $X (por ejemplo $100). Se calcula sobre el total cobrado, después de descuentos y canjes, redondeando para abajo.
- Los puntos van a quien paga. Si la cuenta se divide, cada parte suma para quien la pagó.
- Se acreditan al registrar el pago, nunca al enviar el pedido. Si se anula la venta, se descuentan.
- Promos opcionales: puntos dobles en horarios o productos, bono de cumpleaños.

**Canje.** El dueño elige qué productos de la carta entran en el canje y su costo en puntos (por ejemplo café chico 400, medialuna 300, desayuno completo 1.500), y puede activarlos o pausarlos. En la carta esos productos muestran su precio en puntos. El cliente los agrega como ítem a $0, o el cajero aplica el canje al cobrar. Si cambia un costo, los canjes ya reservados mantienen el anterior. Los puntos se reservan al pedir el canje y se descuentan al cobrar; si la cuenta se cancela, vuelven.

**Vencimiento.** A los 12 meses de ganados, con aviso 30 días antes.

**Niveles (más adelante).** Bronce, Plata y Oro según puntos de los últimos 6 meses, con multiplicadores 1x, 1,25x y 1,5x.

## Amigos y transferencia de puntos

Solo se pueden pasar puntos a un amigo, y la amistad es mutua: uno manda la solicitud y el otro la acepta.

**Agregar un amigo.** Cada afiliado tiene un código corto (por ejemplo `AYR-4821`) y un QR personal. Se agrega escaneando el QR, escribiendo el código o buscando por teléfono (si el otro lo permite). La amistad se puede eliminar o bloquear.

**Pasar puntos.**

1. El usuario entra a "Amigos", elige uno y toca "Pasar puntos".
2. Escribe la cantidad y ve el resumen: "Le pasás 500 puntos a Juli. Te quedan 820."
3. Confirma con su PIN de 4 dígitos (o la huella del celular).
4. El sistema descuenta y acredita en una sola transacción.
5. Los dos reciben un aviso y el movimiento queda en su historial.

**Reglas contra abusos.**

| Regla | Valor inicial | Para qué |
| --- | --- | --- |
| Antigüedad mínima para transferir | 1 compra pagada o 7 días de cuenta | Que una cuenta recién creada no vacíe su bono |
| Puntos transferibles | Solo los ganados por compras; bonos y promos no | Evitar juntar bonos de cuentas falsas |
| Mínimo por transferencia | 50 puntos | Evitar movimientos basura |
| Máximo por día / por mes | 1.000 / 5.000 puntos enviados | Limitar la reventa |
| Vencimiento | El punto transferido conserva su fecha original | Que transferir no estire el vencimiento |
| Reversión | Solo el admin, con motivo | Evitar reclamos de "me arrepentí" |

El admin ve un reporte de transferencias y recibe alertas si una cuenta recibe de muchas cuentas distintas o si varias cuentas comparten dispositivo o teléfono.

## Regalos, campañas, sorteos, sugerencias y rentabilidad

Todo se maneja desde el panel del dueño y se mide (cuánto vendió o cuánto se usó).

### Regalos periódicos

Beneficio que aparece en "Mis regalos" del afiliado y vence.

| Tipo de regalo | Ejemplo |
| --- | --- |
| Descuento en porcentaje | 20% en tu próximo desayuno |
| Descuento en monto | $2.000 en compras desde $8.000 |
| Producto gratis | Una medialuna con tu café |
| 2x1 | Dos cortados al precio de uno |
| Puntos extra | 200 puntos de regalo |

Cuándo se manda: automático cada cierto tiempo (por ejemplo el primer lunes de cada mes a quienes compraron en los últimos 60 días), automático por evento (cumpleaños, 10 visitas, 30 días sin venir) o a mano (a un cliente o a un grupo).

Reglas: un solo uso, uno por cuenta, no se suma a otro descuento, con vencimiento (por ejemplo 15 días) y opcionalmente limitado a días u horarios ("lunes a jueves de 15 a 18"). Se elige al pedir o se muestra en caja; el cajero lo ve aplicado.

### Campañas a clientes que dejaron de venir

Una campaña junta un grupo, un mensaje y un regalo opcional. Grupos por reglas: no vino hace N días, viene más de 4 veces por mes, cumple años este mes, nunca usó sus puntos, todos. Canales: notificación push de la PWA (gratis), email y WhatsApp (la API oficial cobra por mensaje). Tope de un mensaje por semana por cliente y opción de baja. Métricas: recibidos, volvieron en 15 días, usaron el regalo, gasto.

### Sorteos

Premio, fecha y condición (por ejemplo: afiliados con una compra en noviembre; cada visita suma una chance). Participantes anotados solos, ganador al azar en la fecha, resultado guardado con participantes y semilla para auditar. El premio llega como regalo.

### Sugerencias al pedir

Al agregar un producto, el carrito sugiere hasta dos acompañamientos ("¿Lo acompañás con una medialuna?"). Primero las elige el dueño por producto o categoría; más adelante pueden salir de lo que más se pide junto. Reporte de aceptación y ventas sumadas.

### Rentabilidad por producto

El dueño carga el costo de cada producto. Reporte por período: unidades, ventas, costo, margen en pesos y porcentaje. Clasificación: estrellas (mucha venta, mucho margen), populares de bajo margen, rentables poco pedidos (candidatos para sugerencias y regalos) y para revisar. Alerta si un producto queda por debajo del margen mínimo. Con stock por receta, el costo se calcula solo.

## Modelo de datos

Tres bloques: salón, carta y club de puntos. Todas las tablas llevan `local_id`. La configuración del local guarda cuántos pesos vale un punto.

| Tabla | Campos clave | Relación |
| --- | --- | --- |
| `mesas` | número, `codigo_qr`, sector, forma, posición en el plano, activa | Un local tiene muchas mesas |
| `sesiones_mesa` | `mesa_id`, abierta_en, cerrada_en, estado (abierta / cuenta pedida / cerrada) | Una sesión abierta por mesa a la vez |
| `pedidos` | `sesion_id`, origen (QR / mozo / mostrador), estado, confirmado_por, `afiliado_id` opcional | Una sesión tiene muchos pedidos |
| `pedido_items` | `pedido_id`, `producto_id`, cantidad, precio_unitario, opciones, nota, estado | Precio copiado al pedir |
| `llamados` | `sesion_id`, tipo (mozo / cuenta), estado, atendido_por | Una sesión tiene muchos llamados |
| `pagos` | `sesion_id`, monto, medio, propina, `afiliado_id` opcional, `turno_id`, `en_contingencia` | Varios pagos por sesión (cuenta dividida) |
| `turnos_caja` | abierto_por, apertura, cierre, efectivo_declarado, diferencia | Un turno tiene muchos pagos |
| `categorias`, `productos`, `opciones_producto` | nombre, precio, costo, margen mínimo, foto, disponible | La carta |
| `afiliados` | email, teléfono verificado, `codigo_amigo`, `pin_hash`, privacidad | Cliente con cuenta |
| `amistades` | `afiliado_a`, `afiliado_b`, solicitado_por, estado (pendiente / aceptada / bloqueada) | Una fila por par, con a < b |
| `lotes_puntos` | `afiliado_id`, origen (compra / bono / transferencia), cantidad, saldo, vence_en, transferible | Cada ingreso de puntos es un lote |
| `movimientos_puntos` | `afiliado_id`, `lote_id`, cantidad (+/−), tipo, `pago_id` / `transferencia_id` / `canje_id` | Historial inmutable |
| `transferencias` | de_afiliado, a_afiliado, cantidad, creada_en, revertida_por | Origina dos movimientos |
| `premios`, `canjes` | costo en puntos, `producto_id`; estado del canje (reservado / aplicado / devuelto) | Productos habilitados para canje |
| `sugerencias` | `producto_id`, `sugerido_id`, orden | Sugerencias al pedir |
| `regalos` | tipo, valor, condiciones, días y horarios, días de validez | Plantilla de regalo |
| `regalos_afiliado` | `afiliado_id`, `regalo_id`, estado (disponible / usado / vencido), vence_en, `pago_id` | Regalo entregado |
| `campanias` | segmento (regla), mensaje, canal, `regalo_id`, programación | Campañas |
| `envios` | `campania_id`, `afiliado_id`, canal, enviado_en | Para medir y respetar el tope semanal |
| `sorteos`, `sorteo_participantes` | premio, fecha, condición, semilla; chances por afiliado | Sorteos auditables |
| `favoritos` | `afiliado_id`, `producto_id` | Favoritos |
| `resenas`, `resenas_producto` | `sesion_id`, `afiliado_id` opcional, estrellas, comentario, respuesta; estrellas por producto | Reseñas verificadas |
| `invitaciones` | código, invitador, invitado, estado, puntos entregados | Referidos |

**Por qué lotes.** Cada ingreso de puntos es un lote con vencimiento y marca de transferible. Al gastar o transferir se consumen primero los lotes que vencen antes. Así salen solas tres reglas: vencimiento a 12 meses, bonos no transferibles y conservación de la fecha al transferir. El saldo es la suma de los lotes vigentes.

**Transferencia segura.** Una sola transacción con `SELECT … FOR UPDATE` sobre los lotes del que envía, para que dos transferencias simultáneas no gasten los mismos puntos.

## Estados, tiempo real y arquitectura

Cada pantalla (celular del cliente, panel del personal, pantalla de barra) manda sus acciones a la API por HTTPS. La API guarda en la base y publica un evento; el canal en vivo (WebSocket) lo reparte al resto. El cliente se suscribe a la sala de su mesa; personal y barra, a la del local.

**Estados del pedido:** a confirmar → confirmado → en preparación → listo → entregado. Antes de "listo" se puede anular con motivo. Solo el primer pedido de la sesión pasa por "a confirmar" (o todos, si el dueño activa la aprobación en hora pico).

**Estados de la mesa:** libre → ocupada → cuenta pedida → libre (al registrar el pago). Un llamado al mozo es un aviso aparte, pendiente hasta que alguien lo atiende.

| Evento | Lo reciben | Efecto en pantalla |
| --- | --- | --- |
| `pedido.nuevo` | Salón, barra | Tarjeta nueva en la cola; la mesa cambia de color |
| `pedido.estado` | Mesa, salón | Avanza el seguimiento del cliente |
| `llamado.nuevo` | Salón, caja | Alerta con sonido: en el salón si llaman al mozo, en la caja si piden la cuenta |
| `llamado.atendido` | Mesa, salón | "El mozo ya va" en el celular del cliente |
| `mesa.estado` | Salón, caja | Plano y lista de cuentas actualizados |
| `pago.registrado` | Mesa, caja | Cierre de la sesión y puntos acreditados |
| `local.conexion` | Mesas, salón, caja | La carta pasa a modo consulta o vuelve a aceptar pedidos (ver "Contingencia sin conexión") |

**Detalles técnicos.**

- WebSocket (por ejemplo Socket.IO) o Server-Sent Events, porque casi todo el tráfico va del servidor a las pantallas.
- El canal solo avisa; la verdad está en la base. Al reconectar, cada pantalla pide el estado completo.
- El código del QR es aleatorio, no el número de mesa, y se puede regenerar.
- La carta se cachea en el celular para abrir rápido con wifi malo.

## Contingencia sin conexión

Regla de base: **nunca se acepta un pedido que nadie en el local puede ver.** Si el sistema no puede garantizarlo, deja de aceptar pedidos por QR y el café trabaja como siempre: el mozo toma el pedido y la caja cobra.

### Tres situaciones

| Situación | Qué pasa con la carta | Qué pasa con los pedidos | Qué hace el sistema |
| --- | --- | --- | --- |
| Se cae el wifi del café; el servidor anda | El cliente la ve con los datos de su celular | Llegarían al servidor, pero barra y caja no los ven | Detecta que las pantallas del local se desconectaron y pausa los pedidos por QR |
| Se cae el servidor o la API | Se ve la carta de respaldo (ver abajo), sin botón de pedir | No se puede pedir por QR | La carta muestra "Pedile al mozo" |
| El cliente no tiene datos y el wifi está caído | No se ve | No se puede pedir por QR | Nada: se usan las cartas impresas |

### Pausa automática de pedidos por QR

- Las pantallas de barra y de caja mandan una señal de vida al servidor cada 10 segundos.
- Si la barra no da señales durante 30 segundos (valor configurable por el dueño), el local pasa a **modo consulta**: la carta se sigue viendo, pero el botón "Enviar pedido" se reemplaza por "Ahora mismo pedile al mozo". También se pausan "Llamar al mozo" y "Pedir la cuenta", con el mismo mensaje.
- Cuando la barra vuelve a dar señales, los pedidos por QR se reactivan solos.
- El dueño o el encargado también pueden pausar los pedidos por QR a mano desde el panel (por ejemplo, si se rompió la tablet de barra).
- Los pedidos que entraron justo antes de detectar el corte (como máximo los últimos 30 segundos) aparecen resaltados en la barra al reconectar, porque al volver cada pantalla pide el estado completo.
- El evento `local.conexion` avisa a los celulares de la sala del local el cambio entre modo normal y modo consulta.

### Envío de pedidos con conexión inestable

- Cada envío de pedido lleva una clave única generada en el celular (clave de idempotencia). Si el cliente toca dos veces o el celular reintenta por mala señal, el servidor registra el pedido una sola vez.
- Si el envío falla, el carrito no se borra y el cliente ve "No se pudo enviar. Reintentá o pedile al mozo".

### Carta de respaldo

- La web del cliente se sirve como archivos estáticos desde un CDN, separada de la API.
- Cada vez que el dueño cambia la carta, el sistema publica también una copia estática (`carta.json` con las fotos) en ese CDN.
- Si la API no responde, la web muestra esa copia en modo consulta, con los precios y el aviso "Pedile al mozo".
- Además, el local tiene algunas cartas impresas en la barra.

### Cómo se trabaja durante el corte

| Tarea | Procedimiento |
| --- | --- |
| Tomar pedidos | El mozo los anota en papel y se los pasa a la barra |
| Cobrar | Efectivo normal; tarjeta con el posnet (tiene su propia conexión); el cajero anota cada cobro en papel con mesa, total y medio de pago |
| Factura ARCA | Se cobra igual y el comprobante se emite al volver la conexión. Mecanismo a confirmar con el contador (CAE diferido o CAEA) |
| Puntos | Se acreditan después con el ticket, dentro del plazo de acreditación posterior |

### Carga posterior al volver la conexión

- La caja tiene la pantalla **"Cargar ventas del corte"**: el cajero carga cada venta anotada en papel con la hora real, la mesa (opcional), los productos y el medio de pago.
- Esas ventas y sus pagos quedan marcadas con `en_contingencia = true`, quién las cargó y cuándo. Van al turno en el que ocurrieron.
- Los comprobantes ARCA pendientes quedan en una cola que reintenta sola, y la caja muestra cuántos faltan emitir.
- No se puede cerrar el turno con ventas del corte sin cargar si el cajero indicó que hubo un corte. El resumen del turno muestra aparte las ventas del corte.

### Requisito operativo (fuera del software)

Router con respaldo 4G (o, como mínimo, un celular que comparta datos) para la tablet de barra y la caja. Es lo que más reduce la cantidad de cortes que llegan a afectar al servicio.

### Fuera de alcance por ahora

Panel que funcione sin internet y sincronice después (offline-first) y servidor local en el café. Se reconsidera solo si los cortes resultan frecuentes en ese local.

## Funciones tomadas de la competencia

| Idea | Etapa |
| --- | --- |
| Factura electrónica A, B y C con CAE de ARCA, enviada por WhatsApp; datos fiscales guardados | 1 |
| Propina al cobrar, asignada al mozo, con reporte mensual | 1 |
| Botón de agotado que oculta el producto (automático por stock en etapa 5) | 1 |
| Aprobar todos los pedidos a mano en hora pico | 1 |
| Rentabilidad por producto | 2 |
| Pedido para llevar con QR en el mostrador | 2 |
| Sugerencias al pedir | 2 |
| Impresora térmica de comandas además de la pantalla | 2 |
| App de monitoreo para el dueño | 2 |
| Tarjeta de sellos digital ("el 10.º café es gratis") | 2 |
| Sorteos | 3 |
| Campañas a clientes | 3 |
| Stock por receta | 5 |
| Carta en varios idiomas | 5 |
| Integración con PedidosYa y Rappi | Futuro |
| Reservas de mesa | Futuro |

## Reutilización de Kiosco SaaS

| Parte | De Kiosco SaaS | Nuevo |
| --- | --- | --- |
| Multi-local, usuarios y roles | Se reutiliza | Roles nuevos: mozo, barra |
| Productos y categorías | Se reutiliza | Foto, opciones, disponible, costo, costo en puntos |
| Ventas, medios de pago, turnos de caja | Se reutiliza | Pago ligado a sesión de mesa, cuenta dividida, propina |
| Club de afiliados | Ya planeado para el maxikiosco | Amigos, transferencias, lotes con vencimiento |
| Mesas, sesiones, QR, llamados, barra, plano | — | Todo nuevo |

El club de puntos conviene hacerlo como módulo compartido entre el maxikiosco y el café.

## Plan por etapas

1. **MVP del salón.** Carta por QR, pedido, sesión de mesa, panel de salón y barra en vivo, llamar al mozo, pedir la cuenta, cobro en caja con medio de pago, plano del salón en caja, cierre de turno, factura electrónica ARCA, propina, botón de agotado, aprobación de pedidos en hora pico y contingencia sin conexión (pausa automática, carta de respaldo y carga posterior).
2. **Club de puntos y experiencia.** Registro con teléfono, puntos según los pesos por punto del dueño, productos canjeables, canje, vencimiento, historial, tarjeta de sellos, sugerencias, rentabilidad, carta deslizable, favoritos, compartir, reseñas, pedidos para llevar, impresora de comandas y app del dueño.
3. **Amigos y marketing.** Código y QR de amigo, solicitudes, transferencia con PIN, límites, reporte para el admin, invitaciones con puntos, regalos periódicos, campañas y sorteos.
4. **Mercado Pago en la caja.** QR con el monto exacto y acreditación automática. Pagar desde la mesa queda para más adelante.
5. **Extras.** Niveles, promos por horario, reportes de horas pico, stock por receta, varios idiomas, cocina separada si hace falta.

## Decisiones tomadas

- Se paga en la caja.
- No hay cocina aparte: todo sale de la barra.
- El dueño define el valor del punto, los productos canjeables y su costo en puntos.
- Los puntos sirven en un solo café por ahora; agregar sucursales luego implica agrupar locales bajo una marca.
- Sin menú 3D ni asistente con IA.
