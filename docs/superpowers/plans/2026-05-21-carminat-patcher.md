# Carminat TomTom Clock Patcher — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Single `index.html` page that patches the `PNDNavigator` binary in-browser and guides users through both cases (has file / doesn't have file).

**Architecture:** One self-contained `index.html` with inline CSS and JS. Pure browser APIs (`FileReader`, `DataView`, `Blob`). No build step, no dependencies. GitHub Pages serves it directly from `main` branch root.

**Tech Stack:** HTML5, CSS3 (custom properties), vanilla JS (ES2017), GitHub Pages

---

## Task 1: Project scaffold

**Files:**
- Create: `index.html`
- Create: `README.md`

- [ ] **Step 1: Create the HTML shell**

Create `/Users/ffleureau/workspace/oss/renault-horloge/index.html`:

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Réparer l'heure Carminat TomTom</title>
</head>
<body>
  <p>À venir…</p>
</body>
</html>
```

- [ ] **Step 2: Create README.md**

Create `/Users/ffleureau/workspace/oss/renault-horloge/README.md`:

```markdown
# Réparer l'heure Carminat TomTom

Outil web pour patcher le fichier `PNDNavigator` sur la carte SD du système
Carminat TomTom (Clio II / Mégane II / Laguna II).

## Déploiement GitHub Pages

1. Créer un dépôt GitHub (ex. `renault-horloge`)
2. Pousser `main` : `git push -u origin main`
3. Settings → Pages → Source : `main` / `/ (root)`
4. Partager l'URL : `https://<username>.github.io/renault-horloge/`

## Fonctionnement

- 100 % côté client — le fichier ne quitte jamais l'ordinateur de l'utilisateur
- Patch : offset `0x00402928`, remplace `0xD6` par `0xB9`
```

- [ ] **Step 3: Verify HTML opens in browser**

Open `index.html` in any browser.
Expected: page shows "À venir…"

- [ ] **Step 4: Commit**

```bash
git add index.html README.md
git commit -m "chore: project scaffold"
```

---

## Task 2: CSS — Renault dark theme

**Files:**
- Modify: `index.html` (add `<style>` in `<head>`)

- [ ] **Step 1: Replace `<head>` content with full CSS**

Replace the `<head>` block in `index.html`:

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Réparer l'heure Carminat TomTom</title>
  <style>
    :root {
      --bg: #111111;
      --card: #1e1e1e;
      --accent: #FFCC00;
      --text: #ffffff;
      --text-muted: #aaaaaa;
      --success: #4caf50;
      --error: #f44336;
      --warning: #ff9800;
    }

    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      background: var(--bg);
      color: var(--text);
      font-family: -apple-system, 'Segoe UI', Roboto, sans-serif;
      min-height: 100vh;
      padding: 2rem 1rem 3rem;
      line-height: 1.5;
    }

    .container { max-width: 640px; margin: 0 auto; }

    /* Header */
    header { text-align: center; margin-bottom: 2.5rem; }
    header h1 { font-size: 1.6rem; font-weight: 700; color: var(--accent); margin-bottom: 0.4rem; }
    header p { color: var(--text-muted); font-size: 0.9rem; }

    /* Cards */
    .card {
      background: var(--card);
      border-radius: 12px;
      padding: 1.75rem;
      margin-bottom: 1.5rem;
      border: 1px solid #2a2a2a;
    }
    .card-title {
      font-size: 1rem;
      font-weight: 600;
      color: var(--accent);
      margin-bottom: 1.25rem;
    }

    /* Drop zone */
    .drop-zone {
      border: 2px dashed #444;
      border-radius: 8px;
      padding: 2.25rem 1rem;
      text-align: center;
      cursor: pointer;
      transition: border-color 0.15s, background 0.15s;
      color: var(--text-muted);
      font-size: 0.9rem;
      margin-bottom: 0.75rem;
      user-select: none;
    }
    .drop-zone:hover,
    .drop-zone.drag-over { border-color: var(--accent); background: rgba(255,204,0,0.04); color: var(--text); }
    .drop-zone .icon { font-size: 2rem; margin-bottom: 0.5rem; }

    /* File name */
    .file-name { font-size: 0.85rem; color: var(--text-muted); margin-bottom: 1rem; min-height: 1.3em; }
    .file-name.loaded { color: var(--accent); }

    /* Button */
    button {
      background: var(--accent);
      color: #111;
      border: none;
      border-radius: 8px;
      padding: 0.8rem 1.5rem;
      font-size: 1rem;
      font-weight: 700;
      cursor: pointer;
      width: 100%;
      transition: opacity 0.15s, transform 0.1s;
    }
    button:disabled { opacity: 0.35; cursor: not-allowed; }
    button:not(:disabled):hover { opacity: 0.88; }
    button:not(:disabled):active { transform: scale(0.98); }

    /* Status banner */
    .status {
      margin-top: 1rem;
      padding: 0.75rem 1rem;
      border-radius: 8px;
      font-size: 0.9rem;
      display: none;
    }
    .status.success { background: rgba(76,175,80,0.12); border: 1px solid var(--success); color: var(--success); }
    .status.error   { background: rgba(244,67,54,0.12);  border: 1px solid var(--error);   color: var(--error);   }
    .status.warning { background: rgba(255,152,0,0.12);  border: 1px solid var(--warning);  color: var(--warning); }

    /* Divider */
    .divider { text-align: center; color: var(--text-muted); font-size: 0.75rem; letter-spacing: 0.08em; text-transform: uppercase; margin: 0.25rem 0 1.5rem; }

    /* Numbered steps */
    .steps { list-style: none; counter-reset: step; }
    .steps li { display: flex; gap: 1rem; margin-bottom: 1.25rem; align-items: flex-start; }
    .steps li::before {
      counter-increment: step;
      content: counter(step);
      background: var(--accent);
      color: #111;
      min-width: 1.75rem;
      height: 1.75rem;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: 700;
      font-size: 0.8rem;
      flex-shrink: 0;
      margin-top: 0.1rem;
    }
    .steps li p { font-size: 0.9rem; }
    .steps li code {
      background: #2a2a2a;
      padding: 0.1rem 0.35rem;
      border-radius: 4px;
      font-size: 0.82rem;
      font-family: monospace;
      color: var(--accent);
    }
    .steps li a { color: var(--accent); word-break: break-all; }
    .steps li a:hover { text-decoration: underline; }

    /* Warning box inside steps */
    .warn-box {
      margin-top: 0.5rem;
      background: rgba(255,152,0,0.08);
      border: 1px solid var(--warning);
      border-radius: 6px;
      padding: 0.55rem 0.85rem;
      font-size: 0.82rem;
      color: var(--warning);
      line-height: 1.5;
    }

    /* Footer */
    footer { text-align: center; color: var(--text-muted); font-size: 0.78rem; margin-top: 2rem; }
  </style>
</head>
```

