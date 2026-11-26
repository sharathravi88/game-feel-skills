---
name: car-speed-feel
description: Make a car or vehicle feel faster without changing its actual speed - FOV kick with speed and boost, speed lines at the screen edges, speed-based camera shake, engine pitch that follows the revs and gear shifts, and camera lag on acceleration. Use when the user says their racing or driving game feels slow, floaty or has no sense of speed.
---

# Car speed feel

Players judge speed by what they see and hear, not by the number. These five tricks make the same speed feel much faster. They only touch the camera and the sound; the car's physics stay exactly the same.

(Camera *position* matters too: a lower, closer chase camera makes nearby kerbs and objects sweep past faster. Get that right first; this skill is everything else.)

## Numbers that work (from the demo)

"Top speed" below = 270 km/h (75 m/s); `k = speed / topSpeed`, clamped to 0–1.

| Trick | Values |
|---|---|
| FOV kick | + **12°** vertical FOV at top speed (`12 · k^1.4`), + **7°** extra while boosting; ease toward it at **3.5 /s** |
| Speed lines | Appear from **120 km/h**, full at **230 km/h**; max opacity **0.32** (+0.08 on boost); only in the outer **60–95 %** of the screen, left and right edges |
| Speed shake | Smooth noise, **0.045 m · k²** position + **0.006 rad · k²** roll; boost kick **0.07 m** that fades over ~0.6 s |
| Engine pitch | Pitch from the revs (idle 1,100 → redline 7,400 rpm, two firing pulses per revolution); revs drop on each upshift; ~**0.22 s** power cut per shift |
| Camera lag | Camera falls back **0.11 m per m/s²** of acceleration (clamp −0.9 to +1.8 m), easing at **3 /s** |

## Code

```js
const damp = (rate, dt) => 1 - Math.exp(-rate * dt);
const clamp = (x, a, b) => Math.max(a, Math.min(b, x));

function updateSpeedFeel(cam, car, dt) {
  const k = clamp(car.speed / TOP_SPEED, 0, 1);

  // FOV KICK
  const wantFov = BASE_FOV + 12 * Math.pow(k, 1.4) + (car.boosting ? 7 : 0);
  cam.fov += (wantFov - cam.fov) * damp(3.5, dt);

  // CAMERA LAG: positive acceleration pushes the camera back, braking pulls it in.
  cam.lag += (clamp(car.accel * 0.11, -0.9, 1.8) - cam.lag) * damp(3, dt);
  const followDist = CHASE_DIST + cam.lag;

  // SPEED SHAKE (apply after positioning the chase camera, as an offset)
  cam.shakeTime += dt;
  const amp = 0.045 * k * k + 0.07 * cam.boostKick * cam.boostKick;  // boostKick: 1 at boost start, fades at 1.6/s
  const n = (a, b) => Math.sin(cam.shakeTime * a + b) * 0.6 + Math.sin(cam.shakeTime * a * 1.7 + b * 2.1) * 0.4;
  cam.offset.set(amp * n(37, 1.1), amp * 0.8 * n(41, 4.3), 0);
  cam.roll = (0.006 * k * k) * n(29, 7.7);

  // SPEED LINES: opacity from speed; spawn each streak 60–95 % toward the left/right screen edge,
  // a little in front of the camera, and move it toward the camera so it slides outward.
  speedLines.opacity = clamp((car.speed - 120 / 3.6) / ((230 - 120) / 3.6), 0, 1) * 0.32 + (car.boosting ? 0.08 : 0);

  // ENGINE PITCH (Web Audio / your engine's audio): frequency follows the revs.
  engine.frequency = (car.rpm / 60) * 2;              // two firing pulses per revolution
  engine.filterCutoff = 350 + engine.frequency * (3 + 5 * car.throttle);
}

// Gears: rpm = idle + (redline - idle) * speed / gearTopSpeed. Upshift near the redline,
// cut the power for ~0.22 s so the revs audibly drop, and the camera lag "breathes" with each shift.
```

## Common mistakes

- **FOV that snaps.** Always ease it, or the screen pumps on every speed change.
- **FOV too wide** (over ~90° vertical): straight lines bend and the car looks tiny. Keep the kick to ~10–15°.
- **Speed lines over the car or the road ahead.** Keep them at the edges and faint; they should be noticed only when they're gone.
- **Random shake every frame.** Use smooth noise, scale it with speed², and keep it tiny.
- **One looping engine sample at a fixed pitch.** It sounds like a fan. Pitch must follow the revs, with drops on gear shifts.
- **Camera lag that's too strong or too slow,** so the car drifts out of frame. Clamp it and let it settle within about a second.
- **Changing the car's real speed to "feel" faster.** Then it isn't feel any more; it changes the game. Leave the physics alone.

## Check it works

- [ ] Same speed, tricks on vs. off: on feels clearly faster.
- [ ] Boost gives a quick FOV punch and a small kick, then settles.
- [ ] Speed lines only appear at speed, only at the edges, never over the car.
- [ ] You can hear every gear shift with your eyes closed.
- [ ] Accelerating hard: the car pulls slightly away from the camera, then the camera catches up.

Video: Why Your Car Feels Slow: 5 Speed Tricks (Same Speed!) on [@sharathraviofficial](https://www.youtube.com/@sharathraviofficial) (11 Dec 2026)
