# Casualty Vitals In-Game Test Pass

## Install

Copy `dist/CasualtyVitals.dll` to:

```text
<Game Folder>\BepInEx\plugins\CasualtyVitals.dll
```

Launch once, then check:

```text
<Game Folder>\BepInEx\LogOutput.log
```

Expected startup:

```text
Plugin CasualtyVitals is loaded.
MonitorOverlay OnEnable() called.
MonitorOverlay Start() called.
Found ECGVisualizer! Injecting custom UI...
Attached to Body: ...
```

## Optional Debug Hotkeys

After first launch, edit:

```text
<Game Folder>\BepInEx\config\CasualtyVitals.cfg
```

Set:

```ini
[Debug]
EnableDebugHotkeys = true
```

For slower alarm pulsing, set:

```ini
[Accessibility]
PhotosensitiveAlarmMode = true
```

To tune visual sync against heartbeat audio:

```ini
[Monitor]
ProcessingDelayMs = 120
```

Hotkeys:

- `F7` - hemorrhage / hypovolemia
- `F8` - hypoxia
- `F9` - ventricular instability
- `F10` - thoracic / respiratory complication
- `F11` - PEA (Pulseless Electrical Activity)

## Console Debug Commands

Casualty Vitals adds console commands for fields that vanilla `setbodyfield` cannot see:

```text
cvlistbodyfields
cvsetbodyfield <field> <value> [seconds]
cvgetbodyfield <field>
```

Settable fields:

- `hyperkalemiaBurden` - direct potassium burden, `0` to `1`.
- `hypercalcemiaBurden` - direct calcium complication burden, `0` to `1`.
- `calciumDrugLoad` - calcium gluconate overdose load. Around `1` enters overdose range.
- `calcitoninDrugLoad` - calcitonin overdose load. Around `1` enters overdose range.
- `calciumGluconateInjectionMl` / `calciumGluconateOralMl` - applies a fake treatment dose in mL.
- `calcitoninInjectionMl` / `calcitoninOralMl` - applies a fake treatment dose in mL.
- `etco2Offset` - timed EtCO2 numeric offset, in monitor units.
- `etco2Scale` - timed EtCO2 multiplier. `1` is neutral, `0` flattens.
- `etco2Obstruction` - timed shark-fin EtCO2 morphology, `0` to `1`.
- `etco2WaterlogNoise` - timed wet/noisy EtCO2 morphology, `0` to `1`.
- `transientArtifact` - timed waveform artifact strength, `0` to `1`.

Read-only:

- `medicationOverdoseLevel` - current custom medicine overdose moodle severity, `-1` to `3`.

Useful recipes:

```text
cvsetbodyfield hyperkalemiaBurden 1
cvsetbodyfield calciumDrugLoad 1.4
cvsetbodyfield calcitoninDrugLoad 1.3
cvsetbodyfield calciumGluconateInjectionMl 35
cvsetbodyfield calcitoninInjectionMl 35
cvgetbodyfield medicationOverdoseLevel
cvsetbodyfield etco2Offset 18 90
cvsetbodyfield etco2Obstruction 1 90
cvsetbodyfield etco2Scale 0.25 20
cvsetbodyfield transientArtifact 1 5
```

Angina is derived from vanilla physiology rather than stored as a direct field. For a quick angina burden test, use vanilla console fields:

```text
setbodyfield stamina 20
setbodyfield heartRate 205
setbodyfield bloodOxygen 74
setbodyfield bloodPressure 165
```

With `EnableGameplayEffects = true`, this should become much more hostile: stronger chest pain, tremor, stamina collapse, oxygen drag, and at critical burden a fibrillation/BP drop risk.

## Visual Checks

1. Open the vanilla health/wound panel and confirm the custom monitor appears where the vanilla ECG was.
2. Hover `HEART`, `SPO2`, `RESP`, `ABP`, and `ETCO2`.
3. Confirm the waveform fades quickly between channels instead of snapping/flickering.
4. Click a channel and move the cursor away.
5. Confirm the clicked channel stays pinned.
6. Toggle unchipped mode if available.
7. Confirm the health/wound panel monitor is unavailable/hidden in unchipped mode. This is expected; do not expect WoundView access.
8. Confirm the manual defib item remains the only ECG access path in unchipped mode.

## Physiology Checks

With debug hotkeys enabled:

1. Press `F7`.
   - Watch for falling blood volume/bleeding in vanilla UI.
   - Expected log: `Hypovolemia` and possibly `Shock` stages.
   - Expected monitor: weaker pleth/ABP as perfusion falls.

2. Press `F8`.
   - Expected log: `Hypoxia`.
   - Expected vanilla effects: lower stamina and increased shock pressure.
   - Expected monitor: lower-quality pleth, respiratory stress pattern, and `SPO2` button alarm pulsing. Around 80% SpO2 should look compromised, around 55% should be weak/noisy but not flat if perfusion exists, and around 10% should be tiny and unreliable.
   - Expected RR: hypoxia should drive tachypnea/hyperventilation unless the expie is progressing into apnea, waterlogging, or critical failure.

