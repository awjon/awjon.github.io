# Jonathan Wong — Profile Website Redesign Plan

**Author:** Architect review (Claude)
**Executor:** Opus 4.8 — follow this document as the build spec.
**Audience for the site:** Recruiters and hiring managers arriving from Jonathan's LinkedIn profile.
**Prime directive:** Maximum visual impact in the first 3 seconds, and a recruiter must be able to extract Jonathan's value and career arc in under 60 seconds — without being forced through a linear experience.

---

## 1. Review of the current site (`index.html`)

### What works — keep the DNA
- **The pixel-art "life as a video game" concept is genuinely memorable.** Nobody else's portfolio does this. It signals creativity, personality, and hands-on technical craft. It must not be thrown away.
- The narrative arc (Hong Kong → Hawaii → Illinois → Seattle → Amazon → Microsoft → Starbucks → Microsoft MSRC) is a strong, human story.
- Nice details: typewriter dialog boxes, EXP progress bar, chiptune audio with mute, walk-cycle sprites that age with the story, cloud flight transitions with achievement cards.

### What fails against the stated goal
1. **The first thing a visitor sees is a loading screen.** 1.5 MB of base64 images inline in one HTML file means no progressive render, no browser caching, no CDN benefit. The "visually attractive right at the beginning" goal is lost before it starts.
2. **Zero value proposition above the fold.** The title card says "MY LIFE IN PIXELS — JONATHAN WONG" and four locations. A recruiter learns nothing: no current role, no years of experience, no skills, no CTA. They must scroll ~12,000px of scroll-jacked canvas to assemble the picture.
3. **The story is chronological; recruiters read reverse-chronological.** The strongest material — Program Manager at Microsoft Security Response Center, the Starbucks UberEats launch (5,496 stores, +$4.1M) — arrives last, after the baby-in-Hong-Kong scenes. Most visitors will never reach it.
4. **The experience is mandatory, not optional.** There is no skip, no chapter navigation, no way to jump to "career." A recruiter with 45 seconds bounces.
5. **No conversion path.** LinkedIn/GitHub links appear only in the finale. No résumé download, no email, nothing persistent on screen.
6. **No social/SEO metadata.** A link shared from LinkedIn renders with no preview card, no description. `<title>` is the only metadata.
7. **Accessibility problems:** all content lives in a `<canvas>` (invisible to screen readers and search engines), `user-scalable=no` blocks zoom, typewriter pacing can't be accelerated, `prefers-reduced-motion` only disables one bounce animation.
8. **Career content is thin.** Each job gets 1–2 sentences. There are strong quantified wins hiding in there; they deserve to be surfaced as headline stats, not buried in dialog text.

### Verdict
Great creative asset, wrong information architecture for the audience. The fix is not "make the game better" — it's **invert the structure**: lead with a cinematic, instantly-legible hero and a scannable recruiter layer, and demote the pixel journey to a delightful opt-in "Story Mode" that becomes the differentiator instead of the gatekeeper.

---

## 2. Design vision

**Concept: "The portfolio is the pause menu; the game is the bonus level."**

A single-page site with two layers:

- **Layer 1 — The Recruiter Layer (default).** A stunning, animated, pixel-art-flavored hero followed by conventional, scannable sections: impact stats, reverse-chronological career timeline, skills, story-of-me, contact. Everything a recruiter needs with zero clicks and ordinary scrolling. Visual language keeps the retro-game identity (pixel accents, chip badges, EXP bars, CRT-subtle touches) but typeset and structured like a modern product page.
- **Layer 2 — Story Mode (opt-in).** The existing scroll-driven pixel game, preserved nearly as-is, launched via a prominent "▶ PLAY STORY MODE (2 min)" button. Enhanced with chapter navigation and a skip/exit control. This is the memorable hook, not the front door.

This gives you the best of both: instant wow + instant information, with the game as the thing they tell their colleagues about.

---

## 3. Page architecture (Layer 1, top to bottom)

### 3.1 Hero — full viewport, must land in < 1 second
The visual centerpiece. A **living pixel-art Seattle skyline panorama** rendered as 3–4 parallax layers (sky/clouds, distant skyline with Space Needle, mid buildings, foreground rooftop or street). Subtle ambient animation: drifting clouds, twinkling window lights, occasional tiny plane crossing the sky (reusing the plane sprite as a callback). The adult sprite stands idle (2-frame idle animation) at the foreground edge, looking at the city.

