# BetterHitreg for Levi Launcher (Minecraft Bedrock, Android)

A native `.so` mod for [Levi Launcher](https://levilaunchroid.levimc.org/), based on the idea of the Java mod
**BetterHitreg** by Jass: *register hits before the server does so hit registration feels faster.*

> **Status: source project only; Android build and in-game behavior are not verified.**
> The package format, config and build are set up as the Levi Launcher docs describe. The game hooks need
> **signatures for your exact Minecraft version, which are not included.** Without them the mod loads, logs
> why, and does nothing (it never touches the game). No compiled `libBetterHitreg.so` is included either;
> you build it with the steps below.

## What you get when it is built

```
BetterHitreg/
├── libBetterHitreg.so
├── manifest.json
└── config/
    ├── config.json
    └── config.schema.json
```

zipped as `BetterHitreg-<version>-arm64-v8a.levipack` (the folder name is the runtime mod id, keep it stable).

## What it does

When you attack a target, the mod plays the game's own hurt flash on it immediately, then swallows the
server's duplicate flash a moment later. Optional log alerts report late confirmations and ghost hits
(predicted, never confirmed). Hits are never changed or sent differently: it only adds local visuals.

## What it is not

* **Not a conversion of the .jar.** The Java mod is built on Java Edition internals (Mixins). Bedrock is a
  different program, so this is a fresh implementation of the core idea, written against the Levi Launcher SDK.
* **Not ported:** sound muffling/sharpening, 1.8 sounds, hitbox/reach/jump overlays, jump-reset colors, void
  world / custom ground, the `/hitreg` commands and in-game UI. Hit sounds and crit particles are not predicted.

## Build the .levipack (no PC needed)

1. Create a GitHub repository and upload all files from this folder. Keep `.github/workflows/build.yml`
   (if the upload skips the hidden `.github` folder, use *Add file → Create new file* and paste it in).
2. Open the **Actions** tab → **Build BetterHitreg.levipack** → **Run workflow**.
3. When it finishes, download the `BetterHitreg-levipack` artifact. GitHub gives you a zip; the
   `.levipack` is inside it.
4. In Levi Launcher, import the `.levipack` in the mod manager and enable it.

On a computer with the Android NDK (28.2.13676358 or compatible), CMake and Ninja:

```
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake \
  -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-28 -DANDROID_STL=c++_shared
cmake --build build --target levi_package
```

Already have a `libBetterHitreg.so` from somewhere else? `python3 scripts/make_levipack.py path/to/libBetterHitreg.so`

## Fill in the signatures

The mod needs two functions from `libminecraftpe.so`. Put their symbol or byte pattern in `config.json`
(`hooks`), using the launcher's config editor:

| Setting | Function it must point to |
| --- | --- |
| `attackSignature` | called when the local player attacks an entity; the target entity must be argument `attackTargetArg` (0 = `this`, 1 = first parameter) |
| `entityEventSignature` | receives entity events as (`this`, event id, data) |

`hurtEventId` defaults to 2, the hurt animation id in the Bedrock protocol. Both detours forward the first six
integer/pointer registers, so extra parameters on the real functions are fine. Signatures only match one game
version, and a wrong one can crash the game. Open-source Levi Launcher mods such as BedrockTools publish
version-specific signatures and are a good place to look.

## Config

| Key | Default | Meaning |
| --- | --- | --- |
| `enabled` | `true` | whole mod on/off |
| `hitreg.hurtFlash` | `true` | predict the hurt flash |
| `hitreg.reconcileWindowMs` | `250` | wait this long for the server's matching event (50-1000) |
| `hitreg.hurtCooldownMs` | `500` | no new prediction for a target this soon after its last flash (0-2000) |
| `alerts.delays` / `alerts.delayThresholdMs` | `false` / `120` | log confirmations slower than the threshold |
| `alerts.ghosts` | `false` | log predicted hits the server never confirmed |
| `hooks.*` | empty signatures | see above |

Alerts go to the launcher log.

## Tests (run on a normal computer)

The tests use small stand-ins for the SDK (`tests/stubs`), so they check this project's logic, not the real SDK.

```
g++ -std=c++20 -Wall -Wextra -Isrc -Itests/stubs -pthread tests/bridge_test.cpp src/hooks/BedrockBridge.cpp -o bridge_test && ./bridge_test
g++ -std=c++20 -Wall -Wextra -Isrc -Itests/stubs tests/reconciler_test.cpp -o reconciler_test && ./reconciler_test
```

## Notes

* Some servers forbid client mods. Check the rules of anywhere you play.
* The build has not been run against the real SDK yet. If the first CI run fails, the log will point at the line.

## Credits

* Original Java mod: **BetterHitreg** by Jass, Apache-2.0 — https://github.com/jasspir/BetterHitreg
* SDK: https://github.com/LiteLDev/preloader-android · Format docs: https://levilaunchroid.levimc.org/guide/developer
