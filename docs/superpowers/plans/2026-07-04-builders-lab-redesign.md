# Builder's Lab Portfolio Redesign — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild `index.html` as a dark-studio, generative-visual portfolio ("빌더의 실험실") per the approved spec, keeping all Korean content text verbatim.

**Architecture:** Single-file `index.html`. One shared rAF loop (`Lab` engine) drives all Canvas 2D visuals; each visual is a factory returning `{update, draw, resize, renderStatic, onPointer?}`. Canvases are decorative (`aria-hidden`), paused offscreen via IntersectionObserver, and replaced by one static frame under `prefers-reduced-motion`.

**Tech Stack:** Vanilla JS, Canvas 2D, CSS. Fonts: Fraunces + Pretendard (existing) + IBM Plex Mono (new). No libraries, no build step, no WebGL.

**Spec:** `docs/superpowers/specs/2026-07-04-builders-lab-redesign-design.md`

## Global Constraints

- Single file: all changes go into `index.html`. No new runtime assets.
- All Korean copy stays **verbatim** — sections: nav → hero → about → projects → experience → links → footer.
- Keep: Cloudflare Insights `<script>` at the bottom, Pretendard CDN link, existing external links.
- Dark palette base `#0d0c0a`; shared accent gold `#c9a668`; per-project accents — groompick `#e0a458`, NewsLens `#6fc3d6`, 플러피 `#e08b5a`, 뭐먹을래 `#7fd6b2`.
- DPR capped at 2. Cursor-only effects gated on `(pointer: fine)`. `prefers-reduced-motion: reduce` → no animation, static canvas frames.
- No JS ⇒ page still fully readable (canvases and motion are enhancement only).
- Verification is manual/browser-based (no test framework exists for a static page): every task ends with concrete browser checks before commit.

---

### Task 1: Dark theme foundation (full CSS + head)

**Files:**
- Modify: `index.html` — `<head>` links (line ~10), entire `<style>` block (lines 12–222)

**Interfaces:**
- Produces: CSS classes later tasks rely on: `.hero-canvas`, `.hero::after` scrim, `.viz`, `.viz canvas`, `.viz-tag`, `.lab-status`, `.mono`, per-project modifiers `.project.gp/.nl/.fl/.mw` (each sets `--pa`).

- [ ] **Step 1: Baseline check** — Run `Start-Process index.html` (PowerShell). Confirm current light theme renders, note the 6 sections. This is the before-state.

- [ ] **Step 2: Update head** — Replace the Google Fonts link (line 10) with:

```html
<link href="https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,300..600;1,9..144,300..500&family=IBM+Plex+Mono:wght@400;500&display=swap" rel="stylesheet" />
```

and add after the viewport meta:

```html
<meta name="theme-color" content="#0d0c0a" />
```

- [ ] **Step 3: Replace the entire `<style>` block** with the dark-studio stylesheet below (complete, includes styles for elements added in later tasks):

