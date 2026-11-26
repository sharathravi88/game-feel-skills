---
name: camera-dead-zone
description: Add a camera dead zone with smooth, frame-rate-independent follow (and optional look-ahead). Use when the camera jitters, copies every tiny movement, follows too tightly, or makes players dizzy.
---

# Camera dead zone + smooth follow

A camera that copies every small movement makes people dizzy. With a **dead zone**, the camera stays still while the player moves inside a box, and only moves when they push against its edge. Then **smooth follow** eases the camera there instead of snapping.

## Numbers that work

Player ≈ 0.7 units wide and 0.95 tall.

| Setting | Value |
|---|---|
| Dead zone half-size | **1.6** sideways, **1.3** up/down (box ≈ 3.2 × 2.6 units) |
| Smooth follow rate | **4.5 /s** sideways, **3.5 /s** up/down |
| Look-ahead (optional) | **2.6** units in the facing direction, easing at **2.5 /s** |
| Look-ahead turn delay | swing only after facing the new way for **0.35 s** |

## Code

```js
const DZ_X = 1.6, DZ_Y = 1.3;
const FOLLOW_X = 4.5, FOLLOW_Y = 3.5;
const LOOK = 2.6, LOOK_RATE = 2.5;

const clamp = (v, lo, hi) => Math.max(lo, Math.min(hi, v));
const damp = (rate, dt) => 1 - Math.exp(-rate * dt); // frame-rate independent

let focusX = player.x, focusY = player.y, look = 0;

function updateCamera(dt) {
  // DEAD ZONE: the focus point only moves when the player pushes the box edge.
  focusX = clamp(focusX, player.x - DZ_X, player.x + DZ_X);
  focusY = clamp(focusY, player.y - DZ_Y, player.y + DZ_Y);

  // LOOK-AHEAD (optional): show more of where the player is heading.
  const want = player.lookDir * LOOK; // lookDir flips only after 0.35 s facing the new way
  look += (want - look) * damp(LOOK_RATE, dt);

  // SMOOTH FOLLOW: ease toward the goal.
  camera.x += (focusX + look - camera.x) * damp(FOLLOW_X, dt);
  camera.y += (focusY - camera.y) * damp(FOLLOW_Y, dt);
  // Apply screen shake after this, as a separate offset.
}
```

For platformers, also consider a **vertical lock**: keep the camera's height at the last ground level during normal jumps, and only follow big falls.

## Common mistakes

- **`camera.x = lerp(camera.x, target, 0.1)` every frame.** Speed depends on frame rate. Use `damp(rate, dt)`.
- **Dead zone too big.** Near the edge the player can't see what's coming. Combine with look-ahead.
- **Look-ahead that flips instantly** on every tiny turn: add the turn delay and ease it.
- **Following every jump vertically**, which bobs the whole screen.
- **Mixing shake into the follow**, so the camera drifts.

## Check it works

- [ ] Small back-and-forth movement: the camera stays still.
- [ ] Walking steadily: the camera follows smoothly, with no jitter.
- [ ] It feels the same at 30, 60 and 144 fps.
- [ ] With look-ahead, more of the screen is in front of the player, and quick turns don't make it wobble.

Video: Camera Tricks on [@sharathraviofficial](https://www.youtube.com/@sharathraviofficial) (29 Nov 2026)
