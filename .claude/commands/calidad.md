---
description: Configura las reglas de lint de arquitectura y el CI de GitHub Actions en cafe-qr-api y cafe-qr-web
argument-hint: (opcional) "api", "web" o aclaraciones
---

Configurá las reglas automáticas de calidad en los repos de código. Aclaraciones: $ARGUMENTS

Hay dos repos separados, cada uno con su propio git: `cafe-qr-api` (NestJS) y `cafe-qr-web` (React). Cada uno lleva su propia configuración y su propio CI. Si el argumento es "api" o "web", trabajá solo en ese repo.

Este comando se corre cuando ya existe la estructura del repo (con al menos un módulo). Si todavía no existe, decímelo y pará.

## 1. Relevamiento (sin tocar nada)

1. Leé `CLAUDE.md` y la lista de módulos del subagente `arquitecto` (`.claude/agents/arquitecto.md`).
2. En cada repo, mirá: gestor de paquetes (yarn, npm, pnpm), carpetas de cada módulo y capa, scripts existentes en `package.json`, versión de Node y, en la API, cómo se levanta PostgreSQL para tests.
3. Mostrame:
   - La estructura detectada en cada repo.
   - La tabla de qué capa/módulo puede importar a cuál (ver punto 2).
   - Los scripts que vas a agregar y el workflow que vas a crear en cada repo.
4. **Pará y esperá mi aprobación.**

## 2. Reglas de arquitectura

Usá **dependency-cruiser** con un `.dependency-cruiser.cjs` en cada repo y un script `lint:arq` que lo corra.

**En `cafe-qr-api`** (adaptá las rutas a la estructura real):

- **Capas:** `domain` no importa nada de `application`, `infrastructure` ni de librerías de infraestructura (`@nestjs/*`, el ORM, `pg`, `socket.io`, `axios`). `application` no importa `infrastructure`. Los controllers no importan repositorios ni el ORM directamente.
- **Módulos:** un módulo solo importa de otro a través de su API pública (por ejemplo su `index.ts` o sus servicios exportados), nunca entidades, repositorios ni archivos internos de otro módulo.
- **Etapas:** `pedidos`, `salon`, `carta` y `caja` no dependen de `social` ni de `marketing`. `puntos` no depende de `social` ni de `marketing`.
- **Generales:** sin dependencias circulares, sin importar archivos de test desde código de producción, sin módulos huérfanos.
- Regla de ESLint que prohíba `parseFloat` y `toFixed` en los módulos `caja` y `puntos` (montos y puntos van en enteros).

**En `cafe-qr-web`:**

- Las pantallas de cliente, salón, barra y caja no se importan entre sí; comparten solo lo que está en una carpeta común (componentes, cliente de API, tipos).
- Los componentes no llaman a `fetch` ni a la librería HTTP directamente: pasan por el cliente de API común.
- Sin dependencias circulares.
- Regla de ESLint que prohíba `parseFloat` y `toFixed` sobre montos fuera del helper de formato de moneda.

**Probá que las reglas funcionan:** en cada repo, creá temporalmente una violación de cada tipo (por ejemplo, un import del ORM dentro de `domain`), confirmá que `lint:arq` falla con un mensaje claro y borrá la violación. Mostrame la salida de cada prueba.

## 3. CI con GitHub Actions

Un `.github/workflows/ci.yml` **en cada repo**, que corra en cada pull request a `develop` y `main` y en cada push a esas ramas.

**`cafe-qr-api`:**
1. Instalar dependencias con el gestor del proyecto y caché.
2. `lint` (ESLint) y `lint:arq` (dependency-cruiser).
3. Chequeo de tipos (`tsc --noEmit`).
4. Tests unitarios.
5. Tests de integración contra un servicio `postgres` del workflow, con las migraciones aplicadas antes.
6. `build`.

**`cafe-qr-web`:**
1. Instalar dependencias con caché.
2. `lint` y `lint:arq`.
3. Chequeo de tipos.
4. Tests.
5. `build`.

El workflow tiene que fallar si falla cualquier paso. Usá la misma versión de Node que el proyecto. Poné al job un nombre fijo (`ci`) para que sea fácil elegirlo al proteger las ramas.

Creá también `.github/pull_request_template.md` en cada repo con: qué cambia, etapa, PR relacionado en el otro repo, migraciones (y si alguna es destructiva, solo en la API), cómo se probó y checklist (tests, lint, revisor).

## 4. Cierre

- Corré localmente todos los scripts nuevos en cada repo y confirmá que pasan.
- En el `CLAUDE.md` de cada repo de código (crealo si no existe), agregá: "Antes de terminar una tarea corré `lint`, `lint:arq`, tests y build. Si agregás un módulo nuevo, sumalo a las reglas de `.dependency-cruiser.cjs`."
- Mostrame los archivos creados en cada repo, la salida de las pruebas de violación y el paso manual que me queda en GitHub (proteger `main` y `develop` en los dos repos exigiendo que el check `ci` pase).
- Los commits van dentro de cada repo, en la rama de la tarea. No hagas push sin que te lo pida.
