---
name: publish-to-wavedash
description: Prepare, test and upload an HTML5/web game build to Wavedash (wavedash.com): wavedash.toml, the required Wavedash.init() call, load progress, local testing with wavedash dev, and uploading a build. Use when the user wants to publish or playtest a web game on Wavedash.
---

# Publish a web game to Wavedash

Based on the official docs ([Quickstart](https://docs.wavedash.com/getting-started/quickstart), [SDK setup](https://docs.wavedash.com/sdk/setup)) and CLI 0.1.99, checked 26 Nov 2026. Check the docs for changes before relying on details.

## 1. CLI (optional: the Developer Portal can do everything too)

```bash
curl -fsSL https://wavedash.com/cli/install.sh | sh
wavedash --version
wavedash auth login      # opens the browser, once per machine
```

## 2. Configure the project

From the game's root folder run `wavedash init`. It writes `wavedash.toml`:

```toml
game_id = "your-game-id"
upload_dir = "dist"          # the folder that contains index.html
entrypoint = "index.html"
```

Godot and Unity projects get a `[godot]` / `[unity]` section instead of `entrypoint`, and `upload_dir` defaults to `build`.

**Demo builds:** keep a second config, e.g. `wavedash.demo.toml` with `upload_dir = "dist-demo"`, and push it with `-c wavedash.demo.toml`.

## 3. Call `Wavedash.init()`: required

Until `init()` is called, the game stays hidden behind the Wavedash loading screen. The SDK is injected by Wavedash; nothing needs installing. Report load progress so players see a real loading bar:

```js
const wd = globalThis.Wavedash; // injected on Wavedash, undefined on localhost

wd?.updateLoadProgressZeroToOne(0);
await loadAssets();
wd?.updateLoadProgressZeroToOne(0.8);
await startGame();
wd?.updateLoadProgressZeroToOne(1);
wd?.init({ debug: false }); // once, when the player can interact
```

The `?.` keeps the game working on localhost without the SDK. For TypeScript types: `npm install @wvdsh/sdk-js`.

## 4. Build and test locally

- Use **relative asset paths** (Vite: `base: "./"`), so the build works from any URL.
- Prefer bundling fonts and assets over runtime CDN requests.
- Build (`npm run build`), then run `wavedash dev`: it serves the build signed in to your account with the SDK injected, against sandbox data.
- Using SharedArrayBuffer or threads? Read [Cross-origin isolation](https://docs.wavedash.com/platform/cross-origin-isolation).

## 5. Upload and publish

```bash
wavedash build push -m "What changed in this build"
```

Or in the [Developer Portal](https://wavedash.com/dev-portal): **Builds → Upload new build** (folder or zip, up to 1 GB uncompressed). An uploaded build goes live only when you click **Publish**. Then open the public game URL in a fresh browser, and fill in the store page (title, description, cover art, preview video, tags).

## Common mistakes

- **No `init()` call:** the game never appears.
- **`upload_dir` pointing at the project root:** this uploads source code, `.env` files and more. Point it at the build output only.
- **Absolute asset paths** (`/assets/...`) that break on the hosted URL.
- **Credentials in the repo.** `wavedash.toml` holds only the game ID; the CLI keeps your login on your machine.

## Check it works

- [ ] `wavedash dev` shows the game after the loading bar, not a stuck loading screen.
- [ ] The loading bar moves smoothly instead of jumping from 0% to 100%.
- [ ] The game still runs on localhost without the SDK.
- [ ] The uploaded build contains only the build output.
