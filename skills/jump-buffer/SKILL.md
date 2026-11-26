---
name: jump-buffer
description: Add jump buffering so a jump pressed just before landing still happens on landing. Use when players say the game "eats" or ignores jump presses, or jumps fail when pressed slightly early.
---

# Jump buffering

Players often press jump a moment **before** they land. Without buffering, that press is ignored and the jump never happens. Buffering remembers the press for a short time and jumps the instant the player can.

## Numbers that work

- **0.13 s**. Useful range 0.1–0.15 s.
- In the demo, early presses came 0.06–0.1 s before landing: all saved by 0.13 s.

## Code

```js
const JUMP_BUFFER = 0.13; // seconds
let bufferTimer = 0;

function update(dt, jumpPressed, jumpHeld) {
  if (jumpPressed) bufferTimer = JUMP_BUFFER;

  const canJump = grounded || canCoyote(); // see coyote-time
  if (bufferTimer > 0 && canJump) {
    jump();
    bufferTimer = 0; // used up
  }
  bufferTimer = Math.max(0, bufferTimer - dt);

  // Variable jump height still works: if the button is already released
  // when a buffered jump fires, the normal "released early" cut makes it a short hop.
  if (jumpedThisJump && vy > 0 && !jumpHeld && !cutApplied) { vy *= 0.45; cutApplied = true; }
}
```

## Common mistakes

- **Not clearing the buffer after the jump**, so the player jumps again on the next landing.
- **Buffering too long (0.3 s or more).** The character jumps when the player didn't mean to.
- **Only checking for a press on the exact landing frame.** That's the bug this trick fixes.
- **Frame counts instead of seconds.**

## Check it works

- [ ] Press jump just before landing: the character jumps the moment it lands.
- [ ] Press jump 0.5 s before landing: no jump on landing.
- [ ] A quick tap before landing gives a short hop; holding gives a full jump.

Video: [Why Your Jump Feels Bad: 5 Invisible Platformer Tricks](https://www.youtube.com/watch?v=-V2MT1N950k)
