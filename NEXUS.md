# Casualty Vitals

Casualty Vitals replaces the vanilla Casualties Unknown ECG panel with a richer patient monitor powered by a BepInEx 5 plugin. It adds multi-channel waveform viewing, progressive clinical-ish complications, alarm tinting, manual defibrillator support, and a few strange but medically flavored edge cases.

This is an experimental gameplay/immersion mod. It is not medical advice, a diagnosis tool, or a realistic clinical simulator.

## Requirements

- Casualties Unknown Demo
- BepInEx 5 for Unity Mono
- RSHLib, optional for the monitor but required for the custom medical solution items
- Windows x64 game install

RSHLib is required for the custom medical solution items. Casualty Vitals treats it as an optional dependency so the monitor can still load if RSHLib is missing; without RSHLib, the custom treatment bottles will not register.

## Installation

1. Install BepInEx 5 into the Casualties Unknown game folder.
2. Launch the game once so BepInEx creates its folders.
3. Optional, but recommended: install `RshLib.dll` into:

```text
<Game Folder>\BepInEx\plugins\
```

4. Copy `CasualtyVitals.dll` into the same plugins folder.
5. Launch the game.
6. Check `BepInEx\LogOutput.log` for:

```text
Plugin CasualtyVitals is loaded.
```

The mod keeps normal runtime logging quiet. If the log grows rapidly, look for warnings/errors rather than routine status spam.

## What It Does

- Replaces the vanilla health-panel ECG display with a custom monitor overlay.
- Adds selectable waveform channels:
  - `HEART`
  - `SPO2`
  - `RESP`
  - `ABP`
  - `ETCO2`
- Lets channels be previewed by hovering and pinned by clicking.
- Adds fast channel blending to reduce flicker when the vanilla UI changes rapidly.
- Adds monitor alarm tinting instead of large alarm text covering the trace.
- Pulses affected channel buttons during alarms.
- Supports a photosensitive alarm mode with slower, softer flashing.
- Hides the custom health monitor during unchipped mode so the vanilla behavior can show through.
- Adds a manual pulse check for unchipped or low-information playthroughs.
- Adds custom electrolyte treatment items through RSHLib.
- Adds elder thornback dread effects with moodles, heartbeat audio, panic HR, and post-fight adrenaline crash.
- Adds optional gameplay complications that can push vanilla Body fields when enabled.

## Manual Defibrillator Support

The manual defibrillator minigame receives its own limited monitor overlay.

It shows:

- `HEART`
- `SPO2`
- `ABP`

It does not show:

- `RESP`
- `ETCO2`

AED/manual defib shocks create a short noisy electrical artifact on the heart trace. Higher manual defib charge, or AED shocks, make the artifact slightly more erratic and last a little longer. In unchipped mode, the manual defib monitor can still show the shock artifact while the normal health monitor stays hidden.

Manual defibrillators can also be made rarer in generic loot/trader pools, or made more fragile after each shock, through config. This does not intentionally nerf AEDs.

## Physiology Layer

Casualty Vitals reads the current Body state and turns it into monitor behavior instead of only drawing the raw vanilla ECG.

Supported/estimated complication patterns include:

- Hypovolemia
- Hypoxia
- Respiratory failure
- Shock
- Angina/chest discomfort
- Ventricular instability
- Waterlogging/drowning-like respiratory stress
- Hyperkalemia from banana overload
- Hypercalcemia from electrolyte medication overdose
- Electrolyte medication toxicity
- Elder thornback panic/adrenaline states
- EtCO2 changes from ventilation, obstruction, waterlogging, perfusion, shock, and external API modifiers

These conditions are progressive. The monitor should often show deterioration before the raw numbers look completely catastrophic.

## Waveform Changes

The monitor can display:

- Sinus rhythm
- Bradycardia
- Tachycardia
- SVT-like fast rhythm
- VTach (now explicitly organized, monomorphic wide-complex)
- VFib (now fades in continuously instead of snapping)
- PEA-like electrical activity with poor/no perfusion
- Asystole (now decays gradually instead of snapping flat)
- Defib shock artifacts
- Hyperkalemia morphology with reduced P waves and tall peaked T waves
- Respiratory distress, apnea, waterlogged, and agonal-like patterns
- Pleth quality loss with poor oxygen/perfusion
- ABP loss with low pressure, low blood volume, arrest, or severe ventricular instability
- Synthetic/robotic ABP morphology when a charged autopump appears to be actively supporting low pressure
- EtCO2 plateau changes, including higher/waterlogged patterns

## Manual Pulse Checks

Holding left click with an empty cursor on the head, wrist, hand, or forearm in the health panel reveals a faint manual pulse trace after a short hold. This is mainly for unchipped playthroughs, but it can also be used in normal runs if you want a rough second opinion without relying fully on the monitor.