```css
:root{
  --bg:#0d0c0a;
  --bg-2:#131210;
  --card:rgba(255,255,255,.04);
  --card-strong:rgba(255,255,255,.075);
  --text:#efeae2;
  --ink-soft:#c9c1b4;
  --muted:#8b8377;
  --line:rgba(239,234,226,.16);
  --line-soft:rgba(239,234,226,.08);
  --accent:#c9a668;
  --accent-deep:#a4854e;
  --accent-soft:rgba(201,166,104,.14);
  --gp:#e0a458; --nl:#6fc3d6; --fl:#e08b5a; --mw:#7fd6b2;
  --shadow:0 30px 60px -30px rgba(0,0,0,.85);
  --shadow-soft:0 18px 40px -24px rgba(0,0,0,.7);
  --ease:cubic-bezier(.22,.61,.36,1);
  --serif:"Fraunces","Pretendard Variable",Pretendard,serif;
  --sans:"Pretendard Variable",Pretendard,-apple-system,system-ui,sans-serif;
  --mono:"IBM Plex Mono",ui-monospace,SFMono-Regular,monospace;
}
*{ box-sizing:border-box; }
html{ scroll-behavior:smooth; -webkit-font-smoothing:antialiased; }
body{
  margin:0; font-family:var(--sans); color:var(--text);
  background:radial-gradient(120% 80% at 70% -10%, var(--bg-2), var(--bg) 55%);
  overflow-x:hidden;
}
a{ color:inherit; text-decoration:none; }
.mono{ font-family:var(--mono); }

/* ---------- ambient depth ---------- */
.ambient{ position:fixed; inset:-12%; z-index:0; pointer-events:none; will-change:transform; }
.orb{ position:absolute; border-radius:50%; filter:blur(90px); }
.orb.warm{ width:58vw; height:58vw; left:-16vw; top:-20vw; background:radial-gradient(circle, rgba(201,166,104,.10), transparent 62%); animation:drift1 30s var(--ease) infinite alternate; }
.orb.cool{ width:50vw; height:50vw; right:-14vw; top:30vh; background:radial-gradient(circle, rgba(111,195,214,.07), transparent 64%); animation:drift2 36s var(--ease) infinite alternate; }
@keyframes drift1{ to{ transform:translate3d(5vw,4vh,0) scale(1.07); } }
@keyframes drift2{ to{ transform:translate3d(-4vw,-3vh,0) scale(1.09); } }
.grain{ position:fixed; inset:0; z-index:1; pointer-events:none; opacity:.05; mix-blend-mode:overlay;
  background-image:url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='160' height='160'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='2' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E"); }

/* ---------- scroll progress ---------- */
.progress{ position:fixed; top:0; left:0; height:2px; width:0; z-index:50;
  background:linear-gradient(90deg, var(--accent-deep), var(--accent)); box-shadow:0 0 14px rgba(201,166,104,.6); transition:width .1s linear; }

/* ---------- shell ---------- */
.wrap{ position:relative; z-index:2; max-width:1160px; margin:0 auto; padding:0 28px 120px; }

.nav{ position:sticky; top:16px; z-index:40; margin:18px auto 0; max-width:1160px; padding:0 28px; }
.nav-inner{ display:flex; justify-content:space-between; align-items:center; gap:20px; padding:13px 20px;
  border:1px solid var(--line-soft); border-radius:100px; background:rgba(19,18,16,.55);
  backdrop-filter:blur(18px) saturate(1.3); -webkit-backdrop-filter:blur(18px) saturate(1.3);
  transition:box-shadow .4s var(--ease), background .4s var(--ease), border-color .4s var(--ease); }
.nav.scrolled .nav-inner{ box-shadow:var(--shadow-soft); background:rgba(19,18,16,.82); border-color:var(--line); }
.brand .eyebrow{ font-size:10px; color:var(--muted); letter-spacing:.36em; text-transform:uppercase; font-weight:500; font-family:var(--mono); }
.brand .eyebrow::before{ display:none; }
.brand .name{ margin-top:3px; font-size:15px; font-weight:700; letter-spacing:-.01em; }
.brand .name em{ font-family:var(--serif); font-style:italic; font-weight:400; color:var(--accent); }
.nav-links{ display:flex; gap:26px; font-size:13px; color:var(--muted); font-family:var(--mono); }
.nav-links a{ position:relative; padding:4px 0; transition:color .3s var(--ease); }
.nav-links a::after{ content:''; position:absolute; left:0; bottom:-1px; width:0; height:1px; background:var(--accent); transition:width .35s var(--ease); }
.nav-links a:hover{ color:var(--text); }
.nav-links a:hover::after{ width:100%; }

.eyebrow{ display:inline-flex; align-items:center; gap:10px; font-size:11px; color:var(--accent);
  letter-spacing:.3em; text-transform:uppercase; font-weight:500; font-family:var(--mono); }
.eyebrow::before{ content:''; width:26px; height:1px; background:linear-gradient(90deg, var(--accent), transparent); }

/* ---------- hero ---------- */
.hero{ position:relative; min-height:92vh; display:grid; grid-template-columns:1.18fr .82fr; gap:40px; align-items:center; padding-top:84px; }
.hero-canvas{ position:absolute; top:-84px; left:50%; transform:translateX(-50%); width:100vw; height:calc(100% + 84px); z-index:-1; pointer-events:none; }
.hero::after{ content:''; position:absolute; inset:-84px -28vw 0; z-index:-1; pointer-events:none;
  background:radial-gradient(90% 70% at 32% 45%, rgba(13,12,10,.72), rgba(13,12,10,0) 68%); }
.hero h1{ margin:20px 0 14px; font-size:clamp(60px, 11vw, 128px); line-height:.92; letter-spacing:-.05em; font-weight:700; }
.hero .sub{ font-family:var(--serif); font-size:clamp(20px, 2.6vw, 30px); color:var(--ink-soft); font-weight:400; letter-spacing:.01em; }
.hero .desc{ margin-top:26px; max-width:640px; font-size:16.5px; line-height:1.95; color:var(--ink-soft); }
.hero .desc strong{ color:var(--text); font-weight:600; box-shadow:inset 0 -.5em 0 var(--accent-soft); }
.hero-actions{ margin-top:32px; display:flex; flex-wrap:wrap; gap:14px; }
.lab-status{ display:inline-flex; align-items:center; gap:10px; margin-top:30px; padding:9px 15px; border:1px solid var(--line-soft);
  border-radius:100px; background:var(--card); font-family:var(--mono); font-size:11.5px; letter-spacing:.06em; color:var(--muted); }
.lab-status .dot{ width:7px; height:7px; border-radius:50%; background:#8fd67f; box-shadow:0 0 8px rgba(143,214,127,.8); animation:pulse 2.2s infinite; }
@keyframes pulse{ 0%,100%{ opacity:1; } 50%{ opacity:.35; } }
.lab-status b{ color:var(--ink-soft); font-weight:500; }

.btn{ display:inline-flex; align-items:center; gap:9px; padding:15px 24px; border-radius:100px; font-weight:600; font-size:14px;
  border:1px solid transparent; will-change:transform; transition:transform .35s var(--ease), box-shadow .4s var(--ease), background .4s var(--ease), color .4s var(--ease); }
.btn.primary{ background:var(--text); color:var(--bg); box-shadow:0 14px 30px -14px rgba(239,234,226,.35); }
.btn.primary:hover{ box-shadow:0 20px 44px -16px rgba(239,234,226,.45); }
.btn.ghost{ background:transparent; border-color:var(--line); color:var(--text); }
.btn.ghost:hover{ border-color:var(--accent); color:var(--accent); box-shadow:var(--shadow-soft); }
.btn .arr{ transition:transform .4s var(--ease); }
.btn:hover .arr{ transform:translate(3px,-3px); }

.pill-row{ display:flex; flex-wrap:wrap; gap:10px; }
.pill{ padding:9px 15px; border-radius:100px; background:var(--card); border:1px solid var(--line-soft);
  color:var(--ink-soft); font-size:12.5px; font-weight:500; transition:transform .35s var(--ease), border-color .35s var(--ease); }
.pill:hover{ transform:translateY(-2px); border-color:var(--accent); }

.hero-right{ display:grid; gap:18px; }

/* ---------- panels ---------- */
.panel{ position:relative; padding:26px; border-radius:24px; background:var(--card); border:1px solid var(--line-soft);
  box-shadow:var(--shadow-soft); backdrop-filter:blur(16px); -webkit-backdrop-filter:blur(16px); overflow:hidden; }
.panel::before{ content:''; position:absolute; inset:0 0 auto 0; height:1px; background:linear-gradient(90deg, transparent, rgba(239,234,226,.22), transparent); }
.panel .positioning{ margin-top:16px; font-family:var(--serif); font-size:27px; line-height:1.38; font-weight:400; letter-spacing:-.01em; color:var(--text); }

.quick-list{ display:grid; gap:0; margin-top:10px; }
.quick-item{ display:flex; justify-content:space-between; gap:14px; padding:13px 0; border-bottom:1px solid var(--line-soft); font-size:14px; }
.quick-item:last-child{ border-bottom:none; }
.quick-item span:first-child{ color:var(--muted); letter-spacing:.02em; font-family:var(--mono); font-size:12.5px; }
.quick-item strong{ text-align:right; font-weight:600; color:var(--text); }

/* ---------- sections ---------- */
.section{ padding-top:128px; }
.section-head{ margin-bottom:42px; max-width:760px; }
.section-head h2{ margin:16px 0 0; font-family:var(--serif); font-size:clamp(32px, 5vw, 58px); letter-spacing:-.02em; font-weight:400; line-height:1.04; will-change:transform, letter-spacing; }
.section-head h2 .it{ font-style:italic; color:var(--accent); font-weight:300; }

.about-grid{ display:grid; grid-template-columns:1.05fr .95fr; gap:24px; align-items:start; }
.about-copy{ padding:34px; border-radius:26px; background:var(--card); border:1px solid var(--line-soft); box-shadow:var(--shadow-soft); }
.about-copy .lead{ font-family:var(--serif); font-size:21px; line-height:1.6; color:var(--text); font-weight:400; }
.about-copy p{ color:var(--ink-soft); line-height:2; font-size:15.5px; }
.skill-list{ display:grid; gap:14px; }
.skill-item{ padding:22px 24px; border-radius:22px; background:var(--card); border:1px solid var(--line-soft); transition:transform .4s var(--ease), box-shadow .4s var(--ease), border-color .4s var(--ease); }
.skill-item:hover{ transform:translateY(-3px); box-shadow:var(--shadow-soft); border-color:var(--line); }
.skill-item .k{ display:inline-flex; align-items:center; gap:9px; color:var(--accent); font-size:11px; letter-spacing:.22em; text-transform:uppercase; font-weight:500; font-family:var(--mono); }
.skill-item .k::before{ content:''; width:7px; height:7px; border-radius:50%; background:var(--accent); box-shadow:0 0 0 4px var(--accent-soft); }
.skill-item .v{ margin-top:12px; color:var(--ink-soft); line-height:1.85; font-size:14.5px; }

/* ---------- projects ---------- */
.cards{ display:grid; grid-template-columns:repeat(2, minmax(0,1fr)); gap:22px; }
.project{ --pa:var(--accent); position:relative; padding:30px; border-radius:28px; background:var(--card); border:1px solid var(--line-soft);
  box-shadow:var(--shadow-soft); overflow:hidden;
  transition:transform .5s var(--ease), box-shadow .5s var(--ease), border-color .5s var(--ease); }
.project.gp{ --pa:var(--gp); } .project.nl{ --pa:var(--nl); }
.project.fl{ --pa:var(--fl); } .project.mw{ --pa:var(--mw); }
.project.lead{ grid-column:1 / -1; }
.project:hover{ transform:translateY(-5px); box-shadow:var(--shadow); border-color:var(--line); }
.viz{ position:relative; height:200px; margin:-8px -8px 22px; border-radius:20px; overflow:hidden;
  border:1px solid var(--line-soft); background:linear-gradient(160deg, rgba(255,255,255,.03), rgba(0,0,0,.25)); }
.project.lead .viz{ height:240px; }
.viz canvas{ position:absolute; inset:0; width:100%; height:100%; display:block; }
.viz-tag{ position:absolute; left:14px; bottom:11px; font-family:var(--mono); font-size:10px; letter-spacing:.18em;
  text-transform:uppercase; color:var(--pa); opacity:.85; pointer-events:none; }
.project-top{ display:flex; justify-content:space-between; gap:18px; align-items:flex-start; position:relative; z-index:1; }
.project h3{ margin:0; font-family:var(--serif); font-size:clamp(26px,3vw,34px); letter-spacing:-.02em; font-weight:500; }
.project .meta{ flex:none; font-size:10.5px; letter-spacing:.14em; text-transform:uppercase; color:var(--pa); font-weight:500;
  font-family:var(--mono); text-align:right; padding:6px 12px; border:1px solid var(--line-soft); border-radius:100px; background:var(--card-strong); }
.project .sub{ margin-top:9px; color:var(--ink-soft); font-weight:500; font-size:15px; position:relative; z-index:1; }
.project p{ color:var(--ink-soft); line-height:1.85; font-size:14.5px; position:relative; z-index:1; }
.project.lead .lead-grid{ display:grid; grid-template-columns:1.2fr .8fr; gap:8px 40px; }
.label{ margin-top:20px; font-size:10px; color:var(--muted); letter-spacing:.24em; text-transform:uppercase; font-weight:500; font-family:var(--mono); position:relative; z-index:1; }
.livelink{ display:inline-flex; align-items:center; gap:7px; margin-top:8px; color:var(--pa); font-weight:600; font-size:14.5px; position:relative; z-index:1; }
.livelink::after{ content:'↗'; transition:transform .35s var(--ease); }
.livelink:hover::after{ transform:translate(3px,-3px); }
.stats-inline{ display:flex; flex-wrap:wrap; gap:12px; margin-top:18px; position:relative; z-index:1; }
.badge{ padding:12px 16px; border-radius:16px; background:var(--card-strong); border:1px solid var(--line-soft); }
.badge strong{ display:block; font-family:var(--serif); font-size:21px; color:var(--pa); font-weight:500; letter-spacing:-.01em; }
.badge span{ display:block; margin-top:4px; color:var(--muted); font-size:12px; }
.stack{ margin-top:20px; padding-top:18px; border-top:1px solid var(--line-soft); font-size:13px; color:var(--ink-soft); position:relative; z-index:1; }
.stack b{ color:var(--pa); font-weight:500; letter-spacing:.08em; text-transform:uppercase; font-size:11px; font-family:var(--mono); }

/* ---------- experience ---------- */
.exp-grid{ display:grid; grid-template-columns:1.05fr .95fr; gap:24px; align-items:start; }
.timeline{ display:grid; gap:18px; }
.timeline-item{ position:relative; padding:26px 26px 26px 34px; border-radius:24px; background:var(--card); border:1px solid var(--line-soft); box-shadow:var(--shadow-soft); overflow:hidden; transition:transform .4s var(--ease); }
.timeline-item:hover{ transform:translateX(4px); }
.timeline-item::before{ content:''; position:absolute; left:0; top:22px; bottom:22px; width:3px; border-radius:100px; background:linear-gradient(180deg, var(--accent), var(--accent-deep)); box-shadow:0 0 14px rgba(201,166,104,.5); }
.timeline-item h3{ margin:0; font-family:var(--serif); font-size:22px; font-weight:500; }
.timeline-item .meta{ margin-top:7px; color:var(--accent); font-size:11px; font-weight:500; letter-spacing:.12em; text-transform:uppercase; font-family:var(--mono); }
.timeline-item ul{ margin:16px 0 0; padding-left:20px; color:var(--ink-soft); line-height:1.85; font-size:14.5px; }
.timeline-item li{ margin-bottom:8px; }
.timeline-item li::marker{ color:var(--accent); }

/* ---------- links ---------- */
.contact-grid{ display:grid; grid-template-columns:1.05fr .95fr; gap:24px; align-items:start; }
.link-list{ display:grid; gap:14px; }
.link-card{ position:relative; padding:22px 24px; border-radius:22px; background:var(--card); border:1px solid var(--line-soft);
  display:flex; justify-content:space-between; gap:14px; align-items:center; box-shadow:var(--shadow-soft);
  transition:transform .4s var(--ease), box-shadow .4s var(--ease), border-color .4s var(--ease); overflow:hidden; }
.link-card::before{ content:''; position:absolute; left:0; top:0; bottom:0; width:0; background:var(--accent-soft); transition:width .4s var(--ease); }
.link-card:hover{ transform:translateY(-3px); box-shadow:var(--shadow); border-color:var(--accent); }
.link-card:hover::before{ width:5px; }
.link-card .name{ font-size:10.5px; color:var(--muted); letter-spacing:.16em; text-transform:uppercase; font-weight:500; font-family:var(--mono); }
.link-card .value{ margin-top:7px; font-weight:600; font-size:16px; font-family:var(--serif); }
.link-card .arr{ color:var(--accent); font-size:20px; transition:transform .4s var(--ease); }
.link-card:hover .arr{ transform:translate(4px,-4px); }

footer{ margin-top:120px; padding:30px 0 6px; border-top:1px solid var(--line); color:var(--muted); display:flex; justify-content:space-between; gap:16px; font-size:13px; flex-wrap:wrap; }
footer .sig{ font-family:var(--serif); font-style:italic; }

/* ---------- reveal motion ---------- */
.reveal{ opacity:0; transform:translateY(26px); transition:opacity 1s var(--ease), transform 1s var(--ease); }
.reveal.in{ opacity:1; transform:none; }
.hero .reveal{ transition:opacity 1.1s var(--ease), transform 1.1s var(--ease); }
[data-d="1"]{ transition-delay:.08s; } [data-d="2"]{ transition-delay:.16s; }
[data-d="3"]{ transition-delay:.24s; } [data-d="4"]{ transition-delay:.32s; }
[data-d="5"]{ transition-delay:.40s; } [data-d="6"]{ transition-delay:.48s; }

@media (max-width: 980px){
  .hero,.about-grid,.exp-grid,.contact-grid,.cards{ grid-template-columns:1fr; }
  .project.lead{ grid-column:auto; }
  .project.lead .lead-grid{ grid-template-columns:1fr; gap:0; }
  .hero{ min-height:auto; padding-top:48px; gap:28px; }
  .hero-canvas{ top:0; height:100%; }
  .hero::after{ inset:0 -18px 0; }
  .nav-links{ display:none; }
  .section{ padding-top:96px; }
}
@media (max-width: 560px){
  .wrap{ padding:0 18px 84px; }
  .nav{ padding:0 16px; }
  .hero h1{ font-size:64px; }
  .project{ padding:24px; }
  .viz{ height:170px; }
  .about-copy{ padding:26px; }
}
@media (prefers-reduced-motion: reduce){
  *{ animation:none !important; }
  .reveal{ opacity:1; transform:none; transition:none; }
  html{ scroll-behavior:auto; }
}
```