- [ ] **Step 2: Verify theme in browser**

Open `index.html`.
Expected: dark background, "À venir…" still visible (no layout yet).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "style: add Renault dark theme CSS"
```

---

## Task 3: HTML structure

**Files:**
- Modify: `index.html` (replace `<body>` content)

- [ ] **Step 1: Replace `<body>` with full markup (no JS yet)**

Replace the `<body>` in `index.html`:

```html
<body>
  <div class="container">

    <header>
      <h1>⏰ Réparer l'heure Carminat TomTom</h1>
      <p>Clio II · Mégane II · Laguna II — Patch PNDNavigator</p>
    </header>

    <!-- ── Cas 1 ─────────────────────────────── -->
    <div class="card">
      <div class="card-title">📁 Cas 1 — J'ai le fichier PNDNavigator</div>

      <div class="drop-zone" id="dropZone" role="button" tabindex="0"
           aria-label="Zone de dépôt : glissez PNDNavigator ici ou cliquez pour choisir">
        <div class="icon">📂</div>
        <div>Glissez-déposez <strong>PNDNavigator</strong> ici</div>
        <div style="margin-top:0.4rem;font-size:0.8rem;opacity:0.7">ou cliquez pour choisir</div>
        <input type="file" id="fileInput" style="display:none" aria-hidden="true">
      </div>

      <div class="file-name" id="fileName">Aucun fichier sélectionné</div>

      <button id="patchBtn" disabled>🔧 Patcher et télécharger</button>

      <div class="status" id="status" role="alert" aria-live="polite"></div>
    </div>

    <div class="divider">— ou —</div>

    <!-- ── Cas 2 ─────────────────────────────── -->
    <div class="card">
      <div class="card-title">📥 Cas 2 — Je n'ai pas le fichier PNDNavigator</div>
      <ol class="steps">
        <li>
          <p>
            Téléchargez le fichier navcore :<br>
            <a href="https://download.tomtom.com/sweet/navcore/navcore_8843.3130258.Carminat_TomTom.cab"
               target="_blank" rel="noopener noreferrer">
              navcore_8843.3130258.Carminat_TomTom.cab
            </a>
          </p>
        </li>
        <li>
          <p>Dézippez le fichier <code>.cab</code> (WinRAR, 7-Zip, ou l'explorateur Windows).</p>
        </li>
        <li>
          <p>Copiez <strong>TOUS</strong> les fichiers directement à la <strong>racine</strong> de la carte SD.</p>
          <div class="warn-box">
            ⚠️ Ne mettez <strong>pas</strong> les fichiers dans le sous-dossier
            <code>IA0000700006swlimage0</code> — copiez-les directement à la racine.
          </div>
        </li>
        <li>
          <p>Si Windows demande de remplacer des fichiers existants → cliquez <strong>Ignorer</strong>.</p>
        </li>
        <li>
          <p>Renommez <code>~tsystem</code> en <code>ttsystem</code> (retirez le <code>~</code>).</p>
        </li>
        <li>
          <p>Vous avez maintenant <code>PNDNavigator</code> sur la carte SD. Utilisez l'outil <strong>Cas 1</strong> ci-dessus pour le patcher.</p>
        </li>
      </ol>
    </div>

    <footer>
      🔒 Traitement 100&nbsp;% local — votre fichier ne quitte jamais votre ordinateur.
    </footer>

  </div>
  <!-- JS goes here in Task 4 -->
</body>
```

- [ ] **Step 2: Verify layout in browser**

Open `index.html`.
Expected:
- Dark background, yellow header title
- Cas 1 card: dashed drop zone, grey "Aucun fichier sélectionné", disabled yellow button
- Divider "— ou —"
- Cas 2 card: 6 numbered steps with yellow circles, orange warning box on step 3, link on step 1
- Footer with lock icon

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add page HTML structure"
```

---

## Task 4: Patch logic — pure function + inline tests

**Files:**
- Modify: `index.html` (add `<script>` before `</body>`)

This task writes the core `patchBuffer` function as a pure, testable unit and verifies it with `console.assert` before wiring to the DOM.

- [ ] **Step 1: Add patch logic + self-tests script**

Add the following `<script>` block just before `</body>` (replace the `<!-- JS goes here -->` comment):

```html
  <script>
    // ── Constants ──────────────────────────────────────────
    const PATCH_OFFSET = 0x00402928;   // row 0x00402920, column 8 (0-indexed)
    const BYTE_BEFORE  = 0xD6;
    const BYTE_AFTER   = 0xB9;
    const MIN_SIZE     = PATCH_OFFSET + 1; // 4204329 bytes

    /**
     * Inspect / patch an ArrayBuffer.
     *
     * Returns { status, buffer? }
     *   status: 'ok' | 'too_small' | 'already_patched' | 'wrong_byte'
     *   buffer: patched ArrayBuffer (only present when status === 'ok')
     *   byte:   actual byte found (only present when status === 'wrong_byte')
     */
    function patchBuffer(buffer) {
      if (buffer.byteLength < MIN_SIZE) {
        return { status: 'too_small' };
      }
      const copy = buffer.slice(0);          // work on a copy
      const view = new DataView(copy);
      const byte = view.getUint8(PATCH_OFFSET);
      if (byte === BYTE_AFTER) {
        return { status: 'already_patched' };
      }
      if (byte !== BYTE_BEFORE) {
        return { status: 'wrong_byte', byte };
      }
      view.setUint8(PATCH_OFFSET, BYTE_AFTER);
      return { status: 'ok', buffer: copy };
    }

    // ── Inline tests (remove after verifying) ───────────────
    (function runTests() {
      function makeBuffer(size, byteAt) {
        const buf  = new ArrayBuffer(size);
        const view = new DataView(buf);
        if (byteAt !== undefined) view.setUint8(PATCH_OFFSET, byteAt);
        return buf;
      }

      // too_small
      const r1 = patchBuffer(new ArrayBuffer(100));
      console.assert(r1.status === 'too_small', 'FAIL too_small');

      // already_patched
      const r2 = patchBuffer(makeBuffer(MIN_SIZE, BYTE_AFTER));
      console.assert(r2.status === 'already_patched', 'FAIL already_patched');

      // wrong_byte
      const r3 = patchBuffer(makeBuffer(MIN_SIZE, 0xAA));
      console.assert(r3.status === 'wrong_byte', 'FAIL wrong_byte status');
      console.assert(r3.byte  === 0xAA,          'FAIL wrong_byte value');

      // ok — patch applied
      const r4 = patchBuffer(makeBuffer(MIN_SIZE, BYTE_BEFORE));
      console.assert(r4.status === 'ok', 'FAIL ok status');
      console.assert(new DataView(r4.buffer).getUint8(PATCH_OFFSET) === BYTE_AFTER, 'FAIL ok byte');

      // ok — original buffer NOT mutated
      const orig = makeBuffer(MIN_SIZE, BYTE_BEFORE);
      patchBuffer(orig);
      console.assert(new DataView(orig).getUint8(PATCH_OFFSET) === BYTE_BEFORE, 'FAIL immutability');

      console.log('%c✅ patchBuffer: tous les tests OK', 'color: #4caf50; font-weight: bold');
    })();
  </script>
```

- [ ] **Step 2: Run tests in browser console**

Open `index.html`, open DevTools → Console.
Expected output: `✅ patchBuffer: tous les tests OK`
No red `FAIL` or `Assertion failed` messages.

If any assertion fails: re-read the constants and the `patchBuffer` logic before continuing.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add patchBuffer logic with inline tests"
```

---

## Task 5: DOM wiring — file selection + patch + download

**Files:**
- Modify: `index.html` (extend `<script>` block)

- [ ] **Step 1: Add DOM wiring after the inline tests**

Append the following inside the existing `<script>` block, after the `runTests` IIFE:

```javascript
    // ── DOM refs ────────────────────────────────────────────
    const dropZone  = document.getElementById('dropZone');
    const fileInput = document.getElementById('fileInput');
    const fileNameEl= document.getElementById('fileName');
    const patchBtn  = document.getElementById('patchBtn');
    const statusEl  = document.getElementById('status');

    let selectedFile = null;

    // ── File selection ───────────────────────────────────────
    dropZone.addEventListener('click', () => fileInput.click());
    dropZone.addEventListener('keydown', e => {
      if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); fileInput.click(); }
    });
    fileInput.addEventListener('change', () => setFile(fileInput.files[0]));

    // Drag-and-drop
    dropZone.addEventListener('dragover', e => { e.preventDefault(); dropZone.classList.add('drag-over'); });
    dropZone.addEventListener('dragleave', () => dropZone.classList.remove('drag-over'));
    dropZone.addEventListener('drop', e => {
      e.preventDefault();
      dropZone.classList.remove('drag-over');
      setFile(e.dataTransfer.files[0]);
    });

    function setFile(file) {
      if (!file) return;
      selectedFile = file;
      fileNameEl.textContent = '📄 ' + file.name;
      fileNameEl.classList.add('loaded');
      patchBtn.disabled = false;
      hideStatus();
    }

    // ── Patch button ─────────────────────────────────────────
    patchBtn.addEventListener('click', () => {
      if (!selectedFile) return;
      patchBtn.disabled = true;
      patchBtn.textContent = '⏳ Traitement…';

      const reader = new FileReader();
      reader.onload = e => {
        try {
          const result = patchBuffer(e.target.result);

          if (result.status === 'too_small') {
            showStatus('error',
              `❌ Fichier trop petit (${e.target.result.byteLength} octets) — vérifiez que c'est bien PNDNavigator.`);

          } else if (result.status === 'already_patched') {
            showStatus('warning', '⚠️ Ce fichier est déjà patché — rien à faire.');

          } else if (result.status === 'wrong_byte') {
            const hex = result.byte.toString(16).toUpperCase().padStart(2, '0');
            showStatus('error',
              `❌ Octet inattendu à l'offset 0x${PATCH_OFFSET.toString(16).toUpperCase()} : 0x${hex}. Mauvais fichier ou mauvaise version.`);

          } else if (result.status === 'ok') {
            const blob = new Blob([result.buffer], { type: 'application/octet-stream' });
            const url  = URL.createObjectURL(blob);
            const a    = document.createElement('a');
            a.href     = url;
            a.download = 'PNDNavigator';
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
            showStatus('success',
              "✅ Fichier patché ! Copiez PNDNavigator à la racine de votre carte SD en remplaçant l'original, puis remettez la carte dans votre voiture.");
          }
        } catch (err) {
          showStatus('error', `❌ Erreur inattendue : ${err.message}`);
        } finally {
          patchBtn.disabled = false;
          patchBtn.textContent = '🔧 Patcher et télécharger';
        }
      };
      reader.readAsArrayBuffer(selectedFile);
    });

    // ── Helpers ──────────────────────────────────────────────
    function showStatus(type, msg) {
      statusEl.className = 'status ' + type;
      statusEl.textContent = msg;
      statusEl.style.display = 'block';
    }
    function hideStatus() {
      statusEl.style.display = 'none';
    }
