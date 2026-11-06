# 0003. Módulos y capas

Estado: Aceptado
Fecha: 2026-11-05

## Contexto
El sistema crece por etapas (salón, puntos, social, marketing, Mercado Pago) y el club de puntos debe poder compartirse con Kiosco SaaS. La especificación pide que el canal en vivo solo avise y que la verdad esté en la base. Hacen falta límites claros desde la primera tarea para que dependency-cruiser los haga cumplir.

## Decisión
- Se usa la misma estructura que Kiosco SaaS, para poder copiar código entre los dos productos.
- Módulos de la API (NestJS), uno por carpeta en `src/`: **carta**, **salon**, **pedidos**, **caja**, **tiempo-real**; en etapas posteriores **puntos** (2), **social** y **marketing** (3). Además, la infraestructura transversal copiada de Kiosco: `src/config/` (variables de entorno), `src/database/` (Prisma y transacciones), `src/common/` (errores, filtros, validación, dinero, logger) y el módulo `src/health/`.
- Un módulo **no accede a las tablas de otro**: usa el servicio público que el otro exporta o reacciona a un evento de dominio.
- Capas dentro de cada módulo (nombres de Kiosco) y su equivalente para las reglas de `/calidad`:

  | Carpeta o archivo | Capa | Qué contiene |
  | --- | --- | --- |
  | `domain/` | dominio | Entidades y reglas puras. No importa NestJS, Prisma ni nada de infraestructura. |
  | `*.service.ts` | aplicación | Casos de uso; orquesta dominio y repositories (por su clase abstracta), define las transacciones con `TransactionManager`. |
  | `repositories/` | infraestructura | Contrato abstracto (`x.repository.ts`) e implementación Prisma (`prisma-x.repository.ts`), con la conversión Decimal ↔ centavos. Única capa, junto con `src/database/`, que puede importar Prisma (regla de ESLint). |
  | `*.controller.ts` y `dto/` | http | Sin lógica de negocio: validan, toman el `local_id` del contexto y llaman al service. |
- **tiempo-real** publica eventos solo después de confirmar la transacción y nunca es fuente de verdad; al reconectar, cada pantalla pide el estado completo.
- La web (`cafe-qr-web`) se organiza por las mismas áreas (cliente, salón, barra, caja) y consume solo lo documentado en `docs/api/contratos.md`.
- Las reglas de dependencia se codifican en dependency-cruiser (`/calidad`) y cada módulo nuevo se suma a ellas en su tarea.

## Alternativas descartadas
- Organización por tipo técnico (`controllers/`, `services/`, `entities/` globales) — borra los límites entre módulos.
- Microservicios — innecesario para un café; un monolito modular basta y deja separar **puntos** más adelante.
- Publicar eventos dentro de la transacción — un rollback dejaría pantallas mostrando algo que no existe.

## Consecuencias
- **puntos** se puede extraer como paquete compartido con Kiosco SaaS sin arrastrar el resto.
- Hay más archivos por caso de uso; a cambio, el dominio se testea sin base.
- Las consultas que cruzan módulos (por ejemplo, el plano de caja con pedidos y pagos) se arman componiendo servicios, no con joins entre tablas ajenas.
