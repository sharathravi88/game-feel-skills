---
name: screen-shake
description: Add trauma-based screen shake for hits, explosions, landings and shots. Use when the user wants more impact or punch, or says their camera shake looks jittery, random or makes people sick.
---

# Screen shake (trauma-based)

Don't shake by a random offset every frame. Keep a **trauma** value from 0 to 1: events add trauma, it decays over time, and the shake strength is **trauma squared**. Small events barely move the camera, big ones really kick, and it always settles smoothly.

## Numbers that work

| Event | Add trauma |
|---|---|
| Shot fired | 0.06 |
| Landing | 0.12 |
| Hit | 0.22 |
| Kill / explosion | 0.5 |

- Decay: **1.8 per second** (a full 1.0 is gone in ~0.55 s; because of the square, most of the visible shake is over in ~0.2 s).
- Max offset: **1.2** units sideways, **0.8** up/down, **0.06 rad** roll, where the player is about 0.7 units wide. Scale to your world.

## Code

```js
const SHAKE_DECAY = 1.8, MAX_X = 1.2, MAX_Y = 0.8, MAX_ROLL = 0.06;
let trauma = 0, shakeTime = 0;
let shakeSetting = 1; // 0..1 from the options menu (accessibility)

const addTrauma = (amount) => { trauma = Math.min(1, trauma + amount * shakeSetting); };

function updateShake(dt) {
  trauma = Math.max(0, trauma - SHAKE_DECAY * dt);
  shakeTime += dt;
}

// Smooth noise: two sine waves at unrelated speeds (Perlin/simplex noise also works).
const noise = (speed, seed) =>
  Math.sin(shakeTime * speed + seed) * 0.6 + Math.sin(shakeTime * speed * 1.7 + seed * 2.1) * 0.4;

// Call AFTER the camera follow code, as an offset on top of it.
function applyShake(camera, followX, followY) {
  const s = trauma * trauma;
  camera.x = followX + s * MAX_X * noise(43, 1.3);
  camera.y = followY + s * MAX_Y * noise(39, 4.7);
  camera.roll = s * MAX_ROLL * noise(31, 8.1);
}
```

## Common mistakes

- **`Math.random()` every frame.** It looks like a glitch. Use smooth noise.
- **Shaking the world or the UI.** Shake only the camera; the HUD stays still.
- **Feeding the shake into the follow target**, so the camera drifts. Keep it a separate offset.
- **Too strong or too long.** It becomes nausea. Keep offsets small and decay fast.
- **No way to turn it off.** Add a "Screen shake" option (0–100%) that scales the trauma.

## Check it works

- [ ] A shot gives a tiny nudge; a kill or explosion gives a strong kick.
- [ ] The shake settles smoothly, with no snap back to centre.
- [ ] The HUD doesn't move.
- [ ] Setting the option to 0% removes the shake completely.

Video: [Game Juice: 4 Tricks That Make Any Game Feel Amazing](https://www.youtube.com/watch?v=8UrQ52Cw048)
