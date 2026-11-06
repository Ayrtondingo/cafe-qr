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

1. **Escanea el QR.** El QR lleva el código de la mesa (por ejemplo `cafe.com/m/A7K2M9QX`; formato en "Reglas confirmadas", regla 7). Se abre la carta con "Mesa 4" arriba y, si no hay sesión abierta, se crea una.
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

## Reglas confirmadas

Reglas de la etapa 1 que el usuario confirmó el 2026-11-05 y amplió el 2026-11-06, después de la migración inicial (`20261106021213_modelo_etapa_1`). Prevalecen sobre cualquier texto anterior de esta especificación que las contradiga. Los valores de enum van en mayúsculas en la base.

### 1. Idempotencia del envío de pedidos

*Confirmada el 2026-11-05. Complementa "Envío de pedidos con conexión inestable". ADR: ninguno (el modelo ya la soporta, ADR 0005).*

- La clave de idempotencia es única **por local**.
- Misma clave + misma sesión + mismo payload (se compara el hash del payload): se devuelve el pedido original con **200**, sin crear otro.
- Clave que ya existe con **otra sesión** o con **otro payload**: **409 `CLAVE_IDEMPOTENCIA_REUTILIZADA`**, sin ningún dato del otro pedido.
- Ante el 409, la web muestra "No se pudo enviar, reintentá" y genera una clave nueva.

### 2. Contramovimientos de pagos

*Confirmada el 2026-11-05. ADR 0006 (complementa al ADR 0004).*

La columna que referencia al pago anulado es `anula_a_id` (ADR 0004), el `anula_pago_id` de la regla.

- `anula_a_id` es obligatorio en un contramovimiento y apunta a un pago del **mismo local** y de la **misma sesión**.
- Se permiten **anulaciones parciales**. La suma de las anulaciones de un pago no supera su monto; la propina se controla por separado (la suma de las propinas anuladas no supera la propina del pago).
- Un contramovimiento **no se puede anular**. Si fue un error, se registra un pago nuevo.
- El contramovimiento usa el **mismo medio de pago** que el original.
- Se imputa al **turno abierto actual**, no al del pago original.
- Lo registra un **cajero o el encargado**, siempre con motivo (código y comentario, regla 8). *Ajustado el 2026-11-06:* si la sesión del pago está cerrada, ver "Anular un pago de una sesión cerrada" abajo.
- En la base: signo del monto, FK y "el pago anulado no es un contramovimiento". En el servicio: el tope de la suma, con `SELECT … FOR UPDATE` sobre el pago original.

**Anular solo la propina** (*confirmada el 2026-11-06*):

- Se puede anular solo la propina: contramovimiento con `monto = 0` y `propina` negativa. Al menos uno de los dos tiene que ser distinto de cero (los dos son ≤ 0).
- Se refleja en el reporte de propinas del **mes de la anulación**.

**Anular un pago de una sesión cerrada** (*confirmada el 2026-11-06*):

- La sesión **no se reabre**.
- El contramovimiento queda ligado a la **sesión original** y se imputa al **turno actual**.
- Error de medio de pago: se registra un **pago de corrección** en la misma sesión cerrada.
- Devolución al cliente: solo el contramovimiento.
- **Permisos:** lo hace el **encargado**; si el turno del pago original ya cerró, **solo el dueño**. Esto reemplaza, para pagos de sesiones cerradas, el "cajero o encargado" de arriba; con la sesión abierta sigue valiendo "cajero o encargado".
- Si el pago estaba facturado, la anulación va con **nota de crédito** y el pago de corrección lleva su propia factura. *Dependencia:* comprobantes ARCA, pospuestos (ADR 0005, P6); no se diseña ahora.
- En la etapa 2, si el pago sumó puntos, se descuentan **en proporción**. *Dependencia:* módulo puntos (etapa 2); no se diseña ahora.

### 3. Opciones de producto al crear el pedido

*Confirmada el 2026-11-05. Complementa P11 del ADR 0005.*

- El servicio valida que cada opción elegida pertenezca a un grupo del producto del ítem y esté disponible, y respeta el mínimo y el máximo de cada grupo y los grupos obligatorios.
- El precio lo recalcula el servidor: se ignora el que manda el cliente y se copia al ítem el precio de cada opción.
- En la base: FK compuesta `(producto_id, opcion_id)` en las opciones del ítem (`pedido_item_opciones`). Nada de triggers para mínimos y máximos: eso lo valida el servicio.

