# CLAUDE.md — Reef Cartographer site

Project rules and locked decisions for this repo. Read before editing.

## What this is
Companion website for the **Reef Cartographer Specialty** (subtitle: *Acoustic & Optical Reef Mapping*), a finished recreational distinctive scuba specialty teaching divers to make georeferenced reef maps from castable consumer sonar + photogrammetry. The site is: public tools (calculators) + course presence + a contact funnel. Live at **reefcartographer.com** (GitHub → Netlify, free tier).

## Hard architecture constraints — do not violate
- **Static files only.** No build tools, no npm, no bundlers, no frameworks. Plain HTML/CSS/JS that a browser runs directly and that can be dragged into a host. Do not introduce a build step.
- **Persistence:** browser `localStorage`, always wrapped in try/catch degrading to in-memory. Never assume storage exists.
- **PWA / offline-installable.** Service worker (`sw.js`) precaches the core pages.
- **`tools.html` is self-contained** — its own inline `<style>` and `<script>`, NOT linked to `styles.css`. This is deliberate to avoid class collisions (`.card`/`.note`/`.num` mean different things there). The marketing pages (`index/course/contact/thanks/rig.html`) DO share `styles.css`.

## Service worker rule (easy to forget, breaks updates if missed)
**Every time you change any cached file, bump the `CACHE` constant in `sw.js`** (e.g. `reef-carto-v26` → `v27`). The SW serves HTML network-first but CSS/JS cache-first, so without a version bump users get stale assets. After deploying, the SW often needs one unregister (DevTools → Application → Service Workers) to pick up changes immediately.

## Branding
- Always **"Reef Cartographer Specialty"** — NEVER "PADI Reef Cartographer."
- PADI appears ONLY as prerequisite certifications: Advanced Open Water + Enriched Air/Nitrox (required); Peak Performance Buoyancy + AWARE + Master Scuba Diver (recommended).
- Source course docs (Course Guide + Student Handbook) are authoritative for facts; write FRESH web copy, don't paste from them. Never expose instructor-only material (answer keys).

## Tone / copy
- Honest, factual, "trust but verify." Recreational & educational — **not survey-grade or navigation-grade** (±3–8 m typical). Never let marketing copy contradict these disclaimers or borrow scientific authority.
- The "why map a reef" framing is witnessing/awareness (shifting baselines; "the first thing it changes is you"), NOT a claim of scientific-grade data. State once, don't sermonize.
- Reef-monitoring orgs (Reef Check, CoralWatch, Reef Life Survey) are named as worthwhile groups to support in their own right — NOT as guaranteed destinations for the user's data.

## The calculators (tools.html) — verified math, don't "simplify"
Five tools, all verified against the Course Guide. Internal math is in SI/metres; a global Metric/Imperial switch converts only at display/input edges.
- **Footprint:** D = 2·d·tan(θ/2); spacing s = D(1−O), round DOWN. (d = water depth surface→bottom, NOT diver depth.)
- **Refraction:** α_water = arcsin(sin α_air / n). **Frame width uses EXACT geometry W = 2·L·tan(α)** — do NOT revert to the course's linear-percentage approximation (it over-estimates W by ~14% at 45°, ~37% at 60°). Refractive index is medium-driven: **Salt 1.340 (default), Fresh 1.333, Custom**. Course Guide examples were computed at n=1.33 (reachable via Custom) — known, deliberate divergence pending doc reconciliation.
- **Baseline:** B = W(1−O); t = B/v (compute t in SI so it's unit-invariant).
- **Orbit:** Δφ = B/r; N ≈ 2πr/B + closure.
- **Accuracy builder:** generates an honest limitations statement from inputs.
- Tools chain: Refraction W → Baseline B → Orbit. "Trust, but verify" disclaimer sits above the tab bar.
- A 6th "Dive info" tab feeds the print slate (metadata only). Print uses native `window.print()` + a hidden print-only slate — NO PDF library.

## The Rig page (rig.html)
SVG line drawings of gear in the site's chart aesthetic. **Deeper Smart Sonar Max is the one REQUIRED, named, linked item** (deepersonar.com). Everything else is a generic "example" — no brand endorsements, no rotting product links — EXCEPT the case (Pelican M40, named softly with link) and buoy, which are illustrated. Key geometry fact: the M40 (17.3×12.7 cm, ~21.5 cm diagonal) fits the buoy's ~30 cm circular central well; longer cases (M60/1060, ~28 cm diagonal) do NOT once the inflated tube intrudes — the limit is the case DIAGONAL vs the circle.

## Known doc-reconciliation items (course docs, under PADI review — not website bugs)
1. Critical-angle inconsistency: docs state arcsin(1/1.33)≈48.6°, but that's the 1.333 value; at 1.33 it's 48.8°. Standardize one index.
2. Frame-width correction: Module 3 scales air width by angular FOV-reduction % (wrong — angular ratio used as linear). Correct factor is tan(α_water)/tan(α_air). Tools already use exact geometry.
3. Tool now defaults to salt n=1.340; Course Guide worked example (120°→40.6°) is at 1.33. Reconcile when PADI opens doc edits.

## Design tokens (styles.css :root)
Paper #f3ecdd, paper-2 #ece1cb, ink #0b2730, ink-soft #46606a, coral #f0613c, coral-600 #d94c28, teal-700 #0e4c5e, teal-300 #5cb7c3, sand-deep #d3bd92. Fonts: Fraunces (display), Archivo (UI), Spline Sans Mono (numeric). Nautical-chart aesthetic, mobile-first.

## Deploy
Repo → Netlify auto-builds (build command and publish dir both EMPTY — static root). Netlify Forms handles the contact form (form name="contact"); email notification goes to kamran@scoooba.com. Custom domain via GoDaddy DNS (A @ → 75.2.60.5; CNAME www → netlify). Sitemap, robots.txt, llms.txt at root — keep in sync when adding/removing pages.

## Outstanding / optional
- README_GO-LIVE.md still references the old "enroll" form name (pre-Contact-reframe) — update if touched.
- thanks.html is noindex and intentionally excluded from the sitemap.
