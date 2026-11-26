# Game Feel Skills

Instructions that teach your **AI coding assistant** how to add game feel properly: hit stop, screen shake, coyote time, jump buffering, camera dead zones, and more.

Every trick comes from a video on [@sharathraviofficial](https://www.youtube.com/@sharathraviofficial) and uses the **same values tuned in the demo you see in the video**. Your AI gets working numbers, frame-rate-independent code, the common mistakes and a test checklist instead of guessing.

## How to use it

| Tool | Do this |
|---|---|
| **Claude Code** | Copy the folders in `skills/` into `.claude/skills/` in your game repo (or `~/.claude/skills/` for all projects). Claude picks a skill when your request matches its description. |
| **Cursor, Copilot, Codex and other agents** | Copy `AGENTS.md` and `skills/` into the root of your game repo. |
| **Any chat AI** | Paste the `SKILL.md` of the trick you want, then ask: *"Add this to my game."* |

Then just describe the problem: *"Players say they pressed jump but fell off the ledge"* → the AI reads `coyote-time`.

## Skills

| Skill | Fixes | Video |
|---|---|---|
| [hit-stop](skills/hit-stop/SKILL.md) | Hits feel weak, no impact | [The 3-Frame Trick](https://www.youtube.com/shorts/BDUfbgo7ZSM) |
| [screen-shake](skills/screen-shake/SKILL.md) | No punch, or shake looks jittery | [Game Juice: 4 Tricks](https://www.youtube.com/watch?v=8UrQ52Cw048) |
| [coyote-time](skills/coyote-time/SKILL.md) | "I pressed jump but fell" | [Why Your Jump Feels Bad](https://www.youtube.com/watch?v=-V2MT1N950k) |
| [jump-buffer](skills/jump-buffer/SKILL.md) | Jump presses get ignored just before landing | [Why Your Jump Feels Bad](https://www.youtube.com/watch?v=-V2MT1N950k) |
| [camera-dead-zone](skills/camera-dead-zone/SKILL.md) | Camera is jittery or makes players dizzy | Camera Tricks (29 Nov) |
| [publish-to-wavedash](skills/publish-to-wavedash/SKILL.md) | Getting a web game live on Wavedash | |

New skills are added with each video.

## How each skill is written

1. **When to use it**: the symptoms a player or developer describes.
2. **Numbers that work**: the tuned defaults from the demo.
3. **Code**: engine-agnostic and frame-rate independent (seconds, not frames).
4. **Common mistakes** the AI should avoid.
5. **Check it works**: a short test list for you to try.

## License

MIT. Use it in any project, commercial or not.

If this helps your game, please subscribe to [@sharathraviofficial](https://www.youtube.com/@sharathraviofficial). It means a lot to me.
