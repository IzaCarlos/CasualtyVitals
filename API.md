# Casualty Vitals Developer API (v2.0)

Casualty Vitals exposes a comprehensive public API for external BepInEx mods through the `CasualtyVitals` namespace. The API is designed for cooperative modding: read vitals and electrophysiology state, dictate minimum simulation mode tiers, drive causal physiology, register custom rhythm/conduction ailments, inject waveform artifacts, trigger pharmacological responses, or listen to clinical events without fighting vanilla `Body` fields.

---

## Table of Contents
1. [Referencing & Setup](#1-referencing--setup)
2. [Simulation Mode Detection & Requirements](#2-simulation-mode-detection--requirements)
3. [Reading Vitals, Snapshots & Hemodynamics](#3-reading-vitals-snapshots--hemodynamics)
4. [Subscribing to Clinical Events](#4-subscribing-to-clinical-events)
5. [Controlling Electrophysiology & Rhythm State](#5-controlling-electrophysiology--rhythm-state)
6. [Electrolytes, Blood Gases & Metabolic Interventions](#6-electrolytes-blood-gases--metabolic-interventions)
7. [Interacting with & Overriding Custom Items](#7-interacting-with--overriding-custom-items)
8. [Continuous Physiology & Waveform Modifiers](#8-continuous-physiology--waveform-modifiers)
9. [Error Handling, Null Safety & Isolation Guarantees](#9-error-handling-null-safety--isolation-guarantees)

---

## 1. Referencing & Setup

Reference `CasualtyVitals.dll` from your mod project:

```xml
<Reference Include="CasualtyVitals">
  <HintPath>path\to\BepInEx\plugins\CasualtyVitals.dll</HintPath>
  <Private>false</Private>
</Reference>
```

Add a dependency attribute to your `BaseUnityPlugin`:

```csharp
[BepInDependency("CasualtyVitals", BepInDependency.DependencyFlags.SoftDependency)]
```

Check API availability before invoking queries:

```csharp
if (CasualtyVitalsApi.IsAvailable && CasualtyVitalsApi.TryGetVitals(body, out VitalsSnapshot v))
{
    Logger.LogInfo($"Rhythm: {v.Rhythm}, HR: {v.HR:0}, BP: {v.BPSys:0}/{v.BPDia:0}, SpO2: {v.SpO2:0}%, K+: {v.Potassium:0.0} mmol/L");
}
```

`CasualtyVitalsApi.ApiVersion` is `"2.0"`.

---

## 2. Simulation Mode Detection & Requirements

CasualtyVitals supports 4 simulation complexity tiers:
- **`SimulationMode.ECG` (Tier 0)**: Monitor visuals and lead projections only (no extra burden calculations).
- **`SimulationMode.Simple` (Tier 1)**: Heuristic burden calculations with hyperkalemia and hypercalcemia.
- **`SimulationMode.Custom` (Tier 2)**: Player-selected feature flags.
- **`SimulationMode.Advanced` (Tier 3)**: Full causal electrophysiology state machine, acid-base metabolism, and blood gas calculations.

### Detecting Active Mode
```csharp
SimulationMode activeMode = CasualtyVitalsApi.CurrentSimulationMode;
// Or:
SimulationMode mode = CasualtyVitalsApi.GetSimulationMode();
```

### Requiring / Forcing a Minimum Simulation Mode
If your mod requires Advanced causal physiology (e.g. potassium shifts, blood gases, or AV block state machines), declare your requirement programmatically. If multiple mods declare requirements, CasualtyVitals automatically resolves to the highest tier:

```csharp
// Require Advanced mode with caller ID and reason
CasualtyVitalsApi.RequireSimulationMode(
    modId: "com.author.toxicology", 
    requiredMode: SimulationMode.Advanced, 
    reason: "Requires causal potassium and arterial blood gas calculations");

// When done or if your feature unloads:
CasualtyVitalsApi.ReleaseSimulationModeRequirement("com.author.toxicology");

// Inspect all active mod requirements:
IReadOnlyList<CasualtyVitalsApi.SimulationModeRequirement> requirements = CasualtyVitalsApi.GetActiveModeRequirements();
foreach (var req in requirements)
{
    Debug.Log($"Mod '{req.ModId}' required '{req.RequiredMode}': {req.Reason}");
}
```

---

## 3. Reading Vitals, Snapshots & Hemodynamics

### Querying Immutable Snapshots
`VitalsSnapshot` gives an immutable, thread-safe view of a patient's vitals, blood gases, electrolytes, and electrophysiology:

```csharp
if (CasualtyVitalsApi.TryGetVitals(body, out VitalsSnapshot snap))
{
    // Hemodynamics & Perfusion
    float hr = snap.HR;
    float sys = snap.BPSys;
    float dia = snap.BPDia;
    bool hasPulse = snap.PulsePresent;
    float cardiacOutput = snap.CardiacOutput; // 0..1

    // Respiratory & Blood Gas
    float rr = snap.RR;
    float spo2 = snap.SpO2;
    float etco2 = snap.EtCO2;
    float paO2 = snap.PaO2;
    float paCO2 = snap.PaCO2;
    float acidosis = snap.Acidosis; // 0..1
    RespiratoryPattern pattern = snap.RespPattern;

    // Electrophysiology
    EcgRhythm rhythm = snap.Rhythm;
    EcgLead lead = snap.ActiveLead;
    IReadOnlyCollection<EcgAilment> ailments = snap.ActiveAilments;

    // Electrolytes
    float k = snap.Potassium;  // e.g. 4.2 mmol/L
    float ca = snap.Calcium;   // e.g. 9.5 mg/dL
}
```

### Direct Hemodynamic Helpers
```csharp
// Get live blood pressure (systolic and diastolic)
CasualtyVitalsApi.GetBloodPressure(body, out float sys, out float dia);

// Calculate Mean Arterial Pressure (MAP) and Pulse Pressure
float map = CasualtyVitalsApi.GetMeanArterialPressure(body); // (Sys + 2*Dia) / 3
float pp = CasualtyVitalsApi.GetPulsePressure(body);         // Sys - Dia

// Check if a specific limb has active arterial tourniquet occlusion
bool isOccluded = CasualtyVitalsApi.IsLimbOccluded(body, limbIndex: 2);
```

---

## 4. Subscribing to Clinical Events

All event callbacks are isolated with individual exception handling to prevent external errors from disrupting the game loop:

```csharp
// Electrophysiological rhythm transitions (e.g. Sinus -> VTach -> VFib)
CasualtyVitalsApi.OnRhythmChanged += (body, oldRhythm, newRhythm) =>
{
    Debug.Log($"Rhythm transition on {body.name}: {oldRhythm} -> {newRhythm}");
};

// Conduction defects or ailments added/removed
CasualtyVitalsApi.OnAilmentAdded += (body, ailment) =>
{
    Debug.Log($"Patient developed ECG defect: {ailment}");
};

CasualtyVitalsApi.OnAilmentRemoved += (body, ailment) =>
{
    Debug.Log($"ECG defect resolved: {ailment}");
};

// Cardiac arrest (VFib or Asystole) detection
CasualtyVitalsApi.OnCardiacArrest += body =>
{
    Debug.LogWarning($"Cardiac arrest on body {body.name}!");
};

// Pulseless Electrical Activity (PEA) decoupling
CasualtyVitalsApi.OnPEA += body =>
{
    Debug.LogWarning($"PEA detected on body {body.name} (electrical activity without mechanical output)!");
};

// Defibrillator shock delivered
CasualtyVitalsApi.OnDefibrillationDelivered += (body, energyScale) =>
{
    Debug.Log($"Transthoracic shock delivered to {body.name} (scale {energyScale:0.0})");
};

// Per-frame vitals tick
CasualtyVitalsApi.OnVitalsUpdated += (body, snap) =>
{
    if (snap.Potassium > 7.0f && snap.Rhythm != EcgRhythm.VFib)
    {
        // React to severe hyperkalemia
    }
};
```

---

## 5. Controlling Electrophysiology & Rhythm State

### Setting Rhythms with Lock Duration
```csharp
// Command a rhythm (e.g. SVT, VFib, Third-Degree AV Block) with a lock duration in seconds
CasualtyVitalsApi.TrySetRhythm(body, EcgRhythm.SVT, lockDuration: 15f);

// Induce acute myocardial infarction / heart attack in a specific coronary territory
CasualtyVitalsApi.TryTriggerHeartAttack(body, IschemiaTerritory.Anterior, severity: 0.90f);
```

### Adding / Removing Conduction Ailments
Ailments automatically respect mutual-exclusion rules:
```csharp
// Inject ectopy or STEMI pattern
CasualtyVitalsApi.TryAddAilment(body, EcgAilment.Bigeminy);
CasualtyVitalsApi.TryAddAilment(body, EcgAilment.AnteriorSTEMI);

// Clear an ailment
CasualtyVitalsApi.TryRemoveAilment(body, EcgAilment.Bigeminy);
```

---

## 6. Electrolytes, Blood Gases & Metabolic Interventions

### Modifying Electrolytes & Blood Chemistry
```csharp
// Adjust blood potassium (mmol/L) and calcium (mg/dL)
CasualtyVitalsApi.TrySetElectrolytes(body, potassium: 6.8f, calcium: 11.2f);

// Administer an insulin dose (drives intracellular potassium shift: -0.35 mmol/L)
CasualtyVitalsApi.ApplyInsulinDose(body, units: 1.0f);

// Drive arterial blood gas (PaO2, PaCO2, and metabolic acidosis 0..1)
CasualtyVitalsApi.TrySetBloodGas(body, paO2: 140f, paCO2: 32f, acidosis: 0.15f);

// Set respiratory pattern (Eupnea, Kussmaul, Cheyne-Stokes, Apnea)
CasualtyVitalsApi.TrySetRespiratoryPattern(body, RespiratoryPattern.Kussmaul);
```

### Vascular Resistance & Drug Loads
```csharp
// Apply systemic vascular resistance multiplier (e.g. powerful vasoconstrictor / pressor)
CasualtyVitalsApi.SetVascularResistanceMultiplier(body, id: "my_pressor", multiplier: 1.45f, duration: 45f);

// Track custom drug load levels (arbitrary unit accumulation)
CasualtyVitalsApi.AddDrugLoad(body, "my_custom_toxin", amount: 15.0f);
float currentLoad = CasualtyVitalsApi.GetDrugLoad(body, "my_custom_toxin");
```

---

## 7. Interacting with & Overriding Custom Items

### Programmatic Treatment Triggers
You can notify CasualtyVitals of medical procedures performed by your mod:
```csharp
// Deliver a manual or AED defibrillation shock (1.0f = full energy)
CasualtyVitalsApi.NotifyDefibrillation(body, strength: 1.0f);

// Apply transcutaneous external cardiac pacing spikes
CasualtyVitalsApi.NotifyTranscutaneousPacing(body, rateBpm: 75f, currentMa: 60f);

// Apply manual or mechanical CPR chest compressions (supports perfusion and PEA drive)
CasualtyVitalsApi.NotifyChestCompressions(body, depthFactor: 1.0f);
```

### Overriding Item Treatment Logic
To override how an item behaves upon application, use `CustomMedicalItems` or hook the treatment pipeline:
```csharp
// Check if an item is a custom CV item
if (item.id == "cv_epinephrine")
{
    // Epinephrine auto-injector detected
}
```

---

## 8. Continuous Physiology & Waveform Modifiers

### Continuous Biological Modifiers (`IPhysiologyModifier`)
For continuous processes (e.g. venom absorption, hypothermia, sepsis, or custom drug kinetics):

```csharp
public class SevereHypothermiaModifier : IPhysiologyModifier
{
    public void ModifyPhysiology(Body body, PatientSimulationState state, float dt)
    {
        if (body.temperature < 30f)
        {
            // Severe hypothermia drives metabolic acidosis and prolongs QT
            state.Acidosis = Mathf.MoveTowards(state.Acidosis, 0.75f, dt * 0.02f);
            state.AddAilment(EcgAilment.LongQT);
        }
    }
}

// Register on mod Awake
CasualtyVitalsApi.AddPhysiologyModifier("hypothermia_mod", new SevereHypothermiaModifier());

// Remove on teardown
CasualtyVitalsApi.RemovePhysiologyModifier("hypothermia_mod");
```

### Synthetic Waveform & Noise Modifiers (`IWaveformModifier`)
```csharp
public class NeuroStimulatorArtifactModifier : IWaveformModifier
{
    public void ModifySignal(ref SignalParameters p)
    {
        // Inject high-frequency baseline jitter
        p.BaselineNoise += 0.08f;
    }
}

CasualtyVitalsApi.AddWaveformModifier("neuro_stim", new NeuroStimulatorArtifactModifier());
```

---

## 9. Error Handling, Null Safety & Isolation Guarantees

1. **Defensive Numeric Clamping**: All inputs to `CasualtyVitalsApi` pass through `NumericSafety`. Non-finite values (`float.NaN`, `float.PositiveInfinity`) are safely replaced with default resting values without throwing `ArithmeticException`.
2. **Provider Auto-Silencing**: If an external `IPhysiologyModifier` or `IWaveformModifier` throws an unhandled exception, it is caught, logged with the provider ID, and silenced for a cooldown period to protect frame rates and prevent game crashes.
3. **Graceful Mode Fallback**: If a mod requests an advanced feature while the user has configured `SimulationMode.ECG` without a forced requirement, the call safely succeeds with no-op or fallback heuristics.