The manual trace is intentionally approximate. Higher intelligence improves interpretation: estimates appear with fewer beats, the BPM number tightens faster, and the trace is a little cleaner. Low intelligence takes longer and produces a rougher read.

During or near fibrillation, the manual pulse check becomes hard to interpret instead of giving a clean diagnostic VFib strip. If the patient is crashing that badly, the pulse check should feel like a warning sign, not a perfect monitor.

## Mod API

Casualty Vitals v0.2.0+ exposes `CasualtyVitalsApi` for other BepInEx mods. This allows other mods to safely read `VitalsSnapshot` without touching vanilla fields, inject additive condition burdens (`ExternalConditionSignal`), or apply custom waveform modifiers.

## Gameplay Effects

The mod can be used as a mostly visual monitor upgrade, or as a gameplay-affecting complication layer.

When `EnableGameplayEffects` is enabled, developing conditions can nudge vanilla Body fields. This can affect stamina, shock, chest pain, abdomen pain, hemothorax pressure, oxygen, respiratory behavior, blood pressure behavior, and fibrillation risk.

The goal is not instant punishment. The goal is to make subtle monitor deterioration matter.

Major gameplay-facing additions include:

- Hyperkalemia can progress from repeated banana eating and can contribute to chest pain, ECG changes, ventricular instability, and fibrillation risk if ignored.
- Hypercalcemia can occur from calcium gluconate overdose and can push angina/rhythm burden.
- Calcium gluconate and calcitonin solution are real custom items, not renamed vanilla saline.
- Both solutions can be consumed orally from the radial menu or injected through the health panel.
- Oral use takes about 14 mL and acts slowly with lower overdose risk.
- Injection uses the syringe minigame and can deliver much more at once, so overdose is easier.
- Each bottle holds 100 mL; injecting more than about 30 mL of either medication is the danger zone.
- Electrolyte medication overdose adds its own chipped-only moodle instead of pretending to be opiate overdose.
- Elder thornbacks can trigger Sense of Impending Doom, Horrified, and Focused moodles, boost heartbeat audio, and push panic heart rate toward a configurable maximum.
- After a thornback encounter ends, the heartbeat fades down over time and the adrenaline crash can lightly drain stamina.
- Thornback panic suppresses the monitor's hypertensive crisis reaction so dread HR spikes do not immediately become a hypertensive crisis by themselves.

For multiplayer, host-driven gameplay effects are the safest setup. Client-only monitor display is much safer than multiple clients writing vanilla Body fields at the same time.

## Custom Medical Items

RSHLib registers two electrolyte solution items:

- `Calcium Gluconate Solution`
- `Calcitonin Solution`

Both use embedded sprites, spawn in medical loot pools, and can occasionally appear at traders. They are liquid containers with 100 mL of solution, custom liquid colors, custom locale text, and health-panel injection support.

Calcium gluconate stabilizes dangerous hyperkalemia. Calcitonin counteracts hypercalcemia after electrolyte treatment overdose. They are meant to be dosed carefully: partial oral doses are safer and slower, while health-panel injection is faster but can overdose hard if you dump too much.

## Banana Hyperkalemia

Because someone had to make potassium a gameplay mechanic:

- Eating more than about 1.5 bananas inside roughly 3 minutes starts the risk.
- Each extra half-banana step after that rolls a 50% chance.
- Each failed roll adds hyperkalemia progress.
- A very unlucky binge can max out the condition.
- Hyperkalemia decays by itself, but very slowly.
- Hyperkalemia contributes to angina burden, chest discomfort, abdomen discomfort, ventricular instability, ECG changes, and fibrillation risk.

## Configuration

After first launch, BepInEx creates:

```text
<Game Folder>\BepInEx\config\CasualtyVitals.cfg
```

Useful settings:

```ini
[Complications]
EnableGameplayEffects = false
EnableAutoPumpDiscomfort = false
EnableThornbackDread = true

[Accessibility]
PhotosensitiveAlarmMode = false

[Monitor]
ProcessingDelayMs = 120
WaveformTimeScale = 0.85

[Thornback Dread]
AuraDistance = 28
PanicDistance = 9.5
MaxHeartRate = 200
PeakHoldSeconds = 4.5
CrashStaminaDrain = 0.18

[Manual Pulse ECG]
EnableManualPulseEcg = true
HoldSeconds = 3.5
HistoryScale = 1.4
DisplayScale = 0
HeadOffsetX = 0
HeadOffsetY = 0
LeftHandOffsetX = 0
LeftHandOffsetY = 0
RightHandOffsetX = 0
RightHandOffsetY = 0

[Manual Defibrillator]
LootChanceMultiplier = 1
ShockConditionLoss = 0

[Debug]
EnableDebugHotkeys = false
```

`EnableGameplayEffects = false` keeps the monitor behavior without letting the complication layer push vanilla Body fields.