```

- [ ] **Step 2: Manual verification — file selection**

Open `index.html`.
1. Click the drop zone → OS file picker opens. Cancel → nothing changes.
2. Click the drop zone → pick any file → file name appears in yellow below the zone, button becomes active yellow.
3. Press Tab to focus the drop zone, press Enter → OS file picker opens.

- [ ] **Step 3: Manual verification — patch scenarios**

You need a real `PNDNavigator` file or a crafted test file. To craft one:

Open browser console and run:
```javascript
// Create a fake 5MB buffer with 0xD6 at offset 0x00402928
const buf = new ArrayBuffer(5 * 1024 * 1024);
new DataView(buf).setUint8(0x00402928, 0xD6);
const blob = new Blob([buf]);
const a = Object.assign(document.createElement('a'), {href: URL.createObjectURL(blob), download: 'PNDNavigator'});
a.click();
```

Then:
1. Select that downloaded `PNDNavigator` → click "Patcher et télécharger"
   Expected: green banner, browser downloads `PNDNavigator`

2. Select the just-downloaded (patched) file → click button
   Expected: orange "Déjà patché" banner

3. Select any random file (e.g. a photo) → click button
   Expected: red "Octet inattendu" or "trop petit" banner

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: wire DOM to patchBuffer, add file select and download"
```