- [ ] **Step 4: Verify in browser** — `Start-Process index.html`. Check: dark charcoal background; all text readable (no dark-on-dark); nav glass is dark; gold accents; grain subtle; no horizontal scrollbar at 1280px and 390px widths; hover states on pills/buttons/cards work.

- [ ] **Step 5: Commit**

```powershell
git add index.html; git commit -m "Restyle to dark studio theme with mono lab typography"
```

---

### Task 2: Lab canvas engine + hero flow field

**Files:**
- Modify: `index.html` — hero section markup (add canvas), replace the whole existing `<script>` block (keep Cloudflare script untouched)

**Interfaces:**
- Produces: `Lab.register(canvas, factory, hoverEl)` where `factory(ctx, {w,h})` returns `{ update(dt), draw(), resize(size), renderStatic(), onPointer?(x,y,inside) }`; `Lab.reduced` (bool), `Lab.fine` (bool). Tasks 3–7 consume this exact API.

- [ ] **Step 1: Add hero canvas markup** — inside `<section class="hero" id="top">`, as its first child:

```html
<canvas class="hero-canvas" id="heroCanvas" aria-hidden="true"></canvas>
```

- [ ] **Step 2: Replace the main `<script>` block** (the existing IIFE with progress/parallax/reveal) with:

