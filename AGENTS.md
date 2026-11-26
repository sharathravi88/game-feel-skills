# Game feel skills: instructions for AI coding agents

When the user wants their game to feel better (more responsive, punchier, fairer, smoother) or names one of these tricks,
read the matching `skills/<name>/SKILL.md` **before writing code**, and follow it.

| If the user says something like… | Read |
|---|---|
| hits feel weak, no impact, hitlag, freeze frames | `skills/hit-stop/SKILL.md` |
| more punch, explosions, camera shake, shake looks jittery | `skills/screen-shake/SKILL.md` |
| jumps don't register, fell off the ledge, platformer feels unfair | `skills/coyote-time/SKILL.md` |
| jump ignored when pressed just before landing | `skills/jump-buffer/SKILL.md` |
| camera jitters, follows too tightly, makes people dizzy | `skills/camera-dead-zone/SKILL.md` |
| motion looks robotic, stiff, linear; tweens, easing functions | `skills/easing/SKILL.md` |
| publish / upload / playtest a web game on Wavedash | `skills/publish-to-wavedash/SKILL.md` |

## Rules for every skill

- **Frame-rate independent.** Timers in seconds using `dt`, never frame counts. Smoothing uses `k = 1 - Math.exp(-rate * dt)`, never a fixed `lerp(a, b, 0.1)` per frame.
- **Named constants** at the top of the file, with the skill's default values, so they are easy to tune.
- **Toggleable.** Put each trick behind a boolean or setting so the user can compare on and off.
- **Feel, not rules.** These tricks change how the game feels, not its difficulty or logic. Coyote time and jump buffering make it fairer, nothing else.
- **Fit the codebase.** Use the project's engine, language and structure; the snippets are pseudocode in JavaScript style.
- **Finish with the test list.** After implementing, tell the user how to check it using the skill's "Check it works" section.
