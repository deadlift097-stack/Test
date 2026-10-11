# Build BetterHitreg on Android using only your phone

This project is source code, not an installable `.levipack` yet. The launcher requires a compiled Android native library named `libBetterHitreg.so`; renaming a `.zip` to `.so` or putting source code in a zip will not work.

## Build from a phone with GitHub Actions

1. Sign in to GitHub in your browser and create a **new private repository**.
2. Upload the contents of this project, including the hidden `.github/workflows/build.yml` file. If the mobile upload UI skips `.github`, create `.github/workflows/build.yml` manually and paste the workflow from this project.
3. Open the repository's **Actions** tab, enable Actions if asked, choose **Build BetterHitreg.levipack**, and tap **Run workflow**.
4. Wait for the workflow to finish. If it succeeds, open its run, scroll to **Artifacts**, and download `BetterHitreg-levipack`. The downloaded artifact is a zip containing the `.levipack` file. Extract it and import the `.levipack` in Levi Launcher.
5. The current source has no version-specific game hook signatures. Until valid signatures for your exact Minecraft Bedrock version are added to `config/config.json`, the mod intentionally does not hook the game and cannot provide hit prediction. Incorrect signatures can crash the game.

## What is verified here

The three local C++ logic tests pass when compiled with the commands in `README.md`. This does **not** verify the Android NDK build, compatibility with your Levi Launcher version, or in-game behavior. This environment cannot install the package into your phone or validate it inside Minecraft.

## Required details for a real in-game test

- Exact Minecraft Bedrock version/build number
- Levi Launcher version
- Valid attack-function and entity-event signatures for that exact game build

Do not enable hooks with guessed signatures. Keep the hooks empty until verified.