---

## Task 6: Remove inline tests + final polish

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Remove the inline `runTests` IIFE**

In `index.html`, delete the entire block:

```javascript
    // ── Inline tests (remove after verifying) ───────────────
    (function runTests() {
      // ... (everything up to and including the closing })();
    })();
```

- [ ] **Step 2: Verify console is clean**

Open `index.html` → DevTools Console.
Expected: no output, no errors.

- [ ] **Step 3: Check mobile layout**

Open `index.html` → DevTools → Toggle device toolbar → iPhone SE (375px).
Expected:
- Header text wraps cleanly
- Cards fill width with padding
- Button is full-width and easy to tap
- Step circles stay aligned

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "chore: remove inline tests, final polish"
```

---

## Task 7: GitHub Pages deployment

**Files:**
- Modify: `README.md` (already has deploy instructions — nothing to add)

- [ ] **Step 1: Create GitHub repo and push**

```bash
git branch -m master main
git remote add origin https://github.com/<YOUR_USERNAME>/renault-horloge.git
git push -u origin main
```

- [ ] **Step 2: Enable GitHub Pages**

On GitHub: Settings → Pages → Source: `Deploy from a branch` → Branch: `main` / `/ (root)` → Save.

- [ ] **Step 3: Verify live URL**

Wait ~60 seconds, then open:
`https://<YOUR_USERNAME>.github.io/renault-horloge/`

Expected: same page as local, all features work.

- [ ] **Step 4: Smoke test on mobile browser**

Open the GitHub Pages URL on a phone.
Expected: dark theme renders, drop zone tappable, steps readable.

---

## Verification Checklist (all tasks done)

- [ ] `index.html` opens locally without a server (file:// protocol)
- [ ] Console shows no errors on load
- [ ] Drop zone: click → file picker; drag-and-drop → file name shown
- [ ] Patch button disabled before file selection, enabled after
- [ ] Patching a valid unpatched file → green banner + download
- [ ] Patching an already-patched file → orange warning
- [ ] Patching a wrong/random file → red error
- [ ] Cas 2 TomTom link opens the .cab download in a new tab
- [ ] Page is readable on 375px mobile width
- [ ] GitHub Pages URL serves the page correctly