3. Press `F9`.
   - Expected log: `Ventricular Instability`.
   - Expected monitor: around `20%` fibrillation, HEART should already show visible irregularity, stronger R waves, and occasional wider ectopic beats without hitting the top/bottom of the monitor. It should then progress into organized VTach with smooth monomorphic wide complexes and a strong inverted dominant deflection, then fade into coarse VFib with chaotic sine-like activity only once VFib is actually developing. The fade into VFib should be continuous, not a snap.
   - Expected vanilla effects: fibrillation progress increases quickly enough to cross the game's warning moodle range, normal ECG becomes unstable.
   - Expected audio: readable organized rhythms, including sinus tachy and organized VTach, should keep heartbeat thumps. Clean vanilla thumps should fade only as VFib becomes chaotic, and should be suppressed in clear VFib-like states.

4. Press `F10`.
   - Expected log: `Respiratory Failure`.
   - Expected vanilla effects: hemothorax/chest pain visibility through existing health panel.
   - Expected monitor: distressed RESP, "shark fin" (slurred upstroke) ETCO2 morphology indicating airway obstruction, and weaker oxygen traces.

5. Press `F11`.
   - Expected log: `PEA (Pulseless Electrical Activity)`.
   - Expected monitor: perfectly normal electrical rhythms (Sinus, Tachycardia) on the HEART channel, but completely flat ABP (blood pressure) and SPO2 (pleth) traces, reflecting no actual mechanical perfusion despite electrical activity.

6. Wait for Asystole (natural death or let PEA progress).
   - Expected monitor: As the expie dies and cardiac arrest sets in, the rhythm should not snap flat. It should decay to a flatline over 1 to 2 seconds.

## Waterlogging Checks

1. Put the expie in water without scuba gear and keep the head/airway in water.
2. Watch `Wet` moodles and stamina.
3. Expected monitor: RESP becomes irregular/waterlogged rather than sinusoidal.
4. Expected monitor: EtCO2 rises and gets a higher plateau as water exposure accumulates.
5. Expected vanilla effects: lowered stamina, reduced oxygen, respiratory strain/hypoventilation if exposure continues.
6. Leave water and allow recovery.
7. Expected monitor: waterlogged pattern and EtCO2 should slowly normalize instead of snapping back instantly.

## Natural Checks

Without debug hotkeys:

- Heavy bleeding should develop `Hypovolemia`, then `Shock`.
- Low oxygen or drowning-like states should develop `Hypoxia`, then `Respiratory Failure`.
- Sustained water exposure without scuba should develop `Waterlogging`, then interact with `Hypoxia` / `Respiratory Failure`.
- High HR plus low oxygen/pressure should develop `Ventricular Instability`.
- Thoracic/internal bleeding should increase respiratory risk and chest pain.

If a condition reaches `Active` or `Critical`, check both the custom monitor and vanilla moodles/panel.

## Alarm Checks

- Medium/high alarms tint the whole monitor instead of drawing a large text banner over the waveform.
- The monitor alarm tint should pulse even when the currently displayed channel is not the offending vital.
- The offending channel button pulses alongside the monitor, for example `SPO2` during hypoxia or `ABP` during hypotension.
- With `PhotosensitiveAlarmMode = true`, pulses should be slower and softer while still readable.
- The alarm tint should be a low-alpha wash, not a full-screen red flash.

## Signal Coupling Checks

- Low SpO2 alone should damage the pleth and drive RESP, but should not flatten ABP while the numeric BP is still strong, for example around `123/82`.
- Change the game simulation timescale, if available, and confirm the monitor sweep and waveform evolution speed up/slow down with the world simulation. ECG complexes should follow simulated HR/rhythm state rather than trying to land exactly on vanilla heartbeat audio.
- ABP should flatten mainly from low pressure, low blood volume, cardiac arrest, or severe ventricular instability.
- In hemorrhagic shock, the ABP pulse pressure should narrow (diastolic stays high while systolic drops) before completely crashing.
- Severe blood volume loss should cause a decoupled temperature drop (hypothermia) due to the lethal triad.
- With a charged worn autopump during heart failure/low pressure, ABP should become highly regular and mechanical: sharp rise, flat synthetic plateau, regular fall. The SPO2 pleth should also adopt a mechanical shape since the pump provides artificial perfusion.
- RESP should advance from the estimated RR and distress state rather than visibly tracking HEART complexes.
- Very low stamina or very low SpO2 should drive tachypnea/hyperventilation unless breathing has stopped, brain health is extremely poor, the expie is critically dying, or severe waterlogging is choking respiration.

## Defibrillator Regression

1. Open the manual defibrillator minigame.
2. Confirm Casualty Vitals replaces the defib ECG with a limited monitor.
3. Confirm only `HEART`, `SPO2`, and `ABP` buttons appear.
4. Confirm `RESP` and `ETCO2` are not selectable on the manual defib.
5. Confirm the vanilla defib ECG is not visible underneath the custom trace.
6. Shock normally and confirm the health panel can still show `???` during `defibShockedFrames`.
7. Confirm the health panel monitor is not stolen by the defib; if both UI surfaces are active, both should keep their own monitor overlay.
8. Close the defib minigame and confirm the normal health panel monitor remains available with all five channels.
9. Shock with AED or manual defib and confirm HEART shows a brief noisy biphasic/spiky artifact. In unchipped mode, only the manual defib monitor needs to show this artifact.