`EnableThornbackDread` is separate from `EnableGameplayEffects`; turn it off if you only want the monitor/items/API without thornback panic physiology.

`PhotosensitiveAlarmMode = true` uses slower, softer alarm pulses.

`ProcessingDelayMs` tunes visual delay against the game's heartbeat/audio feel.

`WaveformTimeScale` slows or speeds the monitor trace without changing the actual physiology.

Manual pulse ECG `DisplayScale = 0` auto-scales the probe from a 1080p baseline: about `1` at 1080p, `1.33` at 1440p, and `2` at 4K. Set a positive value to override it.

Manual pulse ECG offsets use 1080p-reference pixels and are scaled automatically by screen height. Common presets: `150` at 1080p is about `200` at 1440p and `300` at 4K; `-120` becomes about `-160` and `-240`; `-200` becomes about `-267` and `-400`. The probe also nudges away if it would cover the limb being checked.

`LootChanceMultiplier = 0` removes manual defibrillators from generic loot/trader pools. Values above `1` make them more common there. Direct special-case world spawns may still be controlled by the base game.

`ShockConditionLoss` only applies to the manual defibrillator. The AED is not intentionally affected.

## Debug Hotkeys

Debug hotkeys are disabled by default. Enable them in the config if you want to test the monitor quickly.

- `F7` - hemorrhage / hypovolemia
- `F8` - hypoxia
- `F9` - ventricular instability
- `F10` - thoracic / respiratory complication
- `F11` - PEA-like state

These hotkeys intentionally alter the current patient.

The mod also adds console commands for testing internal Casualty Vitals fields:

```text
cvlistbodyfields
cvsetbodyfield <field> <value> [seconds]
cvgetbodyfield <field>
```

See `TESTING.md` in the source package for field examples such as hyperkalemia burden, hypercalcemia burden, electrolyte drug load, EtCO2 modifiers, and transient artifacts.

## Compatibility

This mod is a BepInEx plugin that overlays and disables specific vanilla ECG UI components while active. It may conflict with mods that replace the same health-panel ECG, wound panel, or manual defibrillator minigame UI.

It should be compatible with unrelated item, world, audio, and quality-of-life mods unless they heavily alter the same UI or Body fields.

Mods that add custom items should cooperate through RSHLib. Casualty Vitals uses RSHLib for its two treatment solutions to avoid fighting other custom-item mods.

## Known Limitations

- The mod estimates some hidden game states because not every useful Body/device detail is exposed cleanly.
- Autopump activity is inferred from pressure behavior and wearable/battery state.
- The monitor is stylized for Casualties Unknown rather than a strict medical simulator.
- Some severe or nonsensical vanilla states can still produce odd combinations, because Casualties Unknown allows odd physiology.
- The manual defib monitor is intentionally limited and not perfectly accurate.
- Manual pulse checks are approximate and intentionally affected by intelligence.
- Setting `heartRate` to extremely high debug values can crash the game. This is usually only a risk with absurd values in the billions or higher, not normal gameplay.
- Gameplay-affecting complications should generally be host-driven in multiplayer.
- RSHLib must load before the custom solution items can register.

## Troubleshooting

If the monitor does not appear:

- Confirm BepInEx 5 loads.
- Confirm `CasualtyVitals.dll` is in `BepInEx\plugins`.
- Confirm `RshLib.dll` is in `BepInEx\plugins`.
- Confirm `LogOutput.log` contains `Plugin CasualtyVitals is loaded.`
- Open the health/wound panel; the overlay attaches to the game's ECG UI.
- If using other UI mods, try disabling anything that changes the health panel or defibrillator UI.

If the custom medical solutions do not appear:

- Confirm RSHLib loaded without errors.
- Check medical crates and trader inventories over multiple runs; the items are weighted into pools, not guaranteed in every container.
- External PNG files are not required for the normal release because the item sprites are embedded in `CasualtyVitals.dll`.

If the log file gets noisy:

- Check for repeated warnings or errors from `Casualty Vitals`.
- Routine status messages are intentionally suppressed.
- Debug hotkey messages only appear when debug hotkeys are enabled and pressed.

If alarm flashing is uncomfortable:

- Set `PhotosensitiveAlarmMode = true`.

## Uninstall

Delete:

```text
<Game Folder>\BepInEx\plugins\CasualtyVitals.dll
```

If no other installed mod needs RSHLib, you can also remove:

```text
<Game Folder>\BepInEx\plugins\RshLib.dll
```

Optional:

```text
<Game Folder>\BepInEx\config\CasualtyVitals.cfg
```

## Credits

- Built with BepInEx 5.
- Inspired by the StatPatMon / VitalSign monitor simulation work.
- Developed for experimental Casualties Unknown medical-monitor modding.

## Version

Current workspace version: `0.2.0`

This build is experimental and may change as the underlying game and mod hooks are better understood.
