# SPEC 01 — Cuatro fantasmas con personalidades clásicas

> **Status:** Aprobado
> **Depends on:** —
> **Date:** 2026-08-09
> **Objective:** Sustituir los 2 fantasmas actuales por 4 con personalidades clásicas de Pac-Man (Blinky, Pinky, Inky, Clyde), cada una con una lógica propia, siendo Blinky perseguidor agresivo.

## Scope

**In:**

- Cuatro fantasmas, cada uno con un valor distinto en `kind`, todos arrancando dentro de la pen.
- Cuatro lógicas de decisión greedy en `decideGhost()` (game.js:113): agresiva, emboscada, cooperativa y errante.
- Colores por índice en `render.js` (ya hay 4 en `GHOST_COLORS`); sin cambios ahí.

**Out of scope (para futuros specs):**

- Estados scatter/frightened, comerse a los fantasmas con poder (energizer), ojos volviendo a la pen.
- Salida escalonada de la pen (tiempos de liberación distintos).
- Cambio de velocidad por fantasma.
- BFS/A*: la selección de giro sigue siendo greedy.

## Data model

Extender `GHOST_STARTS` en `maze.js` de 2 a 4 entradas, una por `kind`. Los 4 quedan en el interior de la pen (filas 13-15, columnas 11-16, pasables según `MAZE_STR`):

```js
const GHOST_STARTS = [
  { x: 13, y: 14, kind: 'hunter'  }, // Blinky (rojo) agresivo
  { x: 14, y: 14, kind: 'ambush'  }, // Pinky (rosa) emboscada
  { x: 13, y: 15, kind: 'flank'   }, // Inky (cian) cooperativo
  { x: 14, y: 15, kind: 'wander'  }, // Clyde (naranja) errante
];
```

`game.ghosts` ya propaga `kind` (game.js:39-45); no cambia su forma.

Objetivos según `kind` (celdas a las que apunta el greedy):

- `hunter` → celda actual de Pac-Man.
- `ambush` → celda 4 pasos por delante de la posición de Pac-Man en su `dir`.
- `flank` → del vector "celda delante de Pac-Man − posición de Blinky", duplicado: requiere localizar a Blinky en `game.ghosts`.
- `wander` → si la distancia Manhattan a Pac-Man > 8, apunta a su celda; si es ≤ 8, apunta a la esquina `(0, 30)` (escapa).
- `random` desaparece: pasa a ser `wander`.

## Implementation plan

1. `maze.js`: ampliar `GHOST_STARTS` a las 4 entradas anteriores. Verificación visual: al abrir la página se ven 4 fantasmas en la pen.
2. `game.js`: refactorizar `decideGhost()` para calcular objetivos por `kind` (función `ghostTarget( game, g )`) manteniendo la selección greedy de direcciones existente. `hunter` y `wander` usan el código actual; añadir `ambush` y `flank`.
3. `game.js`: mover el `Math.round( p.x/p.y )` y la lectura de `dir` de Pac-Man fuera del cuerpo para que `nextDir` de Pac-Man no afecte al objetivo (objetivo usa `dir` actual). Revisar que Inky encuentre a Blinky por `kind` y no por índice.
4. Verificación final en navegador (hard-refresh) de los 4 comportamientos distintos.

## Acceptance criteria

- [ ] `MAZE` (pristino) permanece intacto; `GHOST_STARTS` tiene exactamente 4 entradas con los `kind` `hunter`, `ambush`, `flank`, `wander`.
- [ ] La página carga con 4 fantasmas en la pen, cada uno con un color distinto (rojo, rosa, cian, naranja).
- [ ] `hunter` reduce continuamente su distancia Manhattan exacta a Pac-Man (persigue su celda actual).
- [ ] `ambush` tiende a la celda 4 pasos por delante de la dirección de Pac-Man, no a su celda.
- [ ] `flank` depende de la posición de Blinky: su objetivo cambia cuando Blinky se mueve (no es fijo ni independiente).
- [ ] `wander` se aleja hacia la esquina cuando Pac-Man está a ≤8 celdas.
- [ ] Ningún error en consola y el resto de la partida (dots, vidas, score) funciona igual.
- [ ] La puerta (3) sigue bloqueando solo a Pac-Man, nunca a los fantasmas.

## Decisions

- **Sí:** personajes clásicos (Blinky/Pinky/Inky/Clyde) sobre un conjunto a medida. Es la referencia que el usuario quiere imitar y cada uno justifica su `kind`.
- **Sí:** greedy hacia celda objetivo. Se integra con `decideGhost()` actual; BFS se descarta por ser reescritura completa.
- **Sí:** ampliar `kind` (sin renombrar a `blinky/…`). Menos renombrado y el comportamiento queda en `decideGhost()` vía `switch`.
- **Sí:** colores por índice. `GHOST_COLORS` ya son 4; vincular color a `kind` no aporta nada hoy.
- **Sí:** `hunter` apunta a la celda actual de Pac-Man (es la práctica original de Blinky).
- **Sí:** `random` deja de existir y Clyde es `wander` con umbral de 8 celdas.
- **No:** liberación escalonada de la pen, energizers/caza-fantasmas, ojos de retorno: quedan para futuros specs.

## Risks

| Risk | Mitigation |
| --- | --- |
| `flank` necesita la posición de Blinky; si depende del índice es frágil | Localizar a Blinky por `kind` en `game.ghosts` |
| Objetivos de `ambush`/`flank` pueden caer fuera del laberinto (celda no existente) | El greedy compara distancias Manhattan, válidas aunque el objetivo quede fuera |

## What is **not** in this spec

- Energizers, fantasmas comibles, estado frightened y ojos volviendo a la pen.
- Tiempos de salida distintos por fantasma.
- Velocidades individuales.
- Rutas calculadas (BFS/A*).

Cada uno de esos, si llega, va en su propio spec.