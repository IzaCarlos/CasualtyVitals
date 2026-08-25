# CasualtyVitals: In-Game Medical & Resuscitation Guide

This guide details all items, monitor waveforms, and condition treatments in **CasualtyVitals** for *Casualties Unknown*.

> [!NOTE]
> This guide applies to **Simple**, **Advanced**, and **Custom** simulation modes. In **ECG** mode, the mod acts as a purely visual monitor enhancement with no extra items, drug loads, or complex physiology states.

---

## Table of Contents
- [1. Mod Medications & Equipment](#1-mod-medications--equipment)
  - [Albuterol Inhaler (`cv_albuterol`)](#albuterol-inhaler-cv_albuterol)
  - [Adrenaline Auto-Injector (`cv_epinephrine`)](#adrenaline-auto-injector-cv_epinephrine)
  - [Atropine Auto-Injector (`cv_atropine`)](#atropine-auto-injector-cv_atropine)
  - [Calcium Gluconate Solution (`cv_calcium_gluconate`)](#calcium-gluconate-solution-cv_calcium_gluconate)
  - [Calcitonin Solution (`cv_calcitonin_solution`)](#calcitonin-solution-cv_calcitonin_solution)
  - [Sodium Bicarbonate Solution (`cv_sodium_bicarbonate`)](#sodium-bicarbonate-solution-cv_sodium_bicarbonate)
  - [Dextrose Chewables (`cv_dextrose`)](#dextrose-chewables-cv_dextrose)
  - [Insulin Vial (`cv_insulin`)](#insulin-vial-cv_insulin)
  - [Potassium Iodide Vial (`cv_potassium_iodide`)](#potassium-iodide-vial-cv_potassium_iodide)
  - [External Pacemaker (`cv_external_pacemaker`)](#external-pacemaker-cv_external_pacemaker)
- [2. In-Game Monitor Traces & Diagnostics](#2-in-game-monitor-traces--diagnostics)
  - [1. HRT (ECG Channel)](#1-hrt-ecg-channel)
  - [2. SPO2 (Pulse Oximetry / Plethysmograph)](#2-spo2-pulse-oximetry--plethysmograph)
  - [3. RESP (Capnography & Respiration)](#3-resp-capnography--respiration)
  - [4. ABP (Arterial Blood Pressure & Murmurs)](#4-abp-arterial-blood-pressure--murmurs)
  - [5. ETCO2 (End-Tidal CO2)](#5-etco2-end-tidal-co2)
- [3. Core Physiological Concepts: How to Read In-Game Tells](#3-core-physiological-concepts-how-to-read-in-game-tells)
  - [1. PaO2 (Arterial Oxygen Partial Pressure)](#1-pao2-arterial-oxygen-partial-pressure)
  - [2. PaCO2 (Arterial Carbon Dioxide Partial Pressure)](#2-paco2-arterial-carbon-dioxide-partial-pressure)
  - [3. Acidosis (Blood pH Balance)](#3-acidosis-blood-ph-balance)
  - [4. Ischemia Burden & STEMI (Heart Oxygen Balance)](#4-ischemia-burden--stemi-heart-oxygen-balance)
  - [5. Electrolyte Derangements (Potassium & Calcium)](#5-electrolyte-derangements-potassium--calcium)
- [4. Congenital & Latent Phenotypes](#4-congenital--latent-phenotypes)
  - [Wolff-Parkinson-White (WPW)](#wolff-parkinson-white-wpw)
  - [Congenital Long QT Syndrome](#congenital-long-qt-syndrome)
  - [Bundle Branch Blocks (RBBB & LBBB)](#bundle-branch-blocks-rbbb--lbbb)
- [5. Resuscitation & Clinical Protocols](#5-resuscitation--clinical-protocols)
  - [Handling Cardiac Arrest & Fibrillation](#handling-cardiac-arrest--fibrillation)
  - [Treating Hyperkalemia & Electrolyte Emergencies](#treating-hyperkalemia--electrolyte-emergencies)
  - [Managing Long QT & Torsades de Pointes (TdP)](#managing-long-qt--torsades-de-pointes-tdp)
  - [Coronary Ischemia, Angina Burden & Heart Attacks](#coronary-ischemia-angina-burden--heart-attacks)
  - [Managing Conduction Blocks (AV Blocks & Pacing)](#managing-conduction-blocks-av-blocks--pacing)
  - [Treating Severe Acidosis & Kussmaul Respiration](#treating-severe-acidosis--kussmaul-respiration)
  - [Recovering from Drug Overdoses](#recovering-from-drug-overdoses)
- [6. Dynamic Hemodynamics & Bleeding Model](#6-dynamic-hemodynamics--bleeding-model)
  - [1. Darcy-Weisbach Pressure Flow](#1-darcy-weisbach-pressure-flow)
  - [2. Blood Volume Easing & Hypervolemia](#2-blood-volume-easing--hypervolemia)
  - [3. Anatomical Vascular Caliber Matrix](#3-anatomical-vascular-caliber-matrix)
  - [4. WoundView & HUD Integration](#4-woundview--hud-integration)

---

## 1. Mod Medications & Equipment

### Albuterol Inhaler (`cv_albuterol`)
- **Type**: 10-puff metered-dose inhaler.
- **When to Use**:
  - Airway obstruction, wheezing, waterlogged lungs, or asthma attacks.
  - Mild hyperkalemia (shifts potassium into cells at $-0.10\text{ mmol/L}$ per puff).
- **In-Game Effect (1–2 Puffs)**:
  - Clears bronchospasm and flattens sloped "shark fin" $\text{EtCO}_2$ capnography waves back to square plateaus.
  - Restores obstructed breathing rate toward normal.
- **Overdose Hazard (6–10 Puffs in rapid succession)**:
  - Triggers severe tachypneic hyperventilation ($> 50\text{ breaths/min}$ / raw RR $> 200$).
  - Surges heart rate ($+40\text{--}80\text{ bpm}$) into extreme tachycardia.
  - Drops diastolic blood pressure (e.g. down to $120/40\text{ mmHg}$) and drains stamina, inducing lactic acidosis.

---

### Adrenaline Auto-Injector (`cv_epinephrine`)
- **Type**: Single-use auto-injector loaded with a **120ml multi-compound cocktail** (applied to any limb).
- **Compound Breakdown (120ml Total)**:
  - **Adrenaline Solution (60ml)**: Golden amber. Concentrated adrenergic agonist ($\alpha/\beta$) driving inotropic cardiac output.
  - **Noradrenaline Solution (35ml)**: Rose crimson. Potent peripheral vasoconstrictor restoring systemic vascular resistance.
  - **Isotonic Carrier Solution (25ml)**: Phosphor cyan. Buffered vehicle ensuring rapid, stabilized intramuscular uptake.
- **When to Use**:
  - Severe shock / collapsing blood pressure ($\text{Systolic BP} < 60\text{ mmHg}$).
  - Sluggish heart rate or severe symptomatic bradycardia ($\text{HR} < 40\text{ bpm}$) when atropine is ineffective.
  - Substantial hemodynamic pressor support following resuscitation / trauma.
- **In-Game Effect & Dynamics**:
  - **Gradual Absorption Onset**: Smoothly ramps up over $3\text{--}5$ seconds into a sustained therapeutic plateau.
  - **Hemodynamic Boost**: Delivers $+36\text{ bpm}$ inotropic rate support, $+28\text{ mmHg}$ systolic pressure, and $+18\text{ mmHg}$ diastolic vascular tone for $35\text{--}65\text{ seconds}$ with a smooth exponential metabolic washout.
  - **Respiratory Assist**: Opens airways and gently supports depressed breathing drive.
- **Important Distinction from Vanilla Combat Stimulant Pen**:
  - The Adrenaline Auto-Injector is a pure medical vasopressor/inotropic formulation. **It does NOT contain the military stimulant cocktail found in the vanilla Combat Stimulant Pen** that vanilla checks to restart a completely arrested heart. If a patient experiences total cardiac failure in vanilla, use a **Combat Stimulant Pen** + CPR to restart cardiac activity, then use the **Adrenaline Auto-Injector** to sustain hemodynamics.
- **Overdose Hazard**:
  - Stacking multiple pens causes malignant hypertension ($\text{BP} > 200\text{ mmHg}$), tachycardia ($> 160\text{ bpm}$), and ischemic ST-elevation demand spikes.

---

### Atropine Auto-Injector (`cv_atropine`)
- **Type**: Spring-loaded **Mark I NAAK style dual auto-injector** (containing 7ml Atropine + 13ml Pralidoxime) for limbs.
- **When to Use**:
  - Sluggish heart rate / sinus bradycardia ($\text{HR} < 45\text{ bpm}$).
  - First-Degree AV Block and Second-Degree Type I AV Block (Wenckebach).
  - **Neurotoxic Venom Envenomation & Nerve Agent Toxicity**: Creature venom attacks and chemical organophosphate poisoning.
- **In-Game Effects**:
  - **Cardiac Conduction**: Blocks vagal parasympathetic tone, accelerates heart rate ($+18\text{ bpm}$), and clears nodal conduction delays/Wenckebach blocks.
  - **Secondary Antidote Action**: The **Pralidoxime (2-PAM)** component reactivates inhibited acetylcholinesterase, rapidly driving down **`venomCurrent` ($-45$)** and purging toxic sickness ($-20$) while atropine prevents lethal parasympathetic bronchospasm and cardiovascular collapse.
- **Limitation**:
  - Does not fix high-grade infranodal blocks (Mobitz II or 3rd Degree Complete Heart Block); use the **External Pacemaker** for those.

---

### Calcium Gluconate Solution (`cv_calcium_gluconate`)
- **Type**: 100ml reusable medical glass bottle (drinkable or injectable via syringe). Persists when empty (`destroyAtZeroCondition = false`).
- **When to Use**:
  - **Dangerous Hyperkalemia ($K^+ > 6.5\text{ mmol/L}$)**: Tall peaked T-waves, broadened QRS, or sine-wave rhythms on the ECG monitor.
  - **Torsades de Pointes / Long QT Crisis**: Membrane stabilization during polymorphic ventricular crisis.
- **In-Game Effect**:
  - Directly stabilizes cardiac cell membranes against potassium toxicity without altering serum potassium level, immediately narrowing the QRS and preventing fatal VFib/asystole progression.
- **Overdose Hazard**:
  - Chugging multiple bottles causes calcium overload / Hypercalcemia ($Ca^{2+} > 13\text{ mg/dL}$), which shortens the QT interval and increases myocardial irritability. Use **Calcitonin Solution** to lower calcium if overdosed.

---

### Calcitonin Solution (`cv_calcitonin_solution`)
- **Type**: 100ml reusable medical glass bottle (drinkable or injectable via syringe). Persists when empty.
- **When to Use**:
  - Hypercalcemia crisis ($Ca^{2+} > 12.0\text{ mg/dL}$) or calcium gluconate overdose.
- **In-Game Effect**:
  - Safely brings down elevated serum calcium levels.

---

### Sodium Bicarbonate Solution (`cv_sodium_bicarbonate`)
- **Type**: 80ml reusable glass bottle.
- **When to Use**:
  - Severe metabolic acidosis ($\text{Acidosis} > 0.30$, rapid acidotic breathing, low pH).
  - Hyperkalemic acidosis (helps shift potassium into cells).
  - Prolonged cardiac arrest with accumulated lactic acid.
- **In-Game Effect**:
  - Buffers blood acidity, eases acidotic hyperventilation (Kussmaul breathing), and accelerates potassium clearing.

---

### Dextrose Chewables (`cv_dextrose`)
- **Type**: Pack of chewable glucose tablets.
- **When to Use**:
  - Stamina exhaustion / negative stamina drain ($< 0$).
  - Hypoglycemia, especially when using insulin.
- **In-Game Effect**:
  - Rapidly restores stamina pool and reverses fatigue/tremor.

---

### Insulin Vial (`cv_insulin`)
- **Type**: 50ml reusable injectable vial (requires syringe).
- **When to Use**:
  - Rapidly lowering dangerously elevated potassium ($K^+ > 6.0\text{ mmol/L}$).
- **In-Game Effect**:
  - Strongly drives extracellular potassium into cells. Always pair with **Dextrose Chewables** to prevent stamina collapse.

---

### Potassium Iodide Vial (`cv_potassium_iodide`)
- **Type**: 100ml reusable vial (oral or injection).
- **When to Use**:
  - Radiation sickness ($> 0\%$).
- **In-Game Effect**:
  - Accelerates logarithmic clearance of internal radiation burden.

---

### External Pacemaker (`cv_external_pacemaker`)
- **Type**: Wearable electronic pacing device equipped to the chest (`UpTorso` / `outertorso`).
- **Battery System**: Utilizes replaceable **Small Batteries** (`BatteryProperties.Small`). Lasts a full **360 seconds (6 minutes)** of continuous pacing per battery charge.
- **When to Use**:
  - Third-Degree Complete Heart Block (CHB) with severe bradycardia ($\text{HR} \approx 30\text{ bpm}$).
  - Mobitz II high-grade AV blocks.
  - Profound nodal arrest / P-wave asystole.
- **In-Game Effect & Progressive Healing**:
  - Paces ventricular contractions firmly at $74\text{ bpm}$ with blood pressure support ($92+\text{ mmHg}$).
  - **AV Block Progressive Healing**: Over the course of 1 battery charge, the therapeutic electrical stimulation progressively steps down AV blocks ($\text{3rd Degree} \to \text{Mobitz II} \to \text{Wenckebach} \to \text{1st Degree} \to \text{Normal Sinus Rhythm}$), fully curing the conduction system by **10% battery charge remaining** (36s of reserve battery to spare!).
- **TACTICAL HAZARDS & WARNINGS**:
> [!WARNING]
> **Chest Slot / Armor Incompatibility**: The device occupies the chest equipment slot (`UpTorso`), meaning heavy ballistic vests and plate carriers **cannot be worn** while the pacemaker is active.

> [!CAUTION]
> **Rate-Fixation & Exertion Hazards (Thornback Encounters / Combat)**: Because ventricular pacing is fixed at $74\text{ bpm}$, cardiac output cannot accelerate to meet extreme oxygen demands. Heavy physical exertion (such as sprinting, fleeing from Thornbacks, or intense combat) causes severe stamina exhaustion and **rapidly drains consciousness**, causing **vision narrowing, darkening, and blurring**. Stop running immediately to rest and regain consciousness before passing out completely.

---

## 2. In-Game Monitor Traces & Diagnostics

The in-game vital monitor renders 5 distinct physiological channels:

### 1. HRT (ECG Channel)
- **Normal Sinus Rhythm**: Rate 60–100 bpm with narrow QRS complexes and normal T waves.
- **Peaked "Tented" T-Waves**: Indicates **Hyperkalemia** ($K^+ > 5.4\text{ mmol/L}$). If left untreated, progresses into broad QRS complexes and a fatal **Sine-Wave** pattern ($K^+ > 7.5$). *Treat with Calcium Gluconate immediately, followed by Albuterol and Sodium Bicarbonate.*
- **Short QT Interval**: Indicates **Hypercalcemia** ($Ca^{2+} > 12.0\text{ mg/dL}$). *Treat with Calcitonin.*
- **ST Elevation (STEMI / Heart Attack)**: Indicates severe coronary occlusion / transmural myocardial ischemia.
  - *Lead II/III/aVF*: Inferior STEMI (RCA).
  - *Lead I/aVL/V1–V4*: Anterior Septal STEMI (LAD).
- **AV Conduction Blocks**:
  - *1st Degree / Wenckebach*: PR delay or progressive lengthening before a dropped beat. *Treat with Atropine.*
  - *Mobitz II / 3rd Degree (CHB)*: Intermittent dropped beats or complete P-wave dissociation with slow escape rate ($\sim 30\text{ bpm}$). *Apply External Pacemaker.*
- **Torsades de Pointes (TdP)**: Twisting polymorphic ventricular tachycardia with a crescendo-decrescendo spindle envelope. *Requires unsynchronized high-energy defibrillation and Calcium Gluconate.*
- **Ventricular Fibrillation (VFib)**: Completely chaotic, irregular baseline with no pulse. *Defibrillate immediately to reduce fibrillation progress.*
- **Asystole**: Flatline. *Administer Epinephrine and perform CPR.*

### 2. SPO2 (Pulse Oximetry / Plethysmograph)
- **Normal Trace**: Sharp pulse peaks with a distinct dicrotic notch.
- **Low Amplitude / Flatline**: Severe blood loss / hypovolemia ($\text{Blood Volume} < 3.0\text{ dL}$), profound shock, or peripheral vasoconstriction.

### 3. RESP (Capnography & Respiration)
- **Eupnea**: Normal regular breathing rate ($12\text{--}20\text{ breaths/min}$).
- **Cheyne-Stokes Respiration**: Waxing-and-waning crescendo-decrescendo breathing with apnea pauses. Indicates severe heart failure or brain hypoperfusion.
- **Kussmaul Breathing**: Deep, rapid, sustained hyperventilation ($> 30\text{ breaths/min}$). Indicates severe metabolic acidosis ($\text{Acidosis} > 0.40$). *Treat with Sodium Bicarbonate.*
- **Agonal Gasps**: Wide, slow, spasmodic gasps ($< 6\text{ breaths/min}$). Indicates imminent cardiac arrest.

### 4. ABP (Arterial Blood Pressure & Murmurs)
- **Normal ($120/80\text{ mmHg}$)**: Systolic peak at 120, diastolic trough at 80 (Pulse Pressure = 40).
- **Narrow Pulse Pressure (e.g. $100/85\text{ mmHg}$)**: Vasoconstriction, hypovolemic shock compensation, or blood loss.
- **Widened Pulse Pressure (e.g. $130/40\text{ mmHg}$)**: Severe vasodilation from septic shock, albuterol overdose, or aortic regurgitation.
- **Heart Murmur Pressure Modulation**:
  - *Systolic Murmur (Aortic Stenosis)*: Characteristic *pulsus parvus et tardus* (delayed systolic rise with turbulent upstroke shudder).
  - *Diastolic Murmur (Aortic Regurgitation)*: Rapid bounding upstroke with collapsing diastolic runoff.

### 5. ETCO2 (End-Tidal CO2)
- **Normal (35–45 mmHg)**: Crisp square-wave plateau.
- **Shark-Fin Waveform**: Sloped expiratory phase indicating asthma, waterlogged airway, or bronchospasm. *Treat with Albuterol Inhaler.*
- **Sudden Rise during CPR ($> 30\text{ mmHg}$)**: Indicates return of spontaneous cardiac circulation (ROSC).

---

## 3. Core Physiological Concepts: How to Read In-Game Tells

CasualtyVitals tracks blood gas dynamics, acid-base equilibrium, and coronary oxygen supply. These values directly reflect in-game monitor readings, waveform shapes, and character physical status.

### 1. $\text{PaO}_2$ (Arterial Oxygen Partial Pressure)
- **What it is**: The physical oxygen gas dissolved in arterial blood (normal: $80\text{--}105\text{ mmHg}$).
- **How to derive it in-game**:
  - **$\text{SpO}_2\ 98\text{--}100\%$**: Normal $\text{PaO}_2$ ($85\text{--}105\text{ mmHg}$). Full cerebral oxygenation and tissue delivery.
  - **$\text{SpO}_2\ 88\text{--}92\%$**: Moderate Hypoxemia ($\text{PaO}_2 \approx 55\text{--}65\text{ mmHg}$). Compensatory tachycardia begins.
  - **$\text{SpO}_2\ 70\text{--}80\%$**: Severe Hypoxia ($\text{PaO}_2 \approx 38\text{--}48\text{ mmHg}$). Triggers vision darkening, gasping, and elevated heart attack (STEMI) risk.
  - **$\text{SpO}_2 < 60\%$**: Critical Hypoxia ($\text{PaO}_2 < 35\text{ mmHg}$). Brain health drains rapidly, leading to loss of consciousness and cardiac arrest.
- **Action**: Provide airway ventilation, clear water from lungs, or equip scuba diving gear.

---

### 2. $\text{PaCO}_2$ (Arterial Carbon Dioxide Partial Pressure)
- **What it is**: Carbon dioxide gas dissolved in blood (normal: $35\text{--}45\text{ mmHg}$).
- **How to derive it in-game**:
  - **Normal Breathing ($\text{RR } 12\text{--}20\text{ bpm},\ \text{EtCO}_2\ 35\text{--}45\text{ mmHg}$)**: $\text{PaCO}_2$ sits at roughly $\text{EtCO}_2 + 4\text{ mmHg}$ ($\sim 40\text{ mmHg}$).
  - **Hypoventilation / Opioid Overdose ($\text{RR} < 10\text{ bpm}$, slow/shallow waves)**: $\text{CO}_2$ is trapped in the body. $\text{PaCO}_2$ climbs to $55\text{--}80\text{ mmHg}$, generating **Respiratory Acidosis**.
  - **Hyperventilation / Rapid Breathing ($\text{RR} > 25\text{ bpm}$)**: Patient blows off $\text{CO}_2$ rapidly. $\text{EtCO}_2$ drops to $20\text{--}28\text{ mmHg}$, lowering $\text{PaCO}_2$ to $20\text{--}30\text{ mmHg}$ (**Respiratory Compensation**).
  - **Shark-Fin Waveform on $\text{EtCO}_2$**: Indicates bronchospasm or blocked airway. $\text{CO}_2$ cannot exit properly, driving $\text{PaCO}_2$ upward.
- **Action**: Use Albuterol Inhaler for bronchospasm, or support respiratory rate if depressed.

---

### 3. Acidosis (Blood pH Balance)
- **What it is**: Dangerous accumulation of acid in the bloodstream caused by trapped $\text{CO}_2$ (Respiratory Acidosis), poor perfusion / lack of oxygen (Lactic Acidosis), or ketone buildup (Diabetic Ketoacidosis).
- **How to identify it in-game**:
  - **Kussmaul Breathing on RESP Trace**: The character enters rapid, deep, non-stop hyperpnea ($\text{RR} > 24\text{--}40\text{ bpm}$) to blow off acid as $\text{CO}_2$.
  - **Low Blood Pressure (ABP Trace)**: Systolic $\text{BP} < 75\text{ mmHg}$ or low blood volume ($< 2.5\text{ dL}$) starves tissues of oxygen, producing lactic acidosis.
  - **High Ketones (Type 1 Diabetes Mod)**: Ketones $> 1.2\text{ mmol/L}$ directly generate metabolic ketoacidosis.
  - **ECG Distortion**: High acidosis depresses heart contractility and promotes potassium leakage.
- **Action**: Drink or inject **Sodium Bicarbonate Solution** to buffer systemic acid and normalize breathing rate.

---

### 4. Ischemia Burden & STEMI (Heart Oxygen Balance)
- **What it is**: Heart muscle starving for oxygen due to high demand vs low supply.
- **How to identify it in-game**:
  - **Supply Failure**: $\text{SpO}_2 < 92\%$ or Diastolic $\text{BP} < 65\text{ mmHg}$ (coronary arteries perfuse during diastole).
  - **High Demand**: Heart Rate $> 110\text{--}160\text{ bpm}$ or Systolic $\text{BP} > 140\text{ mmHg}$.
  - **ECG ST-Elevation**: The flat segment between S and T lifts high above the baseline (Anterior STEMI in Lead I/V1–V4, Inferior STEMI in Lead II/III/aVF).
  - **Torso Pain**: Intense chest pain (Angina) that worsens under movement.
  - **PVCs**: Stray rogue wide beats appearing on the ECG.
- **Action**: Rest to drop heart rate demand, restore oxygen, manage blood pressure, and use Epinephrine if in cardiogenic shock.

---

### 5. Electrolyte Derangements (Potassium & Calcium)
- **Potassium ($K^+$)**:
  - **Tall Peaked T-Waves**: Indicates $K^+ > 5.4\text{ mmol/L}$ (eating bananas, kidney stress, DKA).
  - **Widened QRS / Sine Wave**: Critical Hyperkalemia ($K^+ > 7.5\text{ mmol/L}$). High risk of instant arrest.
  - **Action**: Administer Calcium Gluconate (immediate membrane shield), then Insulin + Dextrose or Albuterol to clear $K^+$.
- **Calcium ($Ca^{2+}$)**:
  - **Shortened QT Interval**: T-wave hugs the QRS complex directly ($Ca^{2+} > 12.0\text{ mg/dL}$).
  - **Action**: Administer Calcitonin Solution to lower calcium levels.

---

## 4. Congenital & Latent Phenotypes

When starting a round on Advanced mode, characters have a realistic background chance ($\sim 5.5\%$) of possessing an innate cardiac phenotype:

### Wolff-Parkinson-White (WPW)
- **Characteristics**: Congenital accessory pathway (Bundle of Kent) bypassing the AV node.
- **Visual Presentation**: Distinct **Delta wave** (slurred upstroke on the QRS complex) with a shortened PR interval.
- **Clinical Danger**: Under physical exertion, panic, or stimulant injection (Epi/Atropine), conduction accelerates down the accessory pathway, triggering acute pre-excited supraventricular tachycardia (SVT).

### Congenital Long QT Syndrome
- **Characteristics**: Delayed ventricular repolarization stretching the QT interval.
- **Visual Presentation**: Extended distance between QRS completion and T-wave termination.
- **Clinical Danger**: High susceptibility to the **R-on-T phenomenon**. If combined with hypokalemia, dehydration, or tachycardia, it triggers **Torsades de Pointes (TdP)**.

### Bundle Branch Blocks (RBBB & LBBB)
- **RBBB**: Delayed right ventricular activation, presenting with classic **M-shaped "Rabbit Ears" ($rsR'$)** in right precordial vectors.
- **LBBB**: Broad notched QRS complexes with discordant ST-T waves in lateral leads. Masks acute STEMI patterns.

---

## 5. Resuscitation & Clinical Protocols

### Handling Cardiac Arrest & Fibrillation
1. **Identify Rhythm on Monitor**:
   - **$55\%\text{--}87\%$ Fibrillation**: Monomorphic or Polymorphic VTach (or TdP in Long QT). Ventricles still produce an organized rapid electrical rate ($180\text{--}240\text{ bpm}$) with readable hypotensive blood pressure.
   - **$\ge 88\%$ Fibrillation (True VFib)**: Unorganized coarse/fine chaotic undulations. Mechanical perfusion ceases; heart rate blanks to `---` and blood pressure displays `---/---`.
2. **Defibrillation & Paddle Placement**:
   - Apply defibrillator paddles **directly to the Torso** (paddles on arms/legs produce heavily attenuated signals and will fail to deliver an effective transthoracic shock).
   - Deliver defibrillation shocks scaled to the estimated power range (`POW.EST`) to drive `fibrillationProgress` back down.
3. **Restoring Circulation & Pulse Acquisition**:
   - Once fibrillation falls **below $78\%$**, the monitor enters a **1.4-second recalculation sequence**, smoothly ramping displayed numbers from zero up to the patient's restored physiological vitals.
   - If cardiac arrest has occurred in vanilla, use a vanilla **Combat Stimulant Pen** and perform CPR to restart cardiac pacing.
   - Administer the **Adrenaline Auto-Injector (`cv_epinephrine`)** to raise sluggish post-arrest heart rate ($+36\text{ bpm}$) and sustain systemic arterial pressure ($+28/18\text{ mmHg}$).
4. **Mechanical Support**: Engage the Auto-Pump or maintain CPR compressions to sustain cerebral perfusion while medications take effect.

---

### Treating Hyperkalemia & Electrolyte Emergencies
1. **Detection**: Tall peaked "tented" T-waves, PR prolongation, and widening QRS sine-waves on the monitor.
2. **Step 1 (Membrane Stabilization)**: Immediately administer **Calcium Gluconate Solution** (oral or syringe). This stabilizes myocardial membranes and prevents immediate VFib/asystole without altering blood potassium level.
3. **Step 2 (Intracellular Shift)**: Take 1–2 puffs of **Albuterol Inhaler** and drink **Sodium Bicarbonate Solution** (or inject **Insulin** + eat **Dextrose Chewables**) to drive potassium into cells.

---

### Managing Long QT & Torsades de Pointes (TdP)
1. **Detection**: Wide polymorphic ventricular tachycardia with a twisting spindle-and-node amplitude envelope across the baseline.
2. **High-Energy Defibrillation**: TdP cannot be synchronized; deliver an **unsynchronized high-energy defibrillator shock** (charge set high on the manual defibrillator or AED) to depolarize the myocardium.
3. **Stabilization**: Administer **Calcium Gluconate Solution** to stabilize cardiac membranes and prevent recurrent TdP degeneration.

---

### Coronary Ischemia, Angina Burden & Heart Attacks
1. **Detection**: Progressive ST segment elevation or deep depression on the ECG, accompanied by severe chest pain.
2. **Prolonged Angina Progression (100–120s Window & Diminishing Die Roll)**:
   - Sustained severe ischemia ($\text{Ischemia Burden} > 0.65$) progresses into a regionalized **STEMI** (Anterior or Inferior).
   - While severe untreated angina persists, a **100–120 second decompensation window** runs in the background.
   - **Diminishing Die Roll Mechanic**: Every second, the engine rolls an ever-decreasing face die ($N = 120 - t$, diminishing from a 120-sided die down to a 1-sided die) against a randomly picked target number ($P = \frac{1}{N}$). As time elapses without treatment, the risk of sudden acute pump failure and **terminal asystole** steadily escalates until guaranteed collapse at 120 seconds.
   - Resolving the ischemia (resting, restoring oxygenation, relieving hypotension) allows the angina timer to cool down and safely reset.
3. **Management**: Relieve physical exertion, restore oxygenation ($\text{SpO}_2 > 90\%$), and manage severe arterial hypotension.

---

### Managing Conduction Blocks (AV Blocks & Pacing)
1. **Low-Grade Blocks ($1^{\text{st}}\text{ Degree}$ & Wenckebach)**:
   - PR prolongation or dropped beats resulting from vagal excess or mild nodal ischemia.
   - **Treatment**: Administer **Atropine Auto-Injector (`cv_atropine`)** to block parasympathetic tone and restore 1:1 AV conduction.
2. **High-Grade Blocks (Mobitz II & $3^{\text{rd}}\text{ Degree Complete Heart Block}$)**:
   - Infranodal conduction failure with severe bradycardia ($\sim 30\text{ bpm}$) and pulse collapse.
   - **Treatment**: Equip the **External Pacemaker (`cv_external_pacemaker`)** to the chest (`UpTorso`). Ensure a Small Battery is installed. The device maintains continuous ventricular pacing at $74\text{ bpm}$ and progressively repairs the conduction pathway over its **360-second battery life**, curing the heart block with **10% battery remaining**. Beware that armor cannot be worn while pacing and sprinting causes rapid fatigue.

---

### Treating Severe Acidosis & Kussmaul Respiration
1. **Detection**: Rapid deep Kussmaul hyperpnea on the RESP channel, low arterial blood pH, or prolonged shock state.
2. **Treatment**: Drink or inject **Sodium Bicarbonate Solution** to buffer systemic acidity and normalize breathing drive.

---

### Recovering from Drug Overdoses
- **Albuterol Overdose ($6\text{--}10\text{ puffs}$)**: Severe tachycardia, tachypnea ($> 50\text{ breaths/min}$), and negative stamina drain. Buffer lactic acidosis with **Sodium Bicarbonate** and restore stamina with **Dextrose Chewables**.
- **Adrenaline Overdose (Stacked Pens)**: Malignant hypertension ($> 200\text{ mmHg}$) and tachyarrhythmias. Rest and monitor until drug washout ($\approx 45\text{--}60\text{ seconds}$).
- **Calcium Overdose**: Short QT intervals and myocardial irritability. Administer **Calcitonin Solution** to safely lower serum calcium.

---

## 6. Dynamic Hemodynamics & Bleeding Model

When running in **Advanced** mode (or with **Hemodynamics** enabled in **Custom** mode), wound bleeding rates dynamically scale based on arterial blood pressure, circulating blood volume reserves, and localized vascular caliber.

### 1. Darcy-Weisbach Pressure Flow
Wound outflow scales with the square root of effective arterial perfusion pressure ($\sqrt{\text{BP} / 120}$):
- **Hypertensive Crisis ($180\text{ mmHg}$)**: $\approx 1.22\times$ outflow acceleration.
- **Normal Pressure ($120\text{ mmHg}$)**: $1.00\times$ baseline outflow.
- **Decompensated Shock ($60\text{ mmHg}$)**: Outflow naturally slows to $\approx 0.71\times$.
- **Asystole / Cardiac Arrest ($0\text{ mmHg}$)**: Outflow drops to $0.05\times$ (passive capillary ooze).
- **AutoPump Support during Arrest**: Generates artificial mechanical perfusion ($\sim 55\text{ mmHg}$ / $\approx 0.68\times$), maintaining blood flow to the brain while sustaining bleeding until wounds are treated.

### 2. Blood Volume Easing & Hypervolemia
- Outflow scales relative to total circulating blood liters ($V_{\text{liters}} = 2.5\text{L} + \text{bloodVolume} \times 0.025\text{L}$):
  - **$7.5\text{L}$ ($150\%$ IV Overload)**: $1.35\times$
  - **$5.5\text{L}$ ($110\%$ Hypervolemic)**: $1.07\times$
  - **$5.0\text{L}$ ($100\%$ Healthy Adult)**: $1.00\times$
  - **$4.23\text{L}$ ($85\%$ Mild Volume Loss)**: $\approx 0.89\times$
  - **$2.5\text{L}$ ($50\%$ Lethal Shock Threshold)**: $0.65\times$
- This creates a natural physiological buffer window: as blood is lost, bleeding automatically decelerates, giving the player more time to apply tourniquets or bandages.

### 3. Anatomical Vascular Caliber Matrix
Each limb scales according to its major arterial caliber:
- **Upper Torso (Aortic Arch / Chest)**: $1.20\times$
- **Head (Common Carotid)**: $1.15\times$
- **Lower Torso (Abdominal Aorta / Iliac)**: $1.15\times$
- **Thighs (Femoral Artery)**: $1.15\times$
- **Upper Arms (Brachial Artery)**: $1.00\times$ (baseline)
- **Forearms & Calves (Radial & Tibial)**: $0.85\times$
- **Paws & Feet**: $0.70\times$

### 4. WoundView & HUD Integration
- Hovering over or selecting any limb in the **WoundView** displays the real-time gross flow rate in $\text{L/m}$ on the limb card.
- Droplet severity indicators on the paper doll dynamically shift color and tiers based on real-time hemodynamic flow.
- The top-panel Total Bleed readout accurately reflects the sum of all active wound outflows.

