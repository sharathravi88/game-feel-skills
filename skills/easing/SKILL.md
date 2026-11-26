---
name: easing
description: Replace linear movement with easing curves (ease-out, ease-in-out, overshoot, bounce, stagger) for UI, platforms, pickups and objects. Use when the user says motion looks robotic, stiff, linear, cheap or "like PowerPoint", or asks for tweens or easing functions.
---

# Easing

Linear movement runs at one speed and then stops dead, which looks robotic. Easing controls **how things speed up and slow down**. Each curve is one line of code, and only the curve changes, not the timing.

## Which curve for what

| Situation | Curve | Why |
|---|---|---|
| Something **arrives** (menu, panel, popup slides in) | **Ease-out** (`outCubic`) | Starts fast, lands softly. Feels responsive. |
| Something **leaves** | **Ease-in** (`inCubic`) | Starts slowly, then gets out of the way. |
| Moves **between two points** (platform, door, elevator) | **Ease-in-out** (`inOutCubic`) | Gentle start and stop; riders don't get jolted. |
| Something **pops in** (coin, icon, reward, button) | **Overshoot** (`outBack`) | Goes ~10% past full size, then settles. Feels alive. |
| Something **heavy lands** (crate, rock) | **Bounce**, kept small | A little bounce sells the weight. |
| **Several items** appear together | **Stagger** | One after another, ~0.1 s apart, reads better than all at once. |

## Numbers that work (from the demo)

- Menu slide-in: 0.7 s ease-out; buttons scale in over 0.4 s; slide-out 0.5 s ease-in. The demo is slowed down so it reads on video: for real game UI, **0.2–0.4 s** is usually right.
- Pickup pop-in: 0.45 s `outBack` (≈ 10% overshoot); vanish: 0.3 s `inBack`.
- Crate drop: 0.9 s, bounce heights reduced to about **⅓** of the standard bounce.
- Stagger: **0.1 s** between items (0.12 s for heavy objects, ~0.06 s for exits).

## Code

```js
// u = progress 0..1  →  0..1 (outBack/inBack briefly go outside 0..1: that's the overshoot)
const EASE = {
  linear:     (u) => u,
  outCubic:   (u) => 1 - (1 - u) ** 3,
  inCubic:    (u) => u ** 3,
  inOutCubic: (u) => (u < 0.5 ? 4 * u ** 3 : 1 - (-2 * u + 2) ** 3 / 2),
  outBack:    (u) => 1 + 2.70158 * (u - 1) ** 3 + 1.70158 * (u - 1) ** 2, // ~10% past, then settles
  inBack:     (u) => 2.70158 * u ** 3 - 1.70158 * u ** 2,                   // pulls back, then goes
  outBounce:  (u) => {
    const n = 7.5625, d = 2.75;
    if (u < 1 / d) return n * u * u;
    if (u < 2 / d) return n * (u -= 1.5 / d) * u + 0.75;
    if (u < 2.5 / d) return n * (u -= 2.25 / d) * u + 0.9375;
    return n * (u -= 2.625 / d) * u + 0.984375;
  },
};
// Heavy objects: same timing, bounces only ~1/3 as high.
const softBounce = (u) => 1 - 0.35 * (1 - EASE.outBounce(u)) * (u >= 1 / 2.75 ? 1 : 1 / 0.35);

const clamp = (x, a, b) => Math.max(a, Math.min(b, x));
// Value at time t of a tween that starts at `start` and lasts `duration` seconds.
function tween(t, start, duration, from, to, ease) {
  const u = clamp((t - start) / duration, 0, 1);
  return from + (to - from) * ease(u);
}

// Examples
menu.x = tween(t, 0, 0.3, offscreenX, targetX, EASE.outCubic);
coins.forEach((c, i) => {
  const s = tween(t, i * 0.1, 0.45, 0, 1, EASE.outBack); // STAGGER 0.1 s
  c.scale = Math.max(0, s);                              // never negative
});
```

## Common mistakes

- **Linear for everything.** The most common reason motion looks cheap.
- **Ease-in for things that arrive.** A slow start feels laggy; use ease-out.
- **Overshoot on everything.** Keep it for pop-ins and rewards; big panels and text just look wobbly.
- **Big bounces on light or UI objects.** Bounce is for weight; keep it small.
- **Too slow.** Over ~0.5 s for UI and players start waiting for your menus.
- **Not clamping `u`** to 0..1, or letting overshoot push scale/alpha below 0.
- **Restarting a tween mid-flight from its original start value**, so it jumps. Start the new tween from the current value.

## Check it works

- [ ] Menus arrive quickly and settle softly; nothing stops dead.
- [ ] Moving platforms start and stop gently.
- [ ] Pickups pop slightly past full size, then settle.
- [ ] Groups of items appear one after another.
- [ ] Toggle all easing off (linear) and back on: the eased version feels clearly smoother, at the same timing.

Video: Why Your Game Moves Like a Robot: 5 Easing Tricks on [@sharathraviofficial](https://www.youtube.com/@sharathraviofficial) (3 Dec 2026)