```html
<script>
(function(){
  'use strict';

  /* ================= Lab engine ================= */
  var Lab = (function(){
    var reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
    var fine = window.matchMedia('(pointer: fine)').matches;
    var items = [];
    var running = false, last = 0;

    function fit(canvas){
      var dpr = Math.min(window.devicePixelRatio || 1, 2);
      var r = canvas.getBoundingClientRect();
      var w = Math.max(1, r.width), h = Math.max(1, r.height);
      canvas.width = Math.round(w * dpr);
      canvas.height = Math.round(h * dpr);
      canvas.getContext('2d').setTransform(dpr, 0, 0, dpr, 0, 0);
      return { w: w, h: h };
    }
    function loop(t){
      if (!last) last = t;
      var dt = Math.min(0.05, (t - last) / 1000);
      last = t;
      var any = false;
      for (var i = 0; i < items.length; i++){
        if (!items[i].active) continue;
        any = true;
        items[i].v.update(dt);
        items[i].v.draw();
      }
      if (any){ requestAnimationFrame(loop); } else { running = false; last = 0; }
    }
    function wake(){
      if (!running && !reduced){ running = true; last = 0; requestAnimationFrame(loop); }
    }
    function register(canvas, factory, hoverEl){
      if (!canvas || !canvas.getContext) return null;
      var size = fit(canvas);
      var v = factory(canvas.getContext('2d'), size);
      var item = { v: v, active: false };
      var rT;
      window.addEventListener('resize', function(){
        clearTimeout(rT);
        rT = setTimeout(function(){
          v.resize(fit(canvas));
          if (reduced) v.renderStatic();
        }, 150);
      });
      if (reduced){ v.renderStatic(); return v; }
      if ('IntersectionObserver' in window){
        new IntersectionObserver(function(es){
          for (var i = 0; i < es.length; i++) item.active = es[i].isIntersecting;
          wake();
        }, { rootMargin: '100px' }).observe(canvas);
      } else { item.active = true; }
      if (v.onPointer){   /* touch pointers included — spec: hero reacts to touch drag */
        var target = hoverEl || canvas;
        target.addEventListener('pointermove', function(e){
          var r = canvas.getBoundingClientRect();
          v.onPointer(e.clientX - r.left, e.clientY - r.top, true);
        }, { passive: true });
        target.addEventListener('pointerleave', function(){ v.onPointer(-1, -1, false); });
      }
      items.push(item);
      wake();
      return v;
    }
    return { register: register, reduced: reduced, fine: fine };
  })();

  /* ================= hero flow field ================= */
  function makeHeroField(ctx, size){
    var W = size.w, H = size.h;
    var ps = [], t = 0, px = -1e4, py = -1e4;
    function count(){ return Math.round(Math.min(220, (W * H) / 9000)); }
    function reset(p){
      p.x = Math.random() * W; p.y = Math.random() * H;
      p.life = 4 + Math.random() * 6;
      p.speed = 14 + Math.random() * 26;
      p.gold = Math.random() < 0.86;
    }
    function build(){
      ps.length = 0;
      for (var i = 0; i < count(); i++){ var p = {}; reset(p); p.life *= Math.random(); ps.push(p); }
    }
    build();
    function field(x, y){
      return Math.sin(x * 0.0016 + t * 0.28) + Math.cos(y * 0.0019 - t * 0.22)
           + Math.sin((x + y) * 0.0008 + t * 0.11);
    }
    function step(dt){
      t += dt;
      for (var i = 0; i < ps.length; i++){
        var p = ps[i];
        var a = field(p.x, p.y) * Math.PI * 0.75;
        var vx = Math.cos(a) * p.speed, vy = Math.sin(a) * p.speed;
        var dx = px - p.x, dy = py - p.y, d2 = dx * dx + dy * dy;
        if (d2 < 32400){
          var d = Math.sqrt(d2) || 1, f = (1 - d / 180) * 60;
          vx += (dx / d) * f; vy += (dy / d) * f;
        }
        p.x += vx * dt; p.y += vy * dt; p.life -= dt;
        if (p.life <= 0 || p.x < -20 || p.x > W + 20 || p.y < -20 || p.y > H + 20) reset(p);
      }
    }
    function paint(fade){
      ctx.fillStyle = 'rgba(13,12,10,' + fade + ')';
      ctx.fillRect(0, 0, W, H);
      for (var i = 0; i < ps.length; i++){
        var p = ps[i];
        var a = Math.max(0, Math.min(0.55, p.life * 0.14));
        ctx.fillStyle = p.gold ? 'rgba(201,166,104,' + a + ')' : 'rgba(111,195,214,' + a * 0.8 + ')';
        ctx.fillRect(p.x, p.y, 1.6, 1.6);
      }
    }
    return {
      update: step,
      draw: function(){ paint(0.085); },
      resize: function(s){ W = s.w; H = s.h; build(); },
      renderStatic: function(){ for (var k = 0; k < 240; k++) step(1 / 60); paint(1); },
      onPointer: function(x, y, inside){ px = inside ? x : -1e4; py = inside ? y : -1e4; }
    };
  }
  Lab.register(document.getElementById('heroCanvas'), makeHeroField, document.documentElement);

  /* ================= scroll: progress + nav ================= */
  var progress = document.getElementById('progress');
  var nav = document.getElementById('nav');
  var ticking = false;
  function onScroll(){
    var st = window.scrollY || document.documentElement.scrollTop;
    var h = document.documentElement.scrollHeight - window.innerHeight;
    progress.style.width = (h > 0 ? (st / h) * 100 : 0) + '%';
    nav.classList.toggle('scrolled', st > 12);
    ticking = false;
  }
  window.addEventListener('scroll', function(){
    if (!ticking){ window.requestAnimationFrame(onScroll); ticking = true; }
  }, { passive: true });
  onScroll();

  /* ================= reveal on scroll ================= */
  var io = new IntersectionObserver(function(entries){
    entries.forEach(function(en){
      if (en.isIntersecting){ en.target.classList.add('in'); io.unobserve(en.target); }
    });
  }, { threshold: 0.12, rootMargin: '0px 0px -8% 0px' });
  document.querySelectorAll('.reveal').forEach(function(el){ io.observe(el); });
  window.addEventListener('load', function(){
    document.querySelectorAll('.hero .reveal').forEach(function(el){ el.classList.add('in'); });
  });

  window.__lab = { Lab: Lab };   /* debug handle */
})();
</script>
```

