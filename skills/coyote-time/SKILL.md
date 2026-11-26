---
name: coyote-time
description: Add coyote time so the player can still jump for a split second after running off a ledge. Use when players say jumps don't register, "I pressed jump but fell", or the platformer feels unfair.
---

# Coyote time

Players are never frame-perfect. They often press jump a few hundredths of a second **after** their feet left the ledge. Coyote time still allows the jump for a short moment, like the cartoon coyote running off a cliff.

## Numbers that work

- **0.11 s** (≈ 6–7 frames at 60 fps). Useful range 0.08–0.15 s.
- In the demo, "late" presses landed 0.05–0.085 s after leaving the ledge: all of them saved by 0.11 s.

## Code

```js
const COYOTE_TIME = 0.11; // seconds
let grounded = true, jumped = false, coyoteTimer = 0;

// Call when the player walks or falls off an edge WITHOUT jumping.
function onLeftGround() {
  grounded = false;
  jumped = false;
  coyoteTimer = COYOTE_TIME;
}

function update(dt, jumpPressed) {
  if (!grounded) coyoteTimer -= dt;
  const canCoyote = !grounded && !jumped && coyoteTimer > 0;
  if (jumpPressed && (grounded || canCoyote)) jump();
}

function jump() {
  vy = JUMP_VELOCITY;
  grounded = false;
  jumped = true;
  coyoteTimer = 0; // used up
}
```

Combine with [jump-buffer](../jump-buffer/SKILL.md): together they fix most "my jump didn't work" complaints.

## Common mistakes

- **Starting the timer when the player jumps.** That gives a free double jump. Only start it when they leave the ground without jumping.
- **Not clearing the timer after the jump.**
- **Counting frames instead of seconds.** It then feels different at 30, 60 and 144 fps.
- **Too long (over ~0.2 s).** Players notice they are jumping off thin air.

## Check it works

- [ ] Run off a ledge and press jump just after leaving it: the character jumps.
- [ ] Jump normally, then press again in mid-air: no extra jump (unless the game has a double jump).
- [ ] Press jump 0.3 s after leaving the ledge: the character falls.

Video: [Why Your Jump Feels Bad: 5 Invisible Platformer Tricks](https://www.youtube.com/watch?v=-V2MT1N950k)
