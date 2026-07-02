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

`CasualtyVitalsApi.ApiVersion` is currently `"1.1"`.

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
- `WaterExposure`

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

The burden is clamped `0..1` and fades over `duration` seconds. Current built-in
condition IDs read by the engine are:

- `Angina`
- `Hypoxia`
- `Shock`
- `Respiratory Failure`
- `Ventricular Instability`
- `Hyperkalemia`
- `Hypercalcemia`

This is best for "make the monitor/condition engine care about my temporary
effect" without directly setting vitals.

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

Fields are additive burdens clamped `0..1`; Casualty Vitals uses the maximum
contribution across providers each frame. `EtCO2Offset` is signed in mmHg-like
monitor units and is summed/clamped; `EtCO2ObstructionBlend` and
`EtCO2WaterlogNoise` are max-blended.

Remove when your mod unloads or no longer needs the hook:

```csharp
CasualtyVitalsApi.RemoveConditionProvider("my_mod_radiation");
```

Provider exceptions are caught and the provider is silenced for a short cooldown.

## Waveform Modifiers

Waveform modifiers can adjust `SignalParameters` immediately before waveform
generation:

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

Waveform modifiers should be lightweight and deterministic per frame where
possible.

## Transient Artifacts

For a short visual disturbance:

```csharp
CasualtyVitalsApi.AddTransientArtifact(body, "my_mod_electrical_noise", 0.8f, 3f);
```

This is intended for monitor artifacts rather than long-running physiology.

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

Use this for airway obstruction, ventilation changes, procedural artifacts,
chemical exposure, or external CPR/ventilation mods.

## What The API Does Not Do

The public API does not directly set `Body.heartRate`, blood pressure, oxygen,
or other vanilla fields. Mods can still edit vanilla fields themselves, but
Casualty Vitals only promises compatibility for its public read/add/modify API.

EtCO2 can be modified through condition provider fields, waveform modifiers, or
`AddEtCO2Modifier`. Direct vanilla `Body` field mutation is still outside the
Casualty Vitals compatibility contract.

## Multiplayer Notes

Gameplay-affecting changes should generally be driven by the host. Reading
snapshots and drawing client-only UI is safer than writing vanilla `Body` fields
from every client.
