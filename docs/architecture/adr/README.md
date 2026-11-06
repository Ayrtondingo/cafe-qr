# Decisiones de arquitectura (ADR)

Cada decisión que condiciona trabajo futuro se registra en un archivo `NNNN-titulo-corto.md` (por ejemplo `0001-cuenta-dividida.md`). Las escribe el subagente arquitecto después de que el plan se aprueba.

Un ADR no se edita una vez aceptado: si la decisión cambia, se escribe uno nuevo que lo reemplaza y se marca el anterior como "Reemplazado por NNNN".

## Plantilla

```
# NNNN. Título

Estado: Propuesto | Aceptado | Reemplazado por NNNN
Fecha: AAAA-MM-DD

## Contexto
Qué problema hay que resolver y qué dice la especificación.

## Decisión
Qué se decidió.

## Alternativas descartadas
- Alternativa — por qué no.

## Consecuencias
Qué se vuelve más fácil, qué más difícil y qué queda pendiente.
```
