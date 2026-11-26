---
name: hit-stop
description: Add hit stop (hitlag, freeze frames) so hits and kills feel heavy. Use when the user says hits feel weak, floaty or have no impact, or asks for hitlag, freeze frames or "the 3-frame trick".
---

# Hit stop

Freeze the gameplay for a split second when something gets hit. The pause lets the player's eye register the impact. Fighting games have done this for decades.

## Numbers that work

| Event | Freeze | ≈ frames at 60 fps |
|---|---|---|
| Normal hit | 0.05 s | 3 |
| Kill / big hit | 0.11 s | 6–7 |

Keep frequent hits under ~0.15 s. Longer freezes only for rare, huge moments (boss kill, finisher).

## Code

```js
const HIT_STOP_HIT = 0.05;   // seconds
const HIT_STOP_KILL = 0.11;
let freeze = 0;

function onHit(target, killed) {
  freeze = Math.max(freeze, killed ? HIT_STOP_KILL : HIT_STOP_HIT); // max, never add
}

function update(dt) {
  updateCameraShake(dt);                     // keeps running during the freeze
  readInput();                               // keep reading input so presses aren't lost
  if (freeze > 0) { freeze -= dt; return; }  // gameplay paused
  updateGameplay(dt);
}
// render() still runs every frame, so the frozen impact stays on screen.
```

Engines with a global time scale (Unity `Time.timeScale`, Godot `Engine.time_scale`): set it to 0 (or ~0.05) and restore it with a timer that uses **unscaled** time.

Pairs well with [screen-shake](../screen-shake/SKILL.md) (shake keeps going while the game is frozen) and a white hit flash on the target.

## Common mistakes

- **Adding freezes together.** Five quick hits become a long stall. Use `max`.
- **Freezing shake and input too.** Shake should keep moving; jump/attack presses during the freeze must not be lost (buffer them).
- **Removing the target before the freeze.** The frozen frame should show the impact: keep a killed enemy visible until the freeze ends.
- **Using it everywhere.** Only on hits, not on shots fired, footsteps or pickups.
- **Online games:** don't pause the shared simulation; freeze only local visuals and animation.

## Check it works

- [ ] A hit shows a short, visible pause on the impact frame.
- [ ] Kills pause noticeably longer than normal hits.
- [ ] Rapid hits don't stack into a long stall.
- [ ] A button pressed during the freeze still does its action afterwards.
- [ ] Toggle it off and on: hits feel heavier with it, and the game doesn't feel laggy.

Video: [The 3-Frame Trick That Makes Hits Feel 10x Stronger](https://www.youtube.com/shorts/BDUfbgo7ZSM)