Note: later tasks add factories *inside this IIFE*, before the `window.__lab` line, and register them there.

- [ ] **Step 3: Verify** — open in browser: particles flow gently behind hero text; moving the mouse bends/attracts nearby particles; text stays readable over the scrim; scroll away from hero then check (DevTools > Performance or simply the rAF via `console.log`) that no canvas work runs when hero is offscreen (quick check: scroll to footer, CPU settles). Console must be error-free.

- [ ] **Step 4: Verify reduced motion** — DevTools → Rendering → emulate `prefers-reduced-motion: reduce` → reload: hero shows one static particle frame, nothing animates.

- [ ] **Step 5: Commit**

```powershell
git add index.html; git commit -m "Add Lab canvas engine and cursor-reactive hero flow field"
```

---

### Task 3: groom·pick scan visual

**Files:**
- Modify: `index.html` — groompick `<article>` (add classes + viz block), script (add factory + register)

**Interfaces:**
- Consumes: `Lab.register(canvas, factory, hoverEl)` from Task 2.

- [ ] **Step 1: Markup** — change `<article class="project lead reveal">` to `<article class="project lead gp reveal" id="proj-gp">` and insert as its first child:

```html
<div class="viz" aria-hidden="true">
  <canvas id="vizGp"></canvas>
  <span class="viz-tag">GP-SCAN · live render</span>
</div>
```

- [ ] **Step 2: Factory** — add inside the main IIFE (before `window.__lab`):

