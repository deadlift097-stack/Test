# Notice

This project is a Bedrock / Levi Launcher implementation inspired by the Java Edition mod **BetterHitreg**
by **Jass**, which is licensed under the Apache License 2.0.

* Original mod: https://github.com/jasspir/BetterHitreg (also on Modrinth: https://modrinth.com/mod/betterhitreg)
* Original description: "Clientside mod that registers hits before the server to make hitreg feel faster"

No source code from the Java mod is included here; the C++ code in `src/` is new. The Apache-2.0 license text
of the original is kept in `LICENSE`. Names, ideas and the config option names that carry over are credited to
the original author.

Build dependencies fetched at build time keep their own licenses: preloader-android (LiteLDev), nlohmann/json,
Boost.PFR, magic_enum and {fmt}.
