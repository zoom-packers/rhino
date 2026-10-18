# Rhino for Minecraft 26.3 / NeoForge 26.3.0.89 (unofficial build)

Branch `26.3`: an **unofficial** build of KubeJS's Rhino fork for Minecraft 26.3, made for the Zoompack 5 modpack
together with the KubeJS port in `../KubeJS` (see its `PORTING-26.3.md`). Rhino is © Mozilla and contributors, with
modifications by latvian.dev and contributors, licensed under the Mozilla Public License 2.0 (`LICENSE`); this build
keeps that licence.

Version: `2603.2.8-port.1`.

## Where it comes from

| Ref | What it is |
|---|---|
| `origin/main` (`c08084e`) | Upstream Rhino `2101.2.8`, built for 1.21.1 / NeoForge 21.1 and also used by KubeJS 26.1.2. |
| `wolfie/main` (`c8275d7`) | [KubeJS-Mods/Rhino#78](https://github.com/KubeJS-Mods/Rhino/pull/78) by **wolfieboy09** (titled "Rhino 26.4", contents are 26.3): ModDevGradle 2.0.147, Java 25, `logoFile` -> `iconFile`, MC 26.3 / NeoForge 26.3.0.76. Up to date with `origin/main`. |
| `26.3` (this branch) | `wolfie/main` + the changes below. |

Rhino itself has no Minecraft or NeoForge code: it is a JavaScript engine shipped as a library mod. The upstream
`2101.2.8-build.91` jar would also load on 26.3. The port is about building it with the 26.3 toolchain and giving it a
version that sorts above the 26.1 one.

## Changes on top of #78

- `neoforge_version=26.3.0.89`.
- Gradle wrapper 8.11.1 -> **9.2.1**: Gradle 8.11 cannot run on JDK 25 ("Unsupported class file major version 69").
  For Gradle 9: `base.archivesBaseName` -> `base.archivesName`, and `org.junit.platform:junit-platform-launcher:1.8.2`
  added as `testRuntimeOnly` (Gradle 9 no longer adds it).
- `mod_version=2603.2.8` (#78 used `26.3-1.0`, which sorts *below* `2101.2.8-build.91`, so KubeJS's
  `rhino [<version>,)` dependency would reject it). Follows latvian's `<mc>.<major>.<minor>` scheme, as BAT
  `2603.1.1` does.
- `-Pbuild_tag=<tag>` pins the version suffix (local builds used `local.<epoch>`); the port uses `-Pbuild_tag=port.1`
  so KubeJS can depend on a stable `2603.2.8-port.1`.
- `neoforge.mods.toml`: dropped `modLoader = "lowcodefml"` / `loaderVersion`. FML 12 warns that `lowcodefml` is
  deprecated and says to remove them; a mods.toml without them loads as a plain library mod. The support URL says
  `mc=2603`.

## What's disabled

Nothing. The `unified-publishing` plugin and `publishing` blocks are untouched (they only act with CI credentials).

## Tests

`./gradlew build` runs the JUnit suite on JDK 25: **170 tests, 0 failures, 0 errors, 0 skipped**.
It also loads in NeoForge 26.3.0.89 with KubeJS (`../smoketest`, all 8 GameTests pass), with no warnings from FML.

## Script-API differences

None from Rhino itself; the engine code is identical to upstream `main` (`c08084e`). Script-facing changes on 26.3 come
from KubeJS and vanilla (see the KubeJS notes).

## Building

```
JAVA_HOME=<JDK 25> ./gradlew build publishToMavenLocal -Pbuild_tag=port.1
```

## Keeping it rebased

Remotes: `origin` = KubeJS-Mods/Rhino, `wolfie` = wolfieboy09/Rhino (PR #78's head branch `main`). The port is committed on `26.3` on top of `wolfie/main` (local only, not pushed). To
update:

```
git fetch origin wolfie
git rebase wolfie/main       # while #78 is open
git rebase origin/main       # once #78 is merged
```

Rhino changes rarely conflict with this (only build files and `neoforge.mods.toml`). When an official 26.3 Rhino ships,
use it and bump KubeJS's `rhinoVersion`. Note that `2603.2.8-port.1` sorts above `2603.2.8-build.N`
("port" > "build" as Maven qualifiers), so a KubeJS port requiring `[2603.2.8-port.1,)` would reject an official
`2603.2.8-build.N`; lower the range then.

## Contributing upstream

The Gradle 9 / JDK 25 fixes, the version scheme and the `lowcodefml` removal could go to #78 as review suggestions
(the user decides whether to send them).