```js
function makeGroompickScan(ctx, size){
  var W = size.w, H = size.h;
  var dots = [], t = 0, scanY = 0, targetY = -1;
  function build(){
    dots.length = 0;
    var cx = W / 2, cy = H / 2, R = Math.min(W, H) * 0.38;
    var i, s;
    for (i = 0; i < 46; i++){
      var a = (i / 46) * Math.PI * 2;
      dots.push({ x: cx + Math.cos(a) * R * 0.72, y: cy + Math.sin(a) * R, z: 'head' });
    }
    for (i = 0; i < 9; i++){
      s = i / 8;
      var by = cy - R * 0.34 - Math.sin(s * Math.PI) * R * 0.07;
      dots.push({ x: cx - R * 0.44 + s * R * 0.3, y: by, z: 'brow' });
      dots.push({ x: cx + R * 0.14 + s * R * 0.3, y: by, z: 'brow' });
    }
    dots.push({ x: cx - R * 0.28, y: cy - R * 0.16, z: 'eye' });
    dots.push({ x: cx + R * 0.28, y: cy - R * 0.16, z: 'eye' });
    for (i = 0; i < 5; i++) dots.push({ x: cx, y: cy - R * 0.08 + i * R * 0.07, z: 'nose' });
    for (i = 0; i < 11; i++){
      s = i / 10;
      dots.push({ x: cx - R * 0.22 + s * R * 0.44, y: cy + R * 0.52 + Math.sin(s * Math.PI) * R * 0.05, z: 'lip' });
    }
    for (i = 0; i < 14; i++){
      dots.push({ x: cx + (Math.random() * 2 - 1) * R * 0.52, y: cy + Math.random() * R * 0.34, z: 'skin' });
    }
  }
  build();
  return {
    update: function(dt){
      t += dt;
      var want = targetY >= 0 ? targetY : (H * 0.5 + Math.sin(t * 0.7) * H * 0.36);
      scanY += (want - scanY) * Math.min(1, dt * (targetY >= 0 ? 8 : 2.2));
    },
    draw: function(){
      ctx.clearRect(0, 0, W, H);
      var g = ctx.createLinearGradient(0, scanY - 26, 0, scanY + 26);
      g.addColorStop(0, 'rgba(224,164,88,0)');
      g.addColorStop(0.5, 'rgba(224,164,88,0.10)');
      g.addColorStop(1, 'rgba(224,164,88,0)');
      ctx.fillStyle = g; ctx.fillRect(0, scanY - 26, W, 52);
      ctx.fillStyle = 'rgba(224,164,88,0.5)'; ctx.fillRect(0, scanY, W, 1);
      ctx.font = '10px "IBM Plex Mono", monospace';
      for (var i = 0; i < dots.length; i++){
        var d = dots[i], near = Math.abs(d.y - scanY) < 16;
        if (near){
          ctx.fillStyle = 'rgba(224,164,88,0.95)';
          ctx.fillRect(d.x - 1.4, d.y - 1.4, 2.8, 2.8);
          if (d.z === 'brow' || d.z === 'skin'){
            ctx.strokeStyle = 'rgba(224,164,88,0.5)';
            ctx.strokeRect(d.x - 5, d.y - 5, 10, 10);
          }
        } else {
          ctx.fillStyle = 'rgba(201,193,180,0.34)';
          ctx.fillRect(d.x - 1, d.y - 1, 2, 2);
        }
      }
      ctx.fillStyle = 'rgba(224,164,88,0.9)';
      ctx.fillText('BROW ' + (0.74 + 0.2 * Math.sin(t * 1.3)).toFixed(2), W - 92, 22);
      ctx.fillText('SKIN ' + (0.68 + 0.22 * Math.cos(t * 0.9)).toFixed(2), W - 92, 38);
    },
    resize: function(s){ W = s.w; H = s.h; build(); },
    renderStatic: function(){ scanY = H * 0.42; this.draw(); },
    onPointer: function(x, y, inside){ targetY = inside ? y : -1; }
  };
}
Lab.register(document.getElementById('vizGp'), makeGroompickScan, document.getElementById('proj-gp'));
```

- [ ] **Step 3: Verify** — dotted face renders centered; amber scanline sweeps up/down; dots near the line glow with square markers on brow/skin; `BROW`/`SKIN` readouts tick; hovering anywhere on the card drags the scanline to cursor Y; leaving resumes the sweep. Reduced-motion → one static frame with scanline at 42%.

- [ ] **Step 4: Commit**

```powershell
git add index.html; git commit -m "Add groompick scanline face visual"
```

---

### Task 4: NewsLens sorting visual

**Files:**
- Modify: `index.html` — NewsLens `<article>` + script

**Interfaces:**
- Consumes: `Lab.register` from Task 2.

- [ ] **Step 1: Markup** — change the NewsLens article tag to `<article class="project nl reveal" data-d="1" id="proj-nl">` and insert as first child:

```html
<div class="viz" aria-hidden="true">
  <canvas id="vizNl"></canvas>
  <span class="viz-tag">NL-SORT · live render</span>
</div>
```

- [ ] **Step 2: Factory** — add inside the IIFE:

```js
function makeNewslensSort(ctx, size){
  var W = size.w, H = size.h;
  var LANES = [
    { k: 'FACT',    c: '111,195,214' },
    { k: 'CLAIM',   c: '224,192,122' },
    { k: 'OPINION', c: '224,139,90' },
    { k: 'FRAMING', c: '182,143,214' }
  ];
  var ps = [], boost = 1, hot = -1;
  function laneY(i){ return H * (0.2 + i * 0.2); }
  function reset(p){
    p.x = -30 - Math.random() * W * 0.3;
    p.y = H * (0.12 + Math.random() * 0.76);
    p.lane = Math.floor(Math.random() * 4);
    p.w = 10 + Math.random() * 22;
    p.speed = W * (0.14 + Math.random() * 0.1);
  }
  for (var i = 0; i < 26; i++){ var p = {}; reset(p); p.x = Math.random() * W; ps.push(p); }
  return {
    update: function(dt){
      for (var i = 0; i < ps.length; i++){
        var p = ps[i];
        p.x += p.speed * dt * boost * (hot === p.lane ? 1.6 : 1);
        if (p.x > W * 0.52) p.y += (laneY(p.lane) - p.y) * Math.min(1, dt * 4);
        if (p.x > W + 30) reset(p);
      }
    },
    draw: function(){
      ctx.clearRect(0, 0, W, H);
      ctx.font = '10px "IBM Plex Mono", monospace';
      for (var l = 0; l < 4; l++){
        var y = laneY(l), h = hot === l;
        ctx.strokeStyle = 'rgba(' + LANES[l].c + ',' + (h ? 0.4 : 0.14) + ')';
        ctx.beginPath(); ctx.moveTo(W * 0.5, y); ctx.lineTo(W - 14, y); ctx.stroke();
        ctx.fillStyle = 'rgba(' + LANES[l].c + ',' + (h ? 1 : 0.75) + ')';
        ctx.fillText(LANES[l].k, W * 0.52, y - 6);
      }
      ctx.strokeStyle = 'rgba(239,234,226,0.1)';
      ctx.setLineDash([3, 5]);
      ctx.beginPath(); ctx.moveTo(W * 0.5, H * 0.08); ctx.lineTo(W * 0.5, H * 0.92); ctx.stroke();
      ctx.setLineDash([]);
      for (var i = 0; i < ps.length; i++){
        var p = ps[i], sorted = p.x > W * 0.52;
        ctx.fillStyle = sorted ? 'rgba(' + LANES[p.lane].c + ',0.9)' : 'rgba(201,193,180,0.5)';
        ctx.fillRect(p.x, p.y, p.w, 2);
      }
    },
    resize: function(s){ W = s.w; H = s.h; },
    renderStatic: function(){ for (var k = 0; k < 200; k++) this.update(1 / 60); this.draw(); },
    onPointer: function(x, y, inside){
      boost = inside ? 1.5 : 1; hot = -1;
      if (inside && x > W * 0.4){
        var best = 1e9;
        for (var l = 0; l < 4; l++){
          var d = Math.abs(y - laneY(l));
          if (d < best){ best = d; hot = l; }
        }
      }
    }
  };
}
Lab.register(document.getElementById('vizNl'), makeNewslensSort, document.getElementById('proj-nl'));
```

- [ ] **Step 3: Verify** — gray sentence-dashes flow in from the left, cross the dashed divider, take on lane colors and align into FACT/CLAIM/OPINION/FRAMING rows; hovering near a lane highlights and accelerates it. Reduced-motion → static sorted frame.