Overlaid content, left-aligned or centered:
- **Eyebrow line:** `HELLO, I'M`
- **H1:** `JONATHAN WONG` (large pixel/display font)
- **Headline:** `Program Manager — Microsoft Security Response Center` (or Jonathan's preferred current title)
- **One-line value prop**, e.g.: `12+ years turning ambiguous problems into shipped systems at Microsoft, Amazon, and Starbucks.`
- **Stat chips row** (pixel "achievement badge" style): `12+ YRS EXPERIENCE` · `3 FORTUNE-500s` · `+$4.1M REVENUE LAUNCH` · `5,496 STORES SHIPPED`
- **CTA row:** `▶ PLAY STORY MODE` (primary, launches Layer 2) · `📄 RÉSUMÉ (PDF)` · `in LINKEDIN` · `✉ EMAIL`
- Animated scroll cue at the bottom.

Hero must render its first frame from lightweight layered images/CSS (< 250 KB budget) with **no loading screen**. Text appears immediately; parallax layers can fade in.

Optional flourish (cheap, high impact): a day/night toggle in the corner that swaps the skyline palette (dusk ↔ night) — reinforces the game aesthetic and invites play.

### 3.1a HD-2D depth of field & lighting (Octopath Traveler look)

This is the signature visual upgrade for the redesign — the "diorama" effect where pixel-art layers read as a real miniature 3D scene. It's a lighting/blur treatment applied on top of the layered art from Section 6, not a separate art style, and it applies to **both** the hero and Story Mode.

**Core technique — tilt-shift depth of field:** keep exactly one layer perfectly sharp (the focal plane — wherever the character/action is), and blur every layer in front of and behind it. The eye reads sharp-middle + blurred-front/back as real depth, even though every layer is flat.

| Layer | Blur | Notes |
|---|---|---|
| Sky / far skyline (back) | `blur(4px)` | Slight desaturation (~10%) also sells distance. |
| Mid buildings / street (focal plane) | `blur(0)` | Always tack-sharp. Character sprite and all UI/dialog text live on or in front of this plane — never blur text. |
| Foreground overhang (rooftop ledge, railing, foliage framing the top/edges of frame) | `blur(3px)` | Optional per scene; only needed where art has a near-camera element. |

Implementation differs by layer type:
- **Hero (CSS/DOM layers):** each parallax layer is a stacked `<div>`/`<img>`; apply `filter: blur(Npx)` per layer in CSS. Promote blurred layers to their own GPU compositing layer (`will-change: filter; transform: translateZ(0)`) so blur doesn't get recomputed on scroll.
- **Story Mode (Canvas 2D engine):** the existing engine already draws each backdrop layer via `ctx.drawImage` per frame — use the Canvas 2D context's native `ctx.filter = 'blur(4px)'` immediately before drawing the far layer, then `ctx.filter = 'none'` before drawing the mid layer/character, then re-apply for any foreground layer. No extra `<canvas>` elements needed; this is a ~3-line change to `drawBackdrop()`.

**Bloom / glow:** any bright light source in a scene (sunset, window lights, streetlamps, the "EXP" gold accent) gets a soft glow via `filter: drop-shadow(0 0 8px rgba(255,210,120,.55))` (CSS) or a radial-gradient sprite drawn behind it (canvas). Keep it subtle — this is a highlight accent, not a full bloom-pass shader.

**Vignette:** a fixed full-bleed `radial-gradient(circle at 50% 45%, transparent 55%, rgba(5,6,14,.35) 100%)` overlay, `mix-blend-mode: multiply`, ~20% opacity. Frames the scene and hides layer edges/seams.

**Optional polish (only after the above ships and looks right):** slow-drifting dust-mote/firefly particles in the mid-far layer (a handful of small soft-glow dots animating with `translate` + `opacity`) — cheap, and it's the detail that most reads as "premium game" rather than "static background."

**Performance guardrails:**
- Blur only the far and (if present) foreground layers — never the focal/mid layer, and never blur any layer carrying text or UI.
- Cap blur radius at ≤4px; wider blurs cost significantly more on mobile GPUs for barely more visual effect.
- `prefers-reduced-motion` disables the parallax *motion*, not the blur itself (static blur isn't motion) — dust motes should freeze or be removed under reduced motion.
- QA on a mid-range Android device, not just desktop Chrome — CSS/canvas blur is the single most likely perf regression in this whole redesign.

### 3.2 "Career at a glance" — the recruiter's 30 seconds
Reverse-chronological interactive timeline styled as **level-select cards**. One card per role, each with a small pixel emblem, and a thin timeline rail connecting them (EXP-bar styling as a nod to the game):

1. **Microsoft — Security Response Center (MSRC)** · Program Manager · 3 yrs
   - Delivered scalable, critical Virtual Desktop Infrastructure and Isolated Tenants for security response.
   - Focus: automation, cybersecurity, AI, site reliability.
2. **Starbucks** · Business Analyst · 1 yr
   - Launched UberEats delivery to **5,496 stores → +$4.1M revenue**.
   - Reopened **500+ stores** during the pandemic; built store-delivery systems as BCDR.
3. **Microsoft** · Project / Program Manager · 7 yrs
   - Built and streamlined processes across Azure, DevOps, SQL Server, co-op & email marketing, GitHub, PowerApps, Dynamics CRM.
4. **Amazon** · Site Merchandiser · 1 yr
   - Campaign UI/UX, A/B testing, product design and product management.

Each card: company, role, duration chip (reuse the existing chip color system: gold company / grey role / green years), 2–3 quantified bullets, and small skill tags. Cards animate in on scroll (respect `prefers-reduced-motion`). **Note to executor:** invite Jonathan to supply exact dates and any additional metrics per role before finalizing copy; the bullets above are extracted from the current site's text.

### 3.3 Impact band — numbers that stop the scroll
A full-width strip of 3–4 huge animated counters in arcade-score styling:
`$4.1M revenue unlocked` · `5,496 stores launched` · `500+ stores reopened` · `12+ years / 3 industry giants`.
Count-up animation on first view; static for reduced motion.

### 3.4 Skills & toolset — "Character stats"
Grouped skill inventory presented as a game inventory/stats screen (but instantly scannable text, not canvas):
- **Program & Project Management:** roadmapping, due diligence, process design, A/B testing, BCDR, supply chain & retail ops.
- **Technical:** Azure, DevOps, SQL Server, GitHub, PowerApps, Dynamics CRM, automation, VDI, cybersecurity fundamentals, AI, SRE.
- **Creative & Communication:** UI/UX, photo/video editing, campaign design, bilingual (English/Cantonese).
- **Community:** HKBAW board member, founded own company.
Render as labeled pixel-badge grids or stat bars — decorative but readable as plain text in the DOM.

### 3.5 "The Journey" teaser — bridge to Story Mode
A short horizontal strip: four framed pixel postcards (Hong Kong → Hawaii → Illinois → Washington) with one-line captions from the current script, and a big centered `▶ PLAY THE FULL STORY` button. This gives the life story a visible, skimmable presence in Layer 1 while funneling curious visitors into Layer 2. Reuse the four existing location backdrops as the postcard art.

### 3.6 Contact / finale
Clear closer: short availability line ("Open to conversations about program management roles in …" — confirm wording with Jonathan), then large CTA buttons: LinkedIn (`linkedin.com/in/jonkwong`), GitHub (`github.com/awjon`), Email, Résumé PDF. Footer keeps the arcade voice: `THANKS FOR VISITING — INSERT COIN TO CONTINUE ▶`.

---

## 4. Story Mode (Layer 2) — preserve and polish

Keep the existing engine and scenes with these upgrades only:

1. **Entry/exit:** launched from the hero or journey-teaser buttons. Opens as a distinct mode (own scroll context or route `#story`). Persistent `✕ EXIT` and `SKIP ▸▸` controls top-left; exiting returns to the recruiter layer where they left off.
2. **Chapter navigation:** a dot/level rail (fixed, right edge) with one dot per scene; click to jump. Label on hover ("Hong Kong", "Amazon", …).
3. **Lazy loading:** Story Mode assets load only when the mode is launched (or prefetched after the hero is interactive). The recruiter layer must never pay for them.
4. **Typewriter speed:** click/tap anywhere on the dialog to complete the text instantly (standard JRPG behavior).
5. **Keep** the chiptune, EXP bar, sprites, flight cards, rain effects — they're good.
6. **Fix:** remove `user-scalable=no`; add a real `prefers-reduced-motion` path (no typewriter, no parallax rush — fade transitions instead); give the canvas an offscreen text alternative (visually-hidden HTML transcript of all scene text).
7. **HD-2D depth of field:** every backdrop scene (Hong Kong, Hawaii, Illinois, Seattle, and each career location) gets the same tilt-shift blur/bloom/vignette treatment specified in Section 3.1a — this is what makes Story Mode look like an upgrade rather than a re-skin of the current site. Requires each backdrop to exist as separated depth layers per Section 6, not the current single flattened JPEG per scene.

---

## 5. Technical specification

### Stack & structure
- **Vanilla HTML/CSS/JS, no framework, no build step** (matches current deployment; keeps GitHub Pages simple). Split the monolith:

```
/index.html          — recruiter layer markup + story-mode shell
/css/main.css
/js/main.js          — recruiter layer interactions (counters, reveals, parallax)
/js/story.js         — the existing game engine, extracted & lazy-loaded
/assets/img/...      — all images as real files (WebP/PNG), no base64
/assets/audio/       — (none needed; chiptune stays procedural WebAudio)
/resume/jonathan-wong.pdf
```

- Real semantic HTML for the recruiter layer (`<header> <main> <section> <footer>`, h1–h3 hierarchy). All career content must exist as DOM text — SEO + screen readers + copy-paste for recruiters.

### Performance budget
- First contentful paint: hero text visible < 1 s on 4G; **no loading screen for Layer 1**.
- Above-the-fold payload ≤ 300 KB (hero parallax layers as compressed WebP, each ≤ 60 KB; fonts subset or system-stack pixel font via `font-display: swap`).
- Convert every existing JPEG to WebP at rendering size (they're currently 78–115 KB each; target ≤ 45 KB). Sprites stay PNG. Lazy-load everything below the fold (`loading="lazy"`), prefetch Story Mode assets on idle.

### Metadata & sharing (critical for the LinkedIn → site path)
- `<title>Jonathan Wong — Program Manager · Microsoft, Starbucks, Amazon</title>`
- Meta description summarizing the value prop.
- Full Open Graph + Twitter card tags with a dedicated 1200×630 share image (see assets).
- Favicon set (pixel-art "JW" mark).
- JSON-LD `Person` schema (name, jobTitle, alumniOf, sameAs → LinkedIn/GitHub).

### Accessibility checklist
- Remove `maximum-scale=1.0, user-scalable=no`.
- Visible focus states; all CTAs are real `<a>`/`<button>`.
- `prefers-reduced-motion`: disable parallax drift, counters jump to final value, story mode uses fades.
- Color contrast ≥ 4.5:1 for all text over imagery (use overlay plates as the current site already does — that pattern is fine).
- Hidden transcript for Story Mode canvas content.

### Nice-to-haves (only after everything above is done)
- Day/night hero toggle.
- Konami code easter egg (↑↑↓↓←→←→BA) that triggers a confetti of pixel sprites — recruiters love telling people about this.
- Print stylesheet that renders the recruiter layer as a clean one-pager.

---

## 6. Image assets — required from Jonathan or to be generated

### Reuse (already in the current `index.html` — extract, re-encode as WebP, save to `/assets/img/`)
| Asset | Current key | Use in redesign |
|---|---|---|
| Hong Kong backdrop | `hk` | Story Mode + journey postcard |
| Hawaii backdrop | `hawaii` | Story Mode + journey postcard |
| Illinois backdrop | `illinois` | Story Mode + journey postcard |
| Seattle backdrop | `seattle` | Story Mode + journey postcard (and hero reference) |
| Amazon / Starbucks / Microsoft ×2 backdrops | `amazon`,`starbucks`,`msft1`,`msft2` | Story Mode + timeline card thumbnails |
| Clouds | `clouds` | Story Mode flights |
| Character sprites (8) | `spr_*` | Story Mode + hero idle character + easter egg |
| Plane sprite | `spr_plane` | Story Mode + hero ambient flyby |
| Flight photos (3) | `flightimg1-3` | Story Mode achievement cards |

### Layer format requirements (applies to every scene below and to Section 3.1a's depth-of-field effect)
Each scene needs 3–4 **separated depth layers**, not one flattened image:
- **Back layer** (sky/far skyline): full-bleed, opaque, no transparency needed — JPEG/WebP.
- **Mid/focal layer** (main buildings, street, where the character stands): transparent PNG/WebP cutout so the back layer shows through gaps (sky between buildings, etc.).
- **Foreground layer** (rooftop ledge, railing, foliage), if present: transparent PNG/WebP, positioned to overlap the top/edges of frame.
- Each layer wider than the target viewport (extra horizontal bleed) so it has room to travel independently during parallax pan without exposing an edge.
- This layer separation is what makes the Section 3.1a blur/bloom/vignette treatment possible — a single flattened JPEG (which is what every current backdrop is) can only be blurred as a whole, which kills the depth illusion rather than creating it.

### New — must be created (AI-generate in the same pixel style as existing backdrops, or commission)
1. **Hero parallax set (highest priority):** pixel-art Seattle skyline at dusk in 3–4 separable layers per the format above — (a) sky + clouds, (b) far skyline with Space Needle silhouette, (c) mid buildings with lit windows, (d) foreground ledge/rooftop where the character stands. Landscape ≥ 1920px wide each, plus a night-palette variant if the day/night toggle ships. *(The existing `seattle` backdrop sets the style reference but is a single flattened image — it cannot be used for parallax or depth of field as-is.)*
1a. **Re-layer the existing Story Mode backdrops** (`hk`, `hawaii`, `illinois`, `seattle`, `amazon`, `starbucks`, `msft1`, `msft2`) into the same back/mid/foreground format so Section 4 item 7's HD-2D treatment applies across all of Story Mode, not just the hero. This can reuse the current images as the mid/focal layer and only requires generating new back (sky) and, optionally, foreground layers to match — lower effort than building each scene from scratch.
2. **Idle-pose sprite frames (2)** for the adult character (standing, slight breathing bob) — current sprites are walk-cycle only.
3. **Four pixel company emblems** for timeline cards (Amazon / Microsoft / Starbucks / MSRC-shield motifs). ⚠️ Do **not** reproduce real trademarked logos — use abstract pixel monograms/icons + text labels.
4. **OG share image**, 1200×630: hero skyline + "JONATHAN WONG — Program Manager" in the pixel display type.
5. **Favicon / touch icons:** pixel "JW" monogram (32/180/512 px).

### New — must come from Jonathan (put placeholders in and flag them)
6. **Résumé PDF** (`/resume/jonathan-wong.pdf`).
7. **Optional but recommended: one real photo** (professional or candid-professional headshot). A small framed "player portrait" beside the contact section humanizes the pixel theme. If provided, also make a pixelated variant for a hover "sprite ↔ photo" flip.
8. **Confirmed copy inputs:** current exact job title, employment dates per role, preferred contact email, availability statement.

---

## 7. Content inventory (verbatim source text to reuse)

All existing scene text, chips, and achievements from the current site are the canonical copy source — Hong Kong, Hawaii, Illinois, Washington scene paragraphs; the flight achievement cards ("I speak Cantonese", "HKBAW board member", "Interned with Aloha Tower", "Graduated Summa Cum Laude", "I can photo and video edit", "Started my own company"); and the four career scene texts. Extract them from the `SCENES` array in the current `index.html` before restructuring. Tighten grammar where needed but keep Jonathan's voice.

---

## 8. Build order for the executor

1. Extract all base64 assets from current `index.html` to `/assets/img/`, convert to WebP, commit the old file as `story-legacy.html` reference.
2. Build Layer 1 skeleton: semantic HTML, all copy, metadata, working CTAs. (Site is already useful at this point.)
3. Build hero parallax + ambient animations with the new layered art.
4. Build timeline, impact counters, skills, journey teaser, contact.
5. Extract the game engine into `js/story.js`; wire entry/exit, chapter dots, lazy load, tap-to-complete dialog.
6. Accessibility + reduced-motion pass, performance pass (Lighthouse ≥ 90 on Performance/A11y/SEO/Best Practices), mobile QA at 360px/768px/1440px.
7. Placeholders clearly marked for: résumé PDF, exact dates/title, email, headshot.

### Acceptance criteria
- No loading screen before first paint of the recruiter layer.
- Name, current role, value prop, and at least one CTA visible without scrolling on a 360×740 phone and a 1440×900 laptop.
- A recruiter can reach the full career timeline within one screen of scrolling.
- Story Mode fully playable, skippable, and exitable; never auto-starts.
- Link pasted into LinkedIn/Slack/iMessage shows a proper preview card.
- Lighthouse ≥ 90 across all four categories on mobile emulation.
