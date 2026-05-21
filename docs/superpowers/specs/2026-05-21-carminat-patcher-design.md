# Design: Carminat TomTom Clock Patcher

**Date:** 2026-05-21  
**Status:** Approved  
**Hosting:** GitHub Pages (static, zero build step)

---

## Problem

Renault Clio / Mégane / Laguna owners with Carminat TomTom navigation experience a clock
drift or reset bug. The fix requires patching a single byte in the `PNDNavigator` binary on
the SD card. The original manual procedure requires Notepad++ + HEX Editor plugin and is
inaccessible to non-technical users.

---

## Goal

A single `index.html` page, hosted free on GitHub Pages, that lets any user:

1. Upload their `PNDNavigator` file, patch it in-browser, and download the result.
2. Follow step-by-step instructions if they don't have the file yet (Case 2).

All file processing is 100% client-side. The file never leaves the user's machine.

---

## Architecture

**Deliverable:** one file — `index.html`  
**Dependencies:** none  
**Build step:** none  
**Hosting:** GitHub Pages (push to `main`, enable Pages on `/` root)

---

## Theme

Renault-inspired dark UI:

| Token | Value |
|-------|-------|
| Background | `#111111` |
| Card background | `#1e1e1e` |
| Accent | `#FFCC00` (Renault yellow) |
| Text primary | `#ffffff` |
| Text secondary | `#aaaaaa` |
| Success | `#4caf50` |
| Error | `#f44336` |
| Warning | `#ff9800` |
| Font | System stack (`-apple-system, Segoe UI, sans-serif`) |

---

## Page Structure

```
┌─────────────────────────────────────────────┐
│  HEADER                                     │
│  "Réparer l'heure Carminat TomTom"          │
│  Clio / Mégane / Laguna                     │
├─────────────────────────────────────────────┤
│  CAS 1 — "J'ai le fichier PNDNavigator"     │
│  [Drag & drop zone]                         │
│  [Bouton: Patcher et télécharger]           │
│  [Status message area]                      │
├─────────────────────────────────────────────┤
│  CAS 2 — "Je n'ai pas le fichier"           │
│  ① Download navcore .cab (link)             │
│  ② Unzip → copy ALL files to SD root       │
│  ③ If Windows asks to replace → Ignore     │
│  ④ Rename ~tsystem → ttsystem              │
│  ⑤ Use Cas 1 tool above                    │
└─────────────────────────────────────────────┘
```

---

## Patch Logic

| Property | Value |
|----------|-------|
| Target file | `PNDNavigator` (no extension) |
| Hex row address | `0x00402920` |
| Column (0-indexed) | `8` |
| Absolute byte offset | `0x00402928` |
| Expected byte (before) | `0xD6` |
| Replacement byte (after) | `0xB9` |

### Validation & State Machine

```
user selects file
        │
        ▼
file.size < 0x00402929?
  → ❌ "Fichier trop petit — mauvais fichier ?"
        │ no
        ▼
byte[0x00402928] === 0xB9?
  → ⚠️  "Déjà patché, rien à faire"
        │ no
        ▼
byte[0x00402928] !== 0xD6?
  → ❌ "Octet inattendu (valeur: 0xXX) — mauvais fichier ou mauvaise version"
        │ is 0xD6
        ▼
patch: write 0xB9 at offset 0x00402928
        │
        ▼
✅ trigger download as "PNDNavigator"
```

### Browser APIs Used

- `FileReader.readAsArrayBuffer()` — read file into memory
- `DataView` — read and write individual bytes
- `Blob` + `URL.createObjectURL()` — offer patched file for download
- `<a download>` trick — trigger browser save dialog

---

## Case 2 Instructions

Steps shown on the page with numbered visual markers:

1. Télécharger le fichier navcore :  
   `https://download.tomtom.com/sweet/navcore/navcore_8843.3130258.Carminat_TomTom.cab`
2. Dézipper le fichier `.cab` (WinRAR, 7-Zip, ou l'explorateur Windows)
3. Copier **TOUS** les fichiers directement à la **racine** de la carte SD  
   ⚠️ Ne pas mettre dans le dossier `IA0000700006swlimage0`
4. Si Windows demande de remplacer des fichiers → cliquer **Ignorer**
5. Renommer `~tsystem` en `ttsystem` (enlever le `~`)
6. Retourner à l'outil Cas 1 ci-dessus pour patcher `PNDNavigator`

---

## UX Details

- Drag-and-drop zone accepts any file (user must pick the right one)
- File name displayed after selection for confirmation
- "Patcher et télécharger" button disabled until a file is selected
- Download filename: `PNDNavigator` (no extension, matching original)
- Status messages use colored banners (green/orange/red)
- No page reload needed — can patch multiple files in a row

---

## Deployment

1. Create GitHub repo (e.g. `renault-horloge`)
2. Push `index.html` to `main`
3. Settings → Pages → Source: `main` / `/ (root)`
4. Share URL: `https://<username>.github.io/renault-horloge/`

---

## Out of Scope

- Multiple languages (French only)
- Renaming `~tsystem` automatically (file rename can't be done in-browser on SD card)
- Verifying the `.cab` download or guiding extraction (OS-level steps)
- Any server-side component
