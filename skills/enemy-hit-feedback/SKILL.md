---
name: enemy-hit-feedback
description: Make enemies visibly react when hit - white flash, knockback with a lean, damage numbers, a lagging "ghost" health bar, and a death burst with slow motion. Use when the user says hits don't register, enemies feel like they're made of air, combat feels unresponsive, or asks for damage numbers or health bars.
---

# Enemy hit feedback

Players can't feel your game; they can only see and hear it. If an enemy just loses HP silently, hits feel like hitting air. These five tricks make every hit **show**. Use them together; each one is small.

## Numbers that work (from the demo)

| Trick | Values |
|---|---|
| White flash | **0.09 s** of white (~5 frames at 60 fps) on every hit |
| Knockback | **6 units/s** push that fades with `exp(-9·t)` (≈ 0.7 units total, ×1.5 on crits); lean back **0.32 rad**, fading with `exp(-10·t)` |
| Damage numbers | Live **0.8 s**: pop in with overshoot (~0.2 s), float up, fade. Size `0.75 + damage/40`; crits **1.35×** and orange with "!" |
| Lagging health bar | Real bar drops instantly; pale ghost waits **0.3 s**, then drains at **50 % of max HP per second** |
| Death burst | ~25 pieces in the enemy's color (+ some white) and **slow motion at 0.25× for 0.35 s** |

Enemy in the demo: 60 HP, hits of 9–14 damage, crits double, so 4–6 hits per enemy. Long enough to see every trick.

## Code

```js
const FLASH_T = 0.09;
const KNOCK_V = 6, KNOCK_DECAY = 9, HURT_LEAN = 0.32;
const GHOST_DELAY = 0.3, GHOST_RATE = 0.5;     // share of max HP per second
const SLOWMO_T = 0.35, SLOWMO_SCALE = 0.25;

function onHit(enemy, damage, crit) {
  // Lagging bar: only start the wait if the ghost has caught up, otherwise it never drains during combos.
  if (enemy.ghost <= enemy.hp) enemy.ghostWait = GHOST_DELAY;
  enemy.hp = Math.max(0, enemy.hp - damage);

  enemy.flash = FLASH_T;                                   // WHITE FLASH
  if (enemy.hp > 0) {                                      // KNOCKBACK (not on the killing hit)
    enemy.knock = KNOCK_V * (crit ? 1.5 : 1);
    enemy.lean = HURT_LEAN;
  }
  spawnDamageNumber(enemy, damage, crit);                  // DAMAGE NUMBER
  if (enemy.hp === 0) deathBurst(enemy);                   // DEATH BURST
}

function updateEnemy(enemy, dt) {
  enemy.x += (enemy.knock - enemy.walkSpeed) * dt;         // pushed back against its walk
  enemy.knock *= Math.exp(-KNOCK_DECAY * dt);
  enemy.lean *= Math.exp(-10 * dt);                        // rotate the body back by `lean` (pivot at the feet)
  enemy.flash -= dt;                                       // render white while flash > 0
  if (enemy.ghostWait > 0) enemy.ghostWait -= dt;
  else enemy.ghost = Math.max(enemy.hp, enemy.ghost - GHOST_RATE * enemy.maxHp * dt);
}

function deathBurst(enemy) {
  spawnPieces(enemy, 25);
  slowMotion = SLOWMO_T;                                   // in the game loop: if (slowMotion > 0) { slowMotion -= dt; dt *= SLOWMO_SCALE; }
}
```

**White flash:** the simplest version is a white emissive color (or a "flash" uniform in the shader) switched on while `flash > 0`. Don't swap materials or textures every frame.

**Health bar layers,** back to front: dark background, pale ghost (width = `ghost / maxHp`), red fill (width = `hp / maxHp`). Draw the bar as UI on top of everything, so debris never covers it.

**Damage numbers** are text: draw them with your UI/text system, positioned by projecting the enemy's head to screen space each frame.

## Common mistakes

- **Flash too long (over ~0.15 s)** or tinted red. Short and white reads best.
- **Knockback on the killing hit,** so the death burst happens somewhere else. Skip it when the enemy dies.
- **Knockback that slides forever.** Use a strong decay; it should settle in ~0.3 s.
- **Ghost bar that resets its delay on every hit,** so during fast combos it never drains. Start the delay only when it has caught up.
- **Damage numbers that stack on the same spot.** Add a small random sideways offset and let them float up.
- **Deleting the enemy on death.** Even a cheap burst plus a moment of slow-mo makes kills feel earned. Keep slow-mo short; long slow-mo on every kill gets annoying.
- **Rolling damage differently when feedback is on or off.** Feedback must never change the fight itself.

## Check it works

- [ ] Every hit is visible even with the sound off.
- [ ] Enemies get pushed back and lean, then recover within ~0.3 s.
- [ ] Numbers pop, rise and fade; crits look clearly different.
- [ ] After each hit you can see the pale chunk drain away on the health bar.
- [ ] Kills burst apart with a brief slow-mo, not a disappearing enemy.
- [ ] Toggle all of it off: the same fight feels like hitting air.

Video: Your Enemies Don't Feel Hit: 5 Tricks That Fix It on [@sharathraviofficial](https://www.youtube.com/@sharathraviofficial) (7 Dec 2026)
