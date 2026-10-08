# Harmony Reach — Eye Twin Simulator

A working prototype of a mobile eye-screening service with three connected roles: an **operator** in a screening van takes an eye photo, a **doctor** reviews a 3D digital twin of that eye remotely, and the **patient** follows their result with a visit code.

Built as part of the Topcon Healthcare case in the Product Development course at the University of Oulu (2026). It shows the "digital twin at the doctor's side" idea from our service concept as something people can actually click through.

> **Concept prototype, not a medical device.** The 3D eye is a visual model built from a photo, and the demo's OCT values are simulated. It does not diagnose anything. Harmony Reach is a student concept and not an official Topcon product.

**Live demo:** [icberg.github.io/harmony-reach-eye-twin/?v=3#operator](https://icberg.github.io/harmony-reach-eye-twin/?v=3#operator)

Jump straight to a role: [Operator](https://icberg.github.io/harmony-reach-eye-twin/#operator) · [Doctor](https://icberg.github.io/harmony-reach-eye-twin/#doctor) · [Patient](https://icberg.github.io/harmony-reach-eye-twin/#patient)

![Doctor view: 3D macula twin built from OCT values, ETDRS thickness grid, centre-thickness trend and the proposed care plan](docs/screenshot.png)

### Try it in two minutes

1. **Doctor:** press **Add a sample patient (3 scans)**. The newest scan opens with the 3D macula, the ETDRS grid and the thickness trend. Choose a retinopathy grade (e.g. *Mild NPDR*), read the proposed plan, and press **Approve plan and send to patient**.
2. **Operator:** the patient now appears on the **Recall list** with their due date. Press **Start re-screen**, tap **Simulate scan** in the OCT step, and **Send visit to specialist**. Note the visit code, e.g. `HR-4K7QD`.
3. **Patient:** enter a visit code and press **Open** to see the timeline, the plain-language result and your own 3D eye.

Stay in the same browser tab the whole time. On GitHub Pages, visits live only in that open page, so a reload clears them (see [Two ways it runs](#two-ways-it-runs)).

## The flow

| Role | What they do |
|---|---|
| **Operator** (van driver trained as operator) | Works through the recall list → fills in a short intake form → takes or uploads an eye photo → taps the pupil, or lets the page auto-centre it → checks image quality → optionally adds a fundus (retina) photo and OCT macula values → sends. Gets a visit code like `HR-4K7QD`. |
| **Doctor** (remote specialist) | Sees a queue sorted urgent → waiting → reviewed. Opens a visit and rotates a 3D eye whose iris is the patient's real photo, a 3D retina, and a 3D macula built from OCT thickness. Compares with earlier visits. Grades the photo, reviews the twin's proposed care plan, and approves or changes it. |
| **Patient** | Enters the visit code and sees a timeline (captured → in review → result), a plain-language result with the next check date, and their own 3D eye. |

## What makes it a twin (v2)

Using the model / shadow / twin levels from Kritzinger et al. (2018), v1 was a visual *digital shadow*. v2 adds the parts a twin needs:

| Twin requirement | How v2 does it |
|---|---|
| **Built from measurements** | The macula surface comes from OCT thickness values on the standard ETDRS 9-sector grid (centre, inner and outer rings, in µm), interpolated over the 6 mm area. Height and colour both come from the numbers, not from how the photo looks. |
| **One model that evolves** | Each eye has a twin record (`twins/<patient>-<eye>`). Every visit adds a scan to it. The doctor sees the change per sector (for example "+19 µm"), a change-coloured map, a centre-thickness trend over time, and the 3D surface morphing between visits. Scans from different OCT devices aren't compared, because their thickness scales differ. |
| **Simulation / decision support** | A plan engine combines the doctor's retinopathy grade with the OCT data:<br>• **ICO Guidelines for Diabetic Eye Care (2017):** re-examination and referral intervals for high- and low-resource settings.<br>• **DRCR Retina Network thresholds** for centre-involved macular oedema (Cirrus ≥290 µm F / ≥305 µm M; Spectralis ≥305 F / ≥320 M).<br>• **The twin's own trend rules,** labelled *not validated*: a change of 10% or more since the last scan, and a linear projection of when the threshold would be crossed. |
| **Closed loop** | The doctor approves the proposed plan or changes it (a reason is required). The approved plan is saved to the twin and shown to the patient. The patient then appears on the van operator's **recall list** on their due date. "Start re-screen" fills in the intake, and the new scan updates the same twin. |

The demo's OCT values are simulated, and a real deployment would read them from the OCT device. The trend rules are illustrations and have not been validated.

## Features

- **3D eye twin (Three.js):** sclera sphere with a generated vein texture, an iris cap textured from the cropped photo, a transparent cornea and an optic-nerve stub. Orbit, zoom and auto-rotate.
- **3D retina view:** the fundus photo's brightness becomes a height map on a 160 × 160 mesh, with a foveal pit. If there's no fundus photo, a generated one is used.
- **3D macula view:** a surface built from the nine OCT thickness values (height ×4), with the ETDRS rings drawn on it, coloured by thickness or by change since the last scan.
- **Twin measurements:** ETDRS grid with per-sector change, and a centre-thickness trend chart with the DRCR threshold and the twin's projection.
- **Care-plan engine:** guideline-based proposal with every rule's source shown; overrides need a reason; approved plans feed the operator's recall list.
- **Image-quality check in the browser:** sharpness (Laplacian variance), brightness, glare and how well the eye is centred, all calculated from canvas pixels, so the photo never has to leave the device for this step.
- **Pupil finding:** tap or drag to place the iris ring, a slider for its size, or auto-centre on the darkest region.
- **Sample data:** a "Sample eye" generator, a "Simulate scan" button for OCT, and an "Add a sample patient (3 scans)" button, so you can demo without real photos or devices.
- **Role tabs:** Operator, Doctor and Patient switch instantly within one page, and the URL follows along (`#operator`, `#doctor`, `#patient`), so you can share a link straight to one role.
- Light and dark theme, responsive down to phone width.

## Two ways it runs

The page has a small data layer that picks a backend when it starts:

| Mode | Where | What works |
|---|---|---|
| **Single device** | Any browser (GitHub Pages, opening the file locally) | Everything, inside one open page. Switch roles with the tabs at the top. Data is kept in memory and cleared on reload, so a visit code from another device or an earlier session shows "No visit found". |
| **Live sync** | Published as a Claude artifact on claude.ai | A shared real-time database (the operator's phone and the doctor's laptop update live), stored photos, and an optional AI photo-quality check (Claude, asked only about photo quality, never for a diagnosis). |

## Run it

No build step and no dependencies to install. It's a single HTML file.

```bash
git clone https://github.com/icberg/harmony-reach-eye-twin.git
cd harmony-reach-eye-twin
open index.html          # or double-click it
```

You need to be online, because Three.js r128 loads from cdnjs and jsDelivr.

The live demo is served by GitHub Pages from the `main` branch root (Settings → Pages → Deploy from branch).

## Tech

- HTML, CSS and vanilla JavaScript in one file
- Three.js r128 + OrbitControls
- Canvas 2D for image analysis and texture generation
- Figtree + JetBrains Mono (Google Fonts)
- Claude artifact runtime (`db`, `assets`, `sample`), optional, for live sync

## Data model

One document per visit (`visits/<code>`) and one twin per eye (`twins/<id>`):

```text
visits/<code>
  code, createdAt, patientName, patientKey, twinId, age, sex, diabetes, visionLoss, eye,
  status: urgent | waiting | reviewed,
  iqa:   { score, focus, light, centred, glare, ai },
  iris:  { cx, cy, r },  irisData, thumbData,  fundusData | fundusId,
  oct:   { device: cirrus | spectralis, cst, inner{S,N,I,T}, outer{S,N,I,T}, simulated },
  grade: { dr: none | mild | moderate | severe | pdr, ncidme },
  plan:  { dueDays, dueAt, refer, referDays, basis[], proposed, overridden, reason, approvedAt },
  triage:{ level: green | amber | red, note, at }

twins/<patient>-<od|os>
  patientName, age, sex, eye, history[{ code, at, oct, dr }],
  plan, dueAt, state: awaiting-review | scheduled, updatedAt
```

## Clinical sources

- Wong TY et al. *Guidelines on Diabetic Eye Care: The International Council of Ophthalmology Recommendations for Screening, Follow-up, Referral, and Treatment Based on Resource Settings.* Ophthalmology, 2018 (ICO Guidelines 2017, Tables 2a/2b).
- DRCR Retina Network protocol eligibility criteria for centre-involved DME (e.g. Protocol AN), by OCT device and sex.
- Kritzinger W et al. *Digital Twin in manufacturing: A categorical literature review and classification.* IFAC-PapersOnLine 51(11), 2018.

## Limits

- No live camera stream. The page uses the browser's photo picker, which opens the camera on phones.
- OCT values are typed in or simulated. A real version would import them from the device (for example a Topcon Maestro2). Topcon's own thickness scale would need its own threshold or a published conversion, since the DRCR thresholds here are for Cirrus and Spectralis.
- The plan engine supports a clinician; it doesn't replace one. Every plan needs a doctor's approval, and the twin's trend rules aren't validated.
- There are no accounts or access control. **Don't use real patient data.**

## Changelog

- **2026-10-08:** v2, from digital shadow to digital twin. Added the OCT macula step and 3D macula view, one evolving twin record per eye, the ETDRS grid and thickness trend, the guideline-based care-plan engine with doctor approval and overrides, and the operator's recall list. New screenshot of the doctor view.
- **2026-10-04:** Fixed the role tabs. All three views were showing stacked on one page because a layout style overrode the `hidden` attribute. Switching tabs now shows one role at a time, scrolls to the top, and follows `#role` links. Added a screenshot and a quick-start walkthrough.
- **2026-10-03:** First release on GitHub Pages.

## Context

Part of a team service design for Topcon Healthcare: mobile screening vans run by trained drivers, edge AI image-quality checks, a remote specialist portal, and sales to governments for mandatory screening programmes. This prototype covers the operator → doctor → patient part of that service.

Built by Fahim Murshed with Claude Code.
