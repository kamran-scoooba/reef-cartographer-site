# Reef Cartographer — Go-Live Runbook

The complete website. Static files only — no build step, no npm. You drag a folder into a browser to deploy it.

---

## 1. The files to deploy

Upload **these nine files plus the three icons** as the site root. Do **not** include `footprint-grid-calculator.html` (it was an early single-tool prototype, now superseded by `tools.html`).

```
index.html              landing page
course.html             course detail
tools.html              the five-tool app
enroll.html             enrollment form (Netlify Forms)
styles.css              shared stylesheet (index/course/enroll)
sw.js                   service worker (offline/PWA)
manifest.webmanifest    PWA manifest
icon-192.png            app icon
icon-512.png            app icon
icon-maskable-512.png   app icon (Android maskable)
```

`tools.html` is self-contained (its own inline CSS/JS) and does **not** use `styles.css` — that is deliberate, to avoid class-name collisions. Keep all files in the same folder; every link between them is relative.

---

## 2. Deploy: GitHub → Netlify (free tier)

1. Create a new repository on **github.com** (e.g. `reef-cartographer-site`). On the repo page use **Add file → Upload files**, then drag the ten files in and commit.
2. At **app.netlify.com**, choose **Add new site → Import an existing project → GitHub**, pick the repo. Leave build command empty and publish directory as the repo root. Deploy.
3. You get a live `*.netlify.app` URL immediately. Confirm all four pages load and the tools work.

(You can also skip GitHub entirely: Netlify's **"deploy manually"** box accepts a drag-and-dropped folder. GitHub is recommended so future edits redeploy automatically.)

> I can't create accounts or push to GitHub/Netlify for you — those steps are yours.

---

## 3. Custom domains (DNS — your action)

You own `reefcartographer.com` (canonical) plus `.io / .ai / .org` and the `reefcartography.*` set.

1. In Netlify: **Domain settings → Add a custom domain →** enter `reefcartographer.com`.
2. At your registrar, point DNS to Netlify — either set the domain's nameservers to Netlify DNS, or add the records Netlify shows (an `A`/`ALIAS` for the apex and a `CNAME` for `www`).
3. Add the other domains as **domain aliases** and set redirects so they all 301 to `reefcartographer.com` (one canonical host is better for SEO than several live copies).
4. Netlify provisions a free Let's Encrypt certificate once DNS resolves.

---

## 4. Enrollment form email (your action)

`enroll.html` uses **Netlify Forms** (form name `enroll`). Submissions appear under **Forms** in the Netlify dashboard automatically — the destination email is **never in the page source**.

To get emailed on each enquiry: **Netlify → Forms → Notifications → Add notification → Email**, and enter the address you want enquiries sent to. A honeypot field is already in place; you can add reCAPTCHA in the same panel if spam appears.

---

## 5. Sponsor logo

Every page footer has a reserved slot rendering **"Sponsor slot reserved"** (`<div class="sponsor">`). When you have the Deeper asset, replace that text with the `<img>` in each of `index.html`, `course.html`, `enroll.html`, and (in its own markup) `tools.html`. Keep it to one clean logo.

---

## 6. Decisions locked in the build

- **Refractive index:** a single value, **n = 1.33**, used everywhere in the Refraction tool — matching the formula as written in Module 3.
- **Frame width:** exact geometry, **W = 2·L·tan(α)**, not the manual's linear-percentage approximation (which overestimates underwater width — ~14% at a 45° half-FOV, ~37% at 60°).
- **Pre-measured line:** presented as a *suggested* convenience aid (a step-over gauge that doubles as a photogrammetry scale reference), framed as a gauge, never a tether.
- **Tool chaining:** Refraction's W → Baseline; Baseline's B → Orbit, via "use value from…" buttons (values persist in `localStorage`, degrading to in-memory).
- **Architecture:** small multi-page static site sharing `styles.css`; the tools app stays a single self-contained file.

All five tools were checked against the Course Guide's worked examples (Footprint 60 ft/7°→ ~5 ft spacing; Refraction 120°→ 40.6° half / 32% reduction; Baseline 1.2 m/80%→ B = 0.24 m, t = 0.6 s; Orbit r = 2/B = 0.24 → 6.9°, ~58 frames).

---

## 7. Open items for you (not blockers)

These are content questions in **your course documents** — surfaced for the pre-PADI-submission pass, not deploy blockers:

1. **Critical-angle inconsistency.** The FOV table uses n = 1.33, but the stated critical angle "arcsin(1/1.33) ≈ 48.6°" is the n = **1.333** value. At n = 1.33 it is **48.8°**. Standardise on one index (a one-character fix) so the doc is internally consistent. The website uses 1.33 throughout.
2. **Frame-width correction method.** Module 3 derives underwater width by scaling the air width by the *angular* FOV-reduction percentage. That reuses an angular ratio as a linear one and overestimates width. The correct factor is **tan(α_water)/tan(α_air)**. The tool already uses exact geometry; the manual's worked number will shift if you correct it.
3. **Extending the suggested reference line.** It currently appears where the course supports it (lane spacing + scale). The same idea would fit the **subject standoff distance** (Refraction) and the **orbit radius** (Orbit) as convenience aids — say the word and I'll add those suggestions to those tools.

---

## 8. Updating the site later

Edit a file, re-upload (or `git push` if you used GitHub) — Netlify redeploys. **If you change any cached file, bump the cache name** in `sw.js` (`const CACHE = "reef-carto-v1"` → `-v2`) so returning visitors get the new version instead of the cached one.
