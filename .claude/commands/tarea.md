---
description: Desarrolla una tarea del sistema del café con el flujo completo (arquitecto → implementación → tester → revisor)
argument-hint: descripción de la tarea o número de issue
---

Tarea: $ARGUMENTS

Seguí este flujo en orden. Vos sos el implementador y coordinás a los subagentes.

## 1. Plan
1. Leé `CLAUDE.md` y `docs/especificacion.md`.
2. Pasale la tarea al subagente **arquitecto** y esperá su plan.
3. Mostrame el plan completo, con las preguntas abiertas y las migraciones marcadas como destructivas.
4. **Pará y esperá mi aprobación.** No escribas código hasta que diga que sí. Si respondo preguntas abiertas, pedile al arquitecto que ajuste el plan y escriba el ADR si corresponde.

## 2. Implementación
1. En cada repo que toca el plan (`cafe-qr-api`, `cafe-qr-web` o ambos), creá la rama `feature/<nombre-corto>` desde `develop` (o desde `main` si `develop` no existe). Mismo nombre de rama en los dos.
2. Si el plan cambia endpoints, DTOs o eventos, actualizá primero `docs/api/contratos.md`.
3. Implementá las tareas del plan de a una, en orden: primero la API, después la web. Si aparece algo que el plan no contemplaba y cambia el diseño, frená y consultame.
3. No agregues funciones fuera del plan ni de la etapa actual.

## 3. Tests
1. Pasale al subagente **tester** la tarea, el plan y la rama.
2. Si reporta fallas por bugs, corregí el código de producción (no los tests) y volvé a llamarlo.
3. Seguí hasta que todo pase. Si después de 3 vueltas sigue fallando, pará y explicame el problema.

## 4. Revisión
1. Corré lint y build.
2. Pasale al subagente **revisor** la tarea, el plan y la rama.
3. Corregí todos los hallazgos BLOQUEANTES e IMPORTANTES. Los MENORES, corregilos si son rápidos; si no, listalos.
4. Volvé a llamar al revisor. Máximo 2 vueltas; si sigue con ❌, pará y explicame.

## 5. Cierre
Mostrame:
- Qué se hizo, en pocas líneas.
- Archivos tocados y migraciones nuevas.
- Resultado de tests y del revisor.
- Hallazgos menores pendientes y reglas de negocio que haya que confirmar.
- Qué conviene probar a mano.
- Mensaje de commit y descripción del PR propuestos para cada repo tocado. Si son dos PR, el de la web menciona al de la API y se mergea después.

No hagas merge ni push sin que te lo pida.