- [ ] **Step 4: Commit**

```powershell
git add index.html; git commit -m "Add NewsLens sentence-sorting visual"
```

---

### Task 5: 플러피 rescue visual

**Files:**
- Modify: `index.html` — 플러피 `<article>` + script

**Interfaces:**
- Consumes: `Lab.register` from Task 2.

- [ ] **Step 1: Markup** — change the 플러피 article tag to `<article class="project fl reveal" data-d="2" id="proj-fl">` and insert as first child:

```html
<div class="viz" aria-hidden="true">
  <canvas id="vizFl"></canvas>
  <span class="viz-tag">FL-RESCUE · live render</span>
</div>
```

- [ ] **Step 2: Factory** — add inside the IIFE:

```js
function makeFluffyRescue(ctx, size){
  var W = size.w, H = size.h;
  var ps = [], t = 0, hover = false;
  function reset(p){
    p.x = -10 - Math.random() * W * 0.25;
    p.y = H * (0.16 + Math.random() * 0.42);
    p.r = 2.5 + Math.random() * 2.5;
    p.speed = W * (0.09 + Math.random() * 0.07);
    p.saved = Math.random() < 0.8;
    p.state = 'drift'; p.a = 1; p.rest = 0;
  }
  for (var i = 0; i < 22; i++){ var p = {}; reset(p); p.x = Math.random() * W * 0.6; ps.push(p); }
  return {
    update: function(dt){
      t += dt;
      var bx = W * 0.34, by = H * 0.74, lx = W * 0.8;
      for (var i = 0; i < ps.length; i++){
        var p = ps[i];
        if (p.state === 'drift'){
          p.x += p.speed * dt;
          if ((p.saved || hover) && p.x > W * 0.55) p.state = 'save';
          else if (p.x >= lx) p.state = 'lost';
        } else if (p.state === 'save'){
          p.x += (bx - p.x) * Math.min(1, dt * 3);
          p.y += (by - p.y) * Math.min(1, dt * 3);
          if (Math.abs(p.x - bx) < 8 && Math.abs(p.y - by) < 8) p.state = 'rest';
        } else if (p.state === 'rest'){
          p.rest += dt;
          if (p.rest > 1.6){ p.a -= dt * 2; if (p.a <= 0) reset(p); }
        } else if (p.state === 'lost'){
          p.a -= dt * 2.4;
          if (p.a <= 0) reset(p);
        }
      }
    },
    draw: function(){
      ctx.clearRect(0, 0, W, H);
      var lx = W * 0.8, bx = W * 0.34, by = H * 0.74;
      ctx.font = '10px "IBM Plex Mono", monospace';
      ctx.strokeStyle = 'rgba(224,139,90,0.3)';
      ctx.setLineDash([4, 6]);
      ctx.beginPath(); ctx.moveTo(lx, H * 0.1); ctx.lineTo(lx, H * 0.9); ctx.stroke();
      ctx.setLineDash([]);
      ctx.fillStyle = 'rgba(224,139,90,0.8)';
      ctx.fillText('CLOSE 21:00', lx + 8, H * 0.16);
      ctx.strokeStyle = 'rgba(224,139,90,0.6)';
      ctx.beginPath(); ctx.arc(lx + 34, H * 0.42, 11, 0, Math.PI * 2); ctx.stroke();
      ctx.beginPath(); ctx.moveTo(lx + 34, H * 0.42);
      ctx.lineTo(lx + 34 + Math.cos(t * 0.9 - Math.PI / 2) * 8, H * 0.42 + Math.sin(t * 0.9 - Math.PI / 2) * 8);
      ctx.stroke();
      ctx.strokeStyle = 'rgba(239,234,226,0.4)';
      ctx.lineWidth = 1.5;
      ctx.beginPath(); ctx.arc(bx, by, 20, 0.15 * Math.PI, 0.85 * Math.PI); ctx.stroke();
      ctx.lineWidth = 1;
      ctx.fillStyle = 'rgba(201,193,180,0.6)';
      ctx.fillText('RESCUED', bx - 26, by + 36);
      for (var i = 0; i < ps.length; i++){
        var p = ps[i];
        var glow = p.state === 'rest' ? 0.95 : 0.7;
        ctx.fillStyle = 'rgba(224,139,90,' + (p.a * glow) + ')';
        ctx.beginPath(); ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2); ctx.fill();
        if (p.state === 'rest'){
          ctx.strokeStyle = 'rgba(224,139,90,' + (p.a * 0.4) + ')';
          ctx.beginPath(); ctx.arc(p.x, p.y, p.r + 3, 0, Math.PI * 2); ctx.stroke();
        }
      }
    },
    resize: function(s){ W = s.w; H = s.h; },
    renderStatic: function(){ for (var k = 0; k < 200; k++) this.update(1 / 60); this.draw(); },
    onPointer: function(x, y, inside){ hover = inside; }
  };
}
Lab.register(document.getElementById('vizFl'), makeFluffyRescue, document.getElementById('proj-fl'));
```

- [ ] **Step 3: Verify** — bread dots drift toward the dashed "CLOSE 21:00" line with ticking clock; most curve away into the RESCUED basket and glow; a few fade at the line; hovering the card rescues everything. Reduced-motion → static frame.

- [ ] **Step 4: Commit**

```powershell
git add index.html; git commit -m "Add Fluffy bread-rescue visual"
```

---

### Task 6: 뭐먹을래 roulette visual

**Files:**
- Modify: `index.html` — 뭐먹을래 `<article>` + script

**Interfaces:**
- Consumes: `Lab.register` from Task 2.

- [ ] **Step 1: Markup** — change the 뭐먹을래 article tag to `<article class="project mw reveal" data-d="1" id="proj-mw">` and insert as first child:

```html
<div class="viz" aria-hidden="true">
  <canvas id="vizMw"></canvas>
  <span class="viz-tag">MW-PICK · live render</span>
</div>
```

- [ ] **Step 2: Factory** — add inside the IIFE:

