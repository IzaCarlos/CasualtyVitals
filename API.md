# Casualty Vitals Developer API

Casualty Vitals exposes a small public API for other BepInEx mods through the
`CasualtyVitals` namespace. The API is designed for cooperative mods: read the
monitor state, add temporary burdens/artifacts, or modify the rendered waveform
without directly fighting vanilla `Body` fields.

## Referencing

Reference `CasualtyVitals.dll` from your mod project if you want compile-time
types:

```xml
<Reference Include="CasualtyVitals">
  <HintPath>path\to\BepInEx\plugins\CasualtyVitals.dll</HintPath>
  <Private>false</Private>
</Reference>
```

Add a soft or hard dependency depending on whether your mod can run without it:

```csharp
[BepInDependency("CasualtyVitals", BepInDependency.DependencyFlags.SoftDependency)]
```

Check availability before using live data:

```csharp
if (CasualtyVitalsApi.IsAvailable &&
    CasualtyVitalsApi.TryGetVitals(body, out VitalsSnapshot v))
{
    Logger.LogInfo($"HR {v.HR:0}, SpO2 {v.SpO2:0}, EtCO2 {v.EtCO2:0}");
}
```

`CasualtyVitalsApi.ApiVersion` is currently `"1.4"`.

## Contract at a Glance

- All IDs are case-sensitive. Adding the same ID again replaces that entry.
- Public timed-entry and registration calls fail soft: a null body, empty ID,
  null callback, or non-positive duration is ignored rather than throwing.
- Numeric inputs are required to be finite. Non-finite burdens and signal fields
  are replaced with safe defaults; extreme finite values are clamped to the
  documented ranges. Timed durations are capped at one hour.
- All API calls, providers, and modifiers must run on Unity's main thread. The
  API collections are not thread-safe.
- The API controls monitor presentation only. It does not write heart rate,
  oxygen, pressure, consciousness, damage, inventory, or multiplayer state.
- Readouts are game-derived monitor estimates, not a medical simulation contract.

## Readable Data

`VitalsSnapshot` is the main read-only view. It is published once per monitor
update and can be queried by `Body`.

Fields:

- `HR`
- `SpO2`
- `RR`
- `BPSys`
- `BPDia`
- `EtCO2`
- `HeartProg`
- `FibrillationProgress`
- `VFibChaos`
- `AsystoleDecay`
- `MechanicalPerfusion`
- `ElectricalStateHint`
- `HyperkalemiaBurden`
- `HypercalcemiaBurden`
- `RadiationPercent`
- `RadiationDoseGy`
- `RadiationBurden`
- `RadiationSourceBurden`
- `WaterExposure`
- `SepsisBurden`
- `SystemicIllnessBurden`
- `OpiateReception` (signed; negative means withdrawal/craving)
- `VenomBurden`

You can also subscribe to updates:

```csharp
CasualtyVitalsApi.OnVitalsUpdated += OnVitalsUpdated;

private void OnVitalsUpdated(Body body, VitalsSnapshot v)
{
    if (v.ElectricalStateHint == "VTach")
    {
        // React without touching the monitor internals.
    }
}
```

Subscriber exceptions are caught by Casualty Vitals so one bad listener should
not crash the monitor.

## Add Condition Burdens

For simple timed effects, call `AddConditionBurden`.

```csharp
CasualtyVitalsApi.AddConditionBurden(body, "my_mod_toxin", 0.65f, 20f);
```

The burden is clamped `0..1` and fades over `duration` seconds. Its ID is only
your ownership key; it does not select a built-in diagnosis. Each active timed
burden contributes a fading ventricular/ectopic monitor drive, without writing
vanilla `Body` fields. Use a condition provider when you need respiratory,
hypoxia, noise, or capnography contributions as well.

## Condition Providers

For continuous effects, register a provider:

```csharp
CasualtyVitalsApi.AddConditionProvider("my_mod_radiation", body =>
{
    return new ExternalConditionSignal
    {
        VentricularBurden = 0.25f,
        RespBurden = 0.10f,
        HypoxiaBurden = 0.20f,
        BaselineNoise = 0.35f,
        EtCO2Offset = 8f,
        EtCO2ObstructionBlend = 0.4f,
    };
});
```

`VentricularBurden`, `RespBurden`, `HypoxiaBurden`, `BaselineNoise`,
`EtCO2ObstructionBlend`, and `EtCO2WaterlogNoise` accept `0..1` and are
max-blended across providers. `EtCO2Offset` is signed in mmHg-like monitor units
and is summed, then clamped to `-80..80`. Providers run during a monitor update:
keep them fast, allocation-light, and non-blocking.

Remove when your mod unloads or no longer needs the hook:

```csharp
CasualtyVitalsApi.RemoveConditionProvider("my_mod_radiation");
```

Provider exceptions are caught, logged, and silence that provider for five
seconds. Registration changes made while a provider is running take effect on a
later monitor update.