### 4. Quién cargó el pedido

*Confirmada el 2026-11-05. ADR: ninguno.*

- Todo pedido con origen distinto de `QR` (`MOZO` o `MOSTRADOR`) exige `cargado_por`, con CHECK en la base.
- Los pedidos de origen `QR` quedan identificados por dispositivo y clave de idempotencia.

### 5. Cierre de la sesión de mesa

*Confirmada el 2026-11-05. ADR: ninguno (complementa P10 del ADR 0005).*

- `motivo_cierre` es obligatorio al cerrar: `PAGO`, `ABANDONO`, `MANUAL` o `AUTOMATICO`.
- `cerrada_por` es obligatorio salvo con motivo `AUTOMATICO`.
- Con `PAGO`, `cerrada_por` es quien registró el pago final.
- `ABANDONO` y `MANUAL`: solo el encargado, con motivo (código y comentario, regla 8). El saldo pendiente queda registrado como pérdida.

**Pérdidas** (*confirmada el 2026-11-06*):

- Tabla `perdidas` ligada al turno: `sesion_id`, `turno_id`, `monto`, `codigo_motivo`, `comentario`, `registrada_por`, `creada_en`.
- Se registra al cerrar una sesión por `ABANDONO` o `MANUAL` **con saldo pendiente**.
- Es **inmutable**.
- Aparece en el resumen del turno y **no afecta el arqueo de efectivo**.

### 6. Pedir la cuenta y estado de la mesa

*Confirmada el 2026-11-05. ADR 0007.*

- El **estado de la sesión** es la fuente de verdad; el llamado de tipo `CUENTA` es solo el aviso para la caja.
- Pedir la cuenta es **una transacción** que pone la sesión en `CUENTA_PEDIDA` y crea el llamado.
- Atender el llamado **no cambia** el estado de la sesión; solo el pago la cierra.
- Color de la mesa, por prioridad: **rojo** (llamado al mozo pendiente) > **amarillo** (`CUENTA_PEDIDA`) > **ocupada** > **libre**.
- Si llega un pedido nuevo después de pedir la cuenta, se acepta, la sesión vuelve a `ABIERTA` y la caja recibe un aviso de que la cuenta cambió.
- *Confirmada el 2026-11-06:* cuando la sesión vuelve a `ABIERTA` por un pedido nuevo, el llamado de cuenta pendiente pasa **automáticamente** a `CANCELADO` (estado nuevo de los llamados). Si el cliente vuelve a pedir la cuenta, se crea un **llamado nuevo**.

### 7. Código QR de la mesa

*Confirmada el 2026-11-05. ADR: ninguno.*

- 8 caracteres en **Crockford base32**: mayúsculas, sin I, L, O ni U (alfabeto `0123456789ABCDEFGHJKMNPQRSTVWXYZ`).
- Generado con un generador criptográfico.
- Único en **todo el sistema**, no por local.
- La entrada se normaliza a mayúsculas antes de buscar.
- Regenerar invalida el código anterior **al instante**; el código viejo muestra "Este QR ya no es válido, pedile al mozo".
- Límite de consultas por IP en `/m/:codigo`.

**QR desconocido** (*confirmada el 2026-11-06*):

- Cualquier código que no corresponda a una mesa activa (inexistente, regenerado o de una mesa desactivada) recibe **la misma respuesta**, con el mismo código HTTP y el mensaje "Este QR ya no es válido, pedile al mozo".
- **No se guardan** los códigos viejos.
- *Propuesta para `docs/api/contratos.md` (tarea de salón):* `404` con código `QR_NO_VALIDO`, igual en los tres casos.

**Token de sesión de mesa y expulsar dispositivo** (*confirmada el 2026-11-06*):

- Regenerar el QR **no corta** a los dispositivos de una sesión abierta.
- Al escanear, el dispositivo recibe un **token de sesión de mesa**, independiente del código QR, válido hasta que se cierra la sesión.
- Se agrega la acción **"expulsar dispositivo"**, para el mozo o el encargado.

### 8. Motivos codificados