```js
function makeMwoRoulette(ctx, size){
  var W = size.w, H = size.h;
  var N = 12, ang = 0, vel = 1.4, phase = 'spin', pick = -1, timer = 0, hover = false;
  return {
    update: function(dt){
      timer += dt;
      if (phase === 'spin'){
        vel += (2.2 - vel) * Math.min(1, dt * 2);
        if (timer > 2.4){ phase = 'slow'; }
      } else if (phase === 'slow'){
        vel *= Math.max(0, 1 - dt * 1.5);
        if (vel < 0.06){
          vel = 0; phase = 'lock'; timer = 0;
          var best = 1e9;
          for (var i = 0; i < N; i++){
            var a = ang + (i / N) * Math.PI * 2;
            var d = Math.abs(Math.atan2(Math.sin(a + Math.PI / 2), Math.cos(a + Math.PI / 2)));
            if (d < best){ best = d; pick = i; }
          }
        }
      } else if (phase === 'lock'){
        if (timer > (hover ? 0.7 : 1.6)){ phase = 'spin'; timer = 0; pick = -1; }
      }
      ang += vel * dt * (hover && phase === 'spin' ? 1.5 : 1);
    },
    draw: function(){
      ctx.clearRect(0, 0, W, H);
      var cx = W / 2, cy = H / 2 + 6, R = Math.min(W, H) * 0.32;
      ctx.strokeStyle = 'rgba(239,234,226,0.1)';
      ctx.beginPath(); ctx.arc(cx, cy, R, 0, Math.PI * 2); ctx.stroke();
      ctx.fillStyle = 'rgba(201,166,104,0.9)';
      ctx.beginPath();
      ctx.moveTo(cx, cy - R - 12); ctx.lineTo(cx - 5, cy - R - 20); ctx.lineTo(cx + 5, cy - R - 20);
      ctx.closePath(); ctx.fill();
      ctx.font = '10px "IBM Plex Mono", monospace';
      for (var i = 0; i < N; i++){
        var a = ang + (i / N) * Math.PI * 2;
        var x = cx + Math.cos(a - Math.PI / 2) * R, y = cy + Math.sin(a - Math.PI / 2) * R;
        if (phase === 'lock' && i === pick){
          var pulse = 0.6 + 0.4 * Math.sin(timer * 9);
          ctx.fillStyle = 'rgba(127,214,178,' + pulse + ')';
          ctx.beginPath(); ctx.arc(x, y, 6, 0, Math.PI * 2); ctx.fill();
          ctx.strokeStyle = 'rgba(127,214,178,0.5)';
          ctx.beginPath(); ctx.arc(x, y, 11, 0, Math.PI * 2); ctx.stroke();
          ctx.fillStyle = 'rgba(127,214,178,1)';
          ctx.fillText('PICK', x + 15, y + 3);
        } else {
          ctx.fillStyle = 'rgba(201,193,180,0.55)';
          ctx.beginPath(); ctx.arc(x, y, 3.4, 0, Math.PI * 2); ctx.fill();
        }
      }
    },
    resize: function(s){ W = s.w; H = s.h; },
    renderStatic: function(){ phase = 'lock'; pick = 2; timer = 0.05; this.draw(); },
    onPointer: function(x, y, inside){ hover = inside; }
  };
}
Lab.register(document.getElementById('vizMw'), makeMwoRoulette, document.getElementById('proj-mw'));
```

- [ ] **Step 3: Verify** — menu dots orbit, decelerate, and the dot nearest the top marker flares mint with a `PICK` label, then respins; hover speeds the cycle. Reduced-motion → static locked frame.

- [ ] **Step 4: Commit**

```powershell
git add index.html; git commit -m "Add MwoMeokUllae roulette visual"
```

---

### Task 7: Signature typography, magnetic buttons, lab status line

**Files:**
- Modify: `index.html` — hero markup (status line), script additions

**Interfaces:**
- Consumes: `Lab.reduced`, `Lab.fine` from Task 2; `.lab-status` CSS from Task 1.

- [ ] **Step 1: Lab status line** — in the hero, after the `.pill-row` div, add:

```html
<div class="lab-status reveal" data-d="6">
  <span class="dot"></span>
  <span>LAB STATUS — <b>groom·pick : LIVE</b> · NewsLens : LIVE · next experiment loading…</span>
</div>
```

- [ ] **Step 2: Scroll-linked section titles + magnetic buttons** — add inside the IIFE (before `window.__lab`):

```js
/* scroll-linked section title tracking */
if (!Lab.reduced){
  var heads = Array.prototype.slice.call(document.querySelectorAll('.section-head h2'));
  var headTick = false;
  function headScroll(){
    var vh = window.innerHeight;
    for (var i = 0; i < heads.length; i++){
      var r = heads[i].getBoundingClientRect();
      if (r.bottom < -80 || r.top > vh + 80) continue;
      var pr = Math.max(0, Math.min(1, 1 - r.top / vh));
      heads[i].style.letterSpacing = ((pr - 0.5) * 2) + 'px';
      heads[i].style.transform = 'translateY(' + ((0.5 - pr) * 16) + 'px)';
    }
    headTick = false;
  }
  window.addEventListener('scroll', function(){
    if (!headTick){ requestAnimationFrame(headScroll); headTick = true; }
  }, { passive: true });
  headScroll();
}

/* magnetic buttons */
if (Lab.fine && !Lab.reduced){
  document.querySelectorAll('.btn').forEach(function(el){
    el.addEventListener('pointermove', function(e){
      var r = el.getBoundingClientRect();
      el.style.transform = 'translate(' + (e.clientX - r.left - r.width / 2) * 0.18 + 'px,'
        + (e.clientY - r.top - r.height / 2) * 0.3 + 'px)';
    }, { passive: true });
    el.addEventListener('pointerleave', function(){ el.style.transform = ''; });
  });
}
```

- [ ] **Step 3: Verify** — section titles subtly shift tracking/position while scrolling (text stays fully legible); buttons lean toward the cursor and snap back; status line pulses green in hero. Reduced-motion → none of these run.

- [ ] **Step 4: Commit**

```powershell
git add index.html; git commit -m "Add scroll-linked titles, magnetic buttons, lab status line"
```

---

### Task 8: Final QA pass

**Files:**
- Modify: `index.html` (fixes only, if QA finds issues)

- [ ] **Step 1: Full-page check at 1280px** — all 6 sections render, all 5 visuals animate, console clean.
- [ ] **Step 2: Mobile 390px** — single column, viz height 170px, no horizontal scroll, hero canvas fills, no magnetic/cursor effects on touch emulation.
- [ ] **Step 3: Reduced motion** — every canvas shows a static frame; no animations anywhere.
- [ ] **Step 4: Links** — click all: groom-pick.vercel.app, newslens GitHub Pages, github.com/lafley-lucas, nav anchors.
- [ ] **Step 5: Performance spot-check** — DevTools Performance: scroll the full page; no long tasks > 50ms sustained from canvas work; offscreen visuals idle.
- [ ] **Step 6: Content diff** — confirm no Korean copy changed: `git diff 33029ad -- index.html` should show no deletions of Korean sentences other than relocations.
- [ ] **Step 7: Commit any fixes**

```powershell
git add index.html; git commit -m "QA fixes for Builder's Lab redesign"
```
