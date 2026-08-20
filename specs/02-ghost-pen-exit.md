# SPEC 02 — Salida escalonada de los fantasmas de la pen

> **Status:** aprobado
> **Depends on:** SPEC 01
> **Date:** 2026-08-19
> **Objective:** Implementar la salida escalonada de los 4 fantasmas de la pen con timing clásico de Pac-Man (Blinky primero, otros en intervalos de ~3-4 segundos).

## Scope

**In:**

- Estado de salida para cada fantasma (`inPen: true/false`)
- Timer de liberación escalonada (Blinky: 0s, Pinky: ~3s, Inky: ~6s, Clyde: ~9s)
- Movimiento directo hacia la puerta mientras el fantasma está en la pen
- Cambio de estado `inPen: false` al alcanzar la puerta (fila 12, cols 13-14)

**Out of scope (para futuros specs):**

- Regreso a la pen después de ser comido (deferido a spec futuro)
- Estado frightened/energizer
- Cambio de velocidad por fantasma
- Modo scatter (huir a esquinas)

## Data model

Añadir campo `inPen` al objeto fantasma en `game.ghosts`:

```js
// En createGame(), al crear cada fantasma:
ghosts: GHOST_STARTS.map( ( g ) => ( {
  x: g.x,
  y: g.y,
  dir: 'up',
  speed: GHOST_SPEED,
  kind: g.kind,
  inPen: true,  // ← NUEVO: fantasma dentro de la pen
} ) ),
```

Añadir timer global al game state:

```js
return {
  state: 'start',
  score: 0,
  lives: 3,
  dotsRemaining: dots,
  grid,
  pacman: { /* ... */ },
  ghosts: [ /* ... con inPen */ ],
  ghostReleaseTimer: 0,  // ← NUEVO: frames desde inicio
};
```

Constantes de timing (en frames, asumiendo 60fps):

```js
const GHOST_RELEASE_TIMES = [
  { kind: 'hunter', frames: 0 },    // Blinky: inmediato
  { kind: 'ambush', frames: 180 },  // Pinky: ~3 segundos
  { kind: 'flank', frames: 360 },   // Inky: ~6 segundos
  { kind: 'wander', frames: 540 },  // Clyde: ~9 segundos
];
```

## Implementation plan

1. **maze.js:** No cambia. Los `GHOST_STARTS` y la pen se mantienen igual.

2. **game.js:** Añadir `inPen: true` a cada fantasma en `createGame()` y `ghostReleaseTimer: 0` al game state.

3. **game.js:** Añadir array `GHOST_RELEASE_TIMES` con los tiempos por `kind`.

4. **game.js:** Crear función `updateGhostRelease( game )` que:
   - Incremente `game.ghostReleaseTimer` cada frame
   - Para cada fantasma con `inPen: true`, verifique si ha pasado su tiempo de liberación
   - Si sí, marque `inPen: false`

5. **game.js:** Modificar `decideGhost()` para que:
   - Si `g.inPen === true`, calcule dirección hacia la puerta (celda objetivo: `{ x: 13, y: 12 }` o `{ x: 14, y: 12 }`)
   - Si `g.inPen === false`, use la lógica existente de `ghostTarget()`

6. **game.js:** Modificar `moveGhost()` para que:
   - Si `g.inPen === true` y el fantasma alcanza fila 12 (puerta), cambie `inPen: false`
   - El fantasma puede atravesar la puerta (valor 3) porque `isWall()` ya lo permite para ghosts

7. **game.js:** Llamar a `updateGhostRelease( game )` al inicio de `update()`.

8. **render.js:** No cambia. Los colores ya están definidos por índice.

9. Verificación en navegador:
   - Blinky se mueve inmediatamente
   - Pinky espera ~3 segundos antes de salir
   - Inky espera ~6 segundos
   - Clyde espera ~9 segundos
   - Todos se dirigen directamente a la puerta mientras están en la pen

## Acceptance criteria

- [ ] `MAZE` (pristino) permanece intacto
- [ ] Cada fantasma tiene campo `inPen` inicializado a `true`
- [ ] `ghostReleaseTimer` se incrementa cada frame
- [ ] Blinky (`hunter`) sale inmediatamente (`inPen` se pone en `false` al inicio)
- [ ] Pinky (`ambush`) sale después de ~3 segundos (180 frames)
- [ ] Inky (`flank`) sale después de ~6 segundos (360 frames)
- [ ] Clyde (`wander`) sale después de ~9 segundos (540 frames)
- [ ] Fantasmas en la pen se mueven directamente hacia la puerta (fila 12)
- [ ] Al alcanzar fila 12, el fantasma cambia `inPen` a `false`
- [ ] Fantasmas fuera de la pen usan su lógica de `ghostTarget()` normal
- [ ] La puerta (3) sigue bloqueando solo a Pac-Man, nunca a los fantasmas
- [ ] Ningún error en consola y el resto de la partida funciona igual

## Decisions

- **Sí:** Timer basado en frames (no `Date.now()`). Consistente con el game loop existente y evita drift por rendimiento.
- **Sí:** Blinky sale inmediato. Es el comportamiento clásico; Blinky siempre persigue.
- **Sí:** Movimiento directo a la puerta. Predictable y fiel al original; wander dentro de la pen añadiría complejidad sin beneficio.
- **Sí:** `inPen` como booleano. Simple y suficiente; no se necesita máquina de estados compleja.
- **No:** Regreso a la pen after being eaten. Deferido a spec futuro según decisión del usuario.
- **No:** Modo scatter dentro de la pen. Los fantasmas siempre apuntan a la puerta.

## Risks

| Risk | Mitigation |
| --- | --- |
| Timer no se resetea al reiniciar partida | `resetPositions()` debe poner `inPen: true` y `ghostReleaseTimer: 0` |
| Fantasma atascado en la pen si no encuentra camino | La pen tiene espacio suficiente; el greedy hacia la puerta siempre encuentra salida |

## What is **not** in this spec

- Regreso a la pen después de ser comido.
- Estado frightened/energizer.
- Cambio de velocidad por fantasma.
- Modo scatter (huir a esquinas).
- Ojos de fantasma volviendo a la pen.

Cada uno de esos, si llega, va en su propio spec.
