# Casualty Vitals

Casualty Vitals is a BepInEx 5 patient-monitor mod for *Casualties Unknown*.
It adds multi-channel waveforms, progressive monitor states, alarms, manual
defibrillator support, and optional clinical-style monitor presentations.

This repository is intentionally documentation-only. It publishes the supported
mod-integration API, not the mod's source code, development project, build tools,
or unreleased assets.

## For players

Install the released `CasualtyVitals.dll` and its accompanying
`Assets/CasualtyVitals/` folder exactly as provided by the release.

Casualty Vitals requires BepInEx 5 and CUCoreLib.

## For mod authors

The supported integration surface is documented in [API.md](API.md). It covers:

- Reading current monitor snapshots.
- Receiving vital-update events.
- Supplying condition burdens and capnography modifiers.
- Adding short-lived artefacts.
- Safely adjusting the rendered signal through waveform modifiers.

To compile against Casualty Vitals, reference the released `CasualtyVitals.dll`
from a local game installation or release package and declare an appropriate
BepInEx dependency:

```csharp
[BepInDependency("CasualtyVitals", BepInDependency.DependencyFlags.SoftDependency)]
```

Check `CasualtyVitalsApi.IsAvailable` before using live data. The currently
documented API version is `1.3`.

## Compatibility contract

Only public types and members documented in [API.md](API.md) are supported for
third-party integration. Internal implementation details, UI hierarchy, renderer
behaviour, private fields, and undocumented configuration entries may change
without notice. Do not use reflection against Casualty Vitals as a compatibility
mechanism.

API calls and callbacks must run on Unity's main thread. Integrations should remain
fast, non-blocking, and defensive: the monitor is a presentation layer and does not
grant authority to modify another player's or the game's physiological state.

## Scope

Casualty Vitals is an experimental gameplay/immersion mod, not medical advice or a
clinical simulation. Its readouts and waveforms are game-derived estimates designed
to communicate play states clearly.