*Confirmada el 2026-11-06. ADR: ninguno.*

- En todo lo que pide motivo (cierre de sesión por `ABANDONO` o `MANUAL`, anulación de ítems, contramovimientos y pérdidas) se guarda un `codigo_motivo` de una **lista fija** más un `comentario` opcional. El comentario es **obligatorio si el código es `OTRO`**.
- Listas iniciales:
  - Cierre de sesión: `SE_FUE_SIN_PAGAR`, `ERROR_DE_CARGA`, `MESA_DUPLICADA`, `OTRO`.
  - Anulaciones y contramovimientos: `ERROR_DE_COBRO`, `PRODUCTO_MAL_HECHO`, `CLIENTE_ARREPENTIDO`, `OTRO`.

### Cambios pendientes en el modelo

Lo que hay que cambiar en `cafe-qr-api/prisma/schema.prisma` y en la migración `20261106021213_modelo_etapa_1` para cumplir estas reglas. La migración inicial todavía no está mergeada en `develop` (PR abierto, sin datos reales): los cambios se pueden hacer editando esa migración o con una migración nueva, y ninguno pierde datos reales. Igual se marcan con ⚠️ los que, aplicados como migración nueva, quitan o reemplazan algo existente.

| # | Regla | Cambio | Tarea | Destructivo |
| --- | --- | --- | --- | --- |
| 1 | 4 | Reemplazar el CHECK `pedidos_mozo_con_carga` (`origen <> 'MOZO' OR cargado_por_id IS NOT NULL`) por `pedidos_no_qr_con_carga`: `origen = 'QR' OR cargado_por_id IS NOT NULL`. Actualizar el comentario `SQL:` de `Pedido`. | pedidos | ⚠️ Reemplaza un CHECK (sin datos: sin impacto) |
| 2 | 1 | Ninguno en la base: `@@unique([localId, claveIdempotencia])` y `huella_solicitud` ya existen. Corregir el comentario de `huellaSolicitud` para decir que la huella es el hash del payload normalizado. La comparación de sesión y huella, y la carrera de dos envíos simultáneos con la misma clave (violación del único → releer y comparar), van en el servicio. | pedidos | No |
| 3 | 5 | Nuevo enum `MotivoCierreSesion` (`PAGO`, `ABANDONO`, `MANUAL`, `AUTOMATICO`) y columna `sesiones_mesa.motivo_cierre` (nullable). Ampliar el CHECK `sesiones_mesa_cierre_completo` a `(estado = 'CERRADA') = (cerrada_en IS NOT NULL AND motivo_cierre IS NOT NULL)` y agregar `sesiones_mesa_cerrada_por`: `estado <> 'CERRADA' OR motivo_cierre = 'AUTOMATICO' OR cerrada_por_id IS NOT NULL`. Que `cerrada_por` sea quien registró el pago final y que `ABANDONO`/`MANUAL` sean solo del encargado se valida en el servicio. | salon | ⚠️ Reemplaza un CHECK (sin datos: sin impacto) |
| 4 | 5 (pérdidas) | Tabla nueva `perdidas` (módulo caja, porque va ligada al turno): `id`, `local_id`, `sesion_id`, `turno_id`, `monto Decimal(14,2)`, `codigo_motivo` (enum `CodigoMotivoCierre`, el mismo de la sesión: *interpretación*, porque las pérdidas salen de un cierre), `comentario`, `registrada_por_id`, `creada_en`. FK compuestas `(local_id, sesion_id) → sesiones_mesa`, `(local_id, turno_id) → turnos_caja`, `(local_id, registrada_por_id) → usuarios`. CHECK `monto > 0`; CHECK comentario obligatorio con `OTRO`. *Propuesta:* único `(local_id, sesion_id)`, porque una sesión se cierra una sola vez. Inmutable como `pagos`: triggers `perdidas_inmutables` (UPDATE/DELETE) y `perdidas_sin_truncate` con la función existente `rechazar_cambio_inmutable`; los tests la vacían deshabilitando el de TRUNCATE, como con `pagos`. Solo `creada_en`, sin `actualizado_en` (ADR 0001). Salon la registra llamando al servicio de caja dentro de la transacción del cierre. El arqueo de efectivo no la suma. | caja / salon | No (tabla nueva) |
| 5 | 3 | Agregar `producto_id` a `opciones_producto` con FK `(local_id, producto_id, grupo_id) → grupos_opciones(local_id, producto_id, id)` (requiere único `grupos_opciones(local_id, producto_id, id)`), y único `opciones_producto(local_id, producto_id, id)`. Así la opción no puede quedar en un producto distinto del de su grupo. | carta | No (columna nueva NOT NULL: sin datos, sin impacto) |
| 6 | 3 | Agregar `producto_id` a `pedido_item_opciones`; reemplazar la FK `(local_id, opcion_id) → opciones_producto` por `(local_id, producto_id, opcion_id) → opciones_producto(local_id, producto_id, id)`, y la FK `(local_id, pedido_item_id) → pedido_items` por `(local_id, pedido_item_id, producto_id) → pedido_items(local_id, id, producto_id)` (requiere único `pedido_items(local_id, id, producto_id)`), para que la opción sea del mismo producto que el ítem. | pedidos | ⚠️ Reemplaza dos FK (sin datos: sin impacto) |
| 7 | 2 | Contramovimientos (detalle en ADR 0006): quitar el único `pagos_local_id_anula_a_id_key` (impide las anulaciones parciales); reemplazar la FK `(local_id, anula_a_id) → pagos(local_id, id)` por la FK compuesta de ADR 0006 (misma sesión, mismo medio, el anulado no es contramovimiento), con sus columnas auxiliares, CHECKs y el único de destino; agregar índice `(local_id, anula_a_id)` no único para sumar las anulaciones; actualizar los comentarios de `Pago` ("Un pago se anula una sola vez" deja de valer). | caja | ⚠️ Quita un índice único y reemplaza una FK (sin datos: sin impacto) |
| 8 | 2 | Permiso para `SELECT … FOR UPDATE` sobre `pagos` (PostgreSQL exige privilegio `UPDATE` en al menos una columna; ver ADR 0006). | caja | No |
| 9 | 7 | CHECK `mesas_codigo_qr_formato`: `codigo_qr ~ '^[0-9A-HJKMNP-TV-Z]{8}$'`; achicar `codigo_qr` a `VarChar(8)`. El único global (`mesas_codigo_qr_key`) ya existe. | salon | ⚠️ Cambia el tipo de una columna (sin datos: sin impacto) |
| 10 | 7 | Límite de consultas por IP en `/m/:codigo`: no es de la base; va en la API (throttler de NestJS sobre esa ruta). Si la API corre en más de una instancia, el contador tiene que ser compartido. | salon | No |
| 11 | 6 | `CUENTA_PEDIDA` y el único de llamado pendiente por tipo ya existen. La transacción de pedir la cuenta y la vuelta a `ABIERTA` van en los servicios (ADR 0007). Para el llamado `CANCELADO`, ver la fila 14. | salon / pedidos | No |
| 12 | 8 | Motivos codificados. Dos enums nuevos: `CodigoMotivoCierre` (`SE_FUE_SIN_PAGAR`, `ERROR_DE_CARGA`, `MESA_DUPLICADA`, `OTRO`) y `CodigoMotivoAnulacion` (`ERROR_DE_COBRO`, `PRODUCTO_MAL_HECHO`, `CLIENTE_ARREPENTIDO`, `OTRO`). Reemplazar `motivo_anulacion` por `codigo_motivo` + `comentario VarChar(200)` en `pedidos`, `pedido_items` y `pagos` (enum `CodigoMotivoAnulacion`), y agregarlos a `sesiones_mesa` (enum `CodigoMotivoCierre`). En cada tabla, CHECK `<tabla>_comentario_si_otro`: `codigo_motivo IS DISTINCT FROM 'OTRO' OR (comentario IS NOT NULL AND btrim(comentario) <> '')`. Ajustar los CHECKs que hoy usan `motivo_anulacion`: `pedidos_anulacion_completa`, `pedido_items_anulacion_completa` y `pagos_normal_o_contramovimiento` (pago normal sin código ni comentario; contramovimiento con código). En `sesiones_mesa`: CHECK `(motivo_cierre IN ('ABANDONO','MANUAL')) IS NOT DISTINCT FROM (codigo_motivo IS NOT NULL)`, para que el código vaya solo con esos dos motivos (sin cerrar, `motivo_cierre` es nulo y el código también). | pedidos / caja / salon | ⚠️ Quita la columna `motivo_anulacion` de tres tablas y reemplaza tres CHECK (sin datos: sin impacto) |
| 13 | 2 (propina) | Ajustar el CHECK de contramovimiento de `pagos_normal_o_contramovimiento` a `anula_a_id IS NOT NULL AND monto <= 0 AND propina <= 0 AND (monto < 0 OR propina < 0) AND codigo_motivo IS NOT NULL`. El pago normal queda igual (`monto > 0`, `propina >= 0`). | caja | ⚠️ Reemplaza un CHECK (sin datos: sin impacto) |
| 14 | 6 (llamado cancelado) | Agregar `CANCELADO` a `EstadoLlamado` y la columna `cancelado_en timestamptz` nullable. CHECK nuevo `llamados_cancelacion_completa`: `(estado = 'CANCELADO') = (cancelado_en IS NOT NULL)`. `llamados_atencion_completa` no cambia: con `CANCELADO` los dos lados dan falso. El índice único parcial `llamados_uno_pendiente_por_tipo` (`WHERE estado = 'PENDIENTE'`) tampoco cambia: un llamado cancelado sale del índice, así que se puede crear uno nuevo. *Propuesta:* CHECK `estado <> 'CANCELADO' OR tipo = 'CUENTA'`, porque la regla solo cancela llamados de cuenta. | salon | No (agrega un valor de enum y una columna) |
| 15 | 7 (token) | Tabla nueva `dispositivos_sesion` (salon), editable: `id`, `local_id`, `sesion_id`, `token_hash` (SHA-256 de un token aleatorio de al menos 256 bits, generado con un generador criptográfico; el token en claro se le da una sola vez al dispositivo y **nunca se guarda**), `creado_en`, `expulsado_en`, `expulsado_por_id`. Único global `token_hash` (la búsqueda por token, como la del QR, no lleva el local). FK `(local_id, sesion_id) → sesiones_mesa` y `(local_id, expulsado_por_id) → usuarios`. CHECK `(expulsado_en IS NULL) = (expulsado_por_id IS NULL)`. Único `(local_id, sesion_id, id)` para la FK de pedidos. El token es válido si existe, no está expulsado y su sesión no está `CERRADA`; el `local_id` y la sesión de cada request del cliente salen de ahí. "Expulsar dispositivo": endpoint para mozo o encargado. *Propuesta:* `pedidos.dispositivo_id` con FK `(local_id, sesion_id, dispositivo_id) → dispositivos_sesion(local_id, sesion_id, id)` y CHECK `origen <> 'QR' OR dispositivo_id IS NOT NULL` (la regla 4 dice que los pedidos por QR se identifican por dispositivo). Al expulsar, tiempo-real desconecta ese dispositivo de la sala de la mesa. | salon / pedidos | No (tabla y columna nuevas) |
| 16 | 2 (sesión cerrada) | Pago de corrección en una sesión cerrada: **el modelo actual ya lo permite**, porque la FK `pagos → sesiones_mesa` no mira el estado y es un pago normal. Va en el servicio de caja: un caso de uso propio (el cobro común exige sesión abierta), con los permisos de la regla 2 (encargado; dueño si el turno del pago original cerró). *Propuesta:* contramovimiento y pago de corrección en una sola transacción. Si los reportes tienen que distinguir un pago de corrección de un cobro común, haría falta una referencia; hoy la regla no lo pide y no se agrega. | caja | No |
| 17 | 7 (QR desconocido) | Nada en la base (no se guardan códigos viejos). La API busca `codigo_qr` normalizado **y** `activa = true`; si no hay, responde `404 QR_NO_VALIDO` (*propuesta* para `docs/api/contratos.md`, tarea de salón), con el mismo cuerpo en los tres casos. | salon | No |

## Decisiones tomadas

- Se paga en la caja.
- No hay cocina aparte: todo sale de la barra.
- El dueño define el valor del punto, los productos canjeables y su costo en puntos.
- Los puntos sirven en un solo café por ahora; agregar sucursales luego implica agrupar locales bajo una marca.
- Sin menú 3D ni asistente con IA.
