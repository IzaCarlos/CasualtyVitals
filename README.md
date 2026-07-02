# Casualty Vitals BepInEx Workspace

This folder is intentionally ignored by the main StatPatMon repository.

## Contents

- `refs/BepInEx.Templates-master/` - local copy of the official BepInEx templates repository.
- `src/CasualtyVitals/` - starter BepInEx 5 plugin project.
- `assets/` - place mod art, audio, and monitor resources here while experimenting.
- `dist/` - build output copied by `build.py`.

## First-Time Setup

Install a .NET SDK. Runtimes alone are not enough for `dotnet build`.

The official template install command, once an SDK exists:

```powershell
dotnet new install BepInEx.Templates --nuget-source https://nuget.bepinex.dev/v3/index.json
```

## Build

```powershell
python .\build.py
```

To build and copy into a BepInEx install:

```powershell
python .\build.py --game-dir "C:\Path\To\Casualties Unknown"
```

or:

```powershell
python .\build.py --plugins-dir "C:\Path\To\Casualties Unknown\BepInEx\plugins"
```

After launching the game, check `BepInEx\LogOutput.log` for:

```text
Casualty Vitals bootstrap is alive.
```

## Custom Medical Item Assets

Custom medical solution items require `RshLib.dll` in `BepInEx/plugins/`.
The monitor, API, and non-item features can still load without RSHLib.

Custom item sprites are embedded into `CasualtyVitals.dll` from:

```text
assets/
```

Expected filenames:

- `cv_calcium_gluconate_solution.png`
- `cv_calcitonin_solution.png`

`assets/cv_calcium_gluconate.png` is accepted as the source image for the embedded calcium sprite.

The plugin still checks `BepInEx/plugins/CasualtyVitals/` and the plugins root first, so loose PNGs can override the embedded sprites during development. Normal distribution only needs the DLL.

## Developer API

See [API.md](API.md) for the public `CasualtyVitalsApi` surface exposed to other BepInEx mods.