## Waveform Modifiers

Waveform modifiers can adjust `SignalParameters` once per monitor update,
immediately before waveform generation:

```csharp
public sealed class TremorArtifact : IWaveformModifier
{
    public void ModifySignal(ref SignalParameters p)
    {
        p.BaselineNoise = Mathf.Max(p.BaselineNoise, 0.45f);
        p.BaselineWander = Mathf.Max(p.BaselineWander, 0.25f);
    }
}

CasualtyVitalsApi.AddWaveformModifier("my_mod_tremor", new TremorArtifact());
```

Remove with:

```csharp
CasualtyVitalsApi.RemoveWaveformModifier("my_mod_tremor");
```

Useful `SignalParameters` groups:

- Electrical: `EffectiveHR`, `QrsWidth`, `PWaveAmplitude`, `TWaveAmplitude`,
  `TWavePeaking`, `StOffset`, `EctopicProbability`, `VFibChaos`,
  `AsystoleDecay`, `BaselineNoise`, `BaselineWander`.
- Mechanical/perfusion: `MechanicalPerfusion`, `PlethQuality`, `AbpSystolic`,
  `AbpMorphologyType`, `AbpPerfusion`.
- Respiratory: `RespDrive`, `RespRate`, `IsBreathing`, distress/apnea blends,
  `RespAmplitude`.
- EtCO2: `EtCO2Value`, `EtCO2Scale`, `EtCO2Height`,
  `EtCO2ObstructionBlend`, `EtCO2WaterlogNoise`.

Most blend fields expect `0..1`; `EffectiveHR` accepts `0..360`,
`QrsDominantPolarity` accepts `-1..1`, and `StOffset` accepts `-1..1`.
Casualty Vitals sanitizes the complete parameter set after each modifier, so
`NaN`, infinity, and extreme values cannot poison the waveform or later frames.
Do not rely on that as normal control flow: modifiers should still return finite,
internally consistent values. A later modifier may replace an earlier modifier's
choice. Modifier exceptions are caught, logged, and silence that ID for five
seconds.

## Transient Artifacts

For a short visual disturbance:

```csharp
CasualtyVitalsApi.AddTransientArtifact(body, "my_mod_electrical_noise", 0.8f, 3f);
```

This is intended for monitor artifacts rather than long-running physiology. The
strength is clamped `0..1`; reuse the same ID to replace the prior artifact.

## Debug Helpers

Debug tooling can manipulate Casualty Vitals-only condition fields without
directly reaching into tracker internals:

```csharp
CasualtyVitalsApi.TrySetDebugConditionField(body, "hyperkalemiaBurden", 0.5f);
CasualtyVitalsApi.ResetDebugState(body);
```

Radiation helpers use the game's percentage scale, where `100` means roughly
`30 Gy`:

```csharp
CasualtyVitalsApi.SetDebugRadiationPercent(body, "my_debug_ui", 60f, 0.35f);
CasualtyVitalsApi.SetDebugRadiationDose(body, "my_debug_ui", 12f, 0.35f);
```

## EtCO2 Modifiers

For a timed EtCO2-specific effect:

```csharp
CasualtyVitalsApi.AddEtCO2Modifier(
    body,
    "my_mod_airway_obstruction",
    offset: 12f,
    scaleMultiplier: 1.15f,
    obstructionBlend: 0.85f,
    waterlogNoise: 0.0f,
    duration: 18f);
```

Parameters:

- `offset`: signed mmHg-like value added to EtCO2.
- `scaleMultiplier`: `1` is neutral, `0` flattens the capnogram, values above
  `1` amplify it.
- `obstructionBlend`: `0..1` shark-fin/slurred upstroke and rising plateau.
- `waterlogNoise`: `0..1` noisy wet plateau.
- `duration`: seconds before the modifier fades out.

Per-entry `offset` is clamped to `-60..60`, and `scaleMultiplier` to `0..2.5`.
The ID is case-sensitive and replaces an existing entry with that ID.

Use this for airway obstruction, ventilation changes, procedural artifacts,
chemical exposure, or external CPR/ventilation mods.

## Limits and Failure Behavior

`OnVitalsUpdated` subscriber exceptions are caught so they cannot crash the
monitor; a throwing subscriber remains subscribed. The API does not guarantee a
fixed UI layout or compatibility with mods that replace the game's monitor
components outright.

EtCO2 can be modified through condition providers, waveform modifiers, or
`AddEtCO2Modifier`. Mods may edit vanilla `Body` fields themselves, but that is
outside the Casualty Vitals compatibility contract. Keep gameplay-affecting
writes host-authoritative in multiplayer.

## Multiplayer Notes

Gameplay-affecting changes should generally be driven by the host. Reading
snapshots and drawing client-only UI is safer than writing vanilla `Body` fields
from every client.
