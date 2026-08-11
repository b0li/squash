# WebP Converter – Handover

## Projekt
- **Lokaler Pfad:** `/Users/lakestudio/localhost/claude/apps/webp-converter/`
- **Repo:** https://github.com/b0li/webp-converter
- **Live:** https://b0li.github.io/webp-converter/
- **Stack:** Single `index.html` – Vanilla JS, kein Build-Step
- **CDNs:** JSZip 3.10.1, heic2any 0.0.4 (beide über cdnjs/jsdelivr)

---

## Design System

### CSS Custom Properties
```css
:root {
    --bg: #1a1d23;
    --surface: #22262e;
    --surface-2: #2a2f39;
    --border: #333844;
    --accent: #d4a55a;
    --accent-dim: rgba( 212, 165, 90, 0.15 );
    --red: #e06c6c;
    --text: #e8eaf0;
    --text-dim: #8a909e;
    --radius: 10px;
}
```

> Kein `--green` – alles läuft über `--accent` (Gold).

### Fonts
```css
@import url('https://fonts.googleapis.com/css2?family=DM+Serif+Display&family=DM+Sans:wght@400;600;700&display=swap');
```
- **h1:** `DM Serif Display`, 2rem, weight 400, color `var(--accent)`
- **Body:** `DM Sans`, system-ui Fallback

### Typografie-Skala
| Element | Font-Size | Weight | Color |
|---|---|---|---|
| h1 | 2rem | 400 (Serif) | --accent |
| Subtitle | 0.9rem | 400 | --text-dim |
| Label | 0.85rem | 400 | --text-dim |
| Body / Name | 0.9rem | 600 | --text |
| Original-Name | 0.75rem | 400 | --text-dim |
| Meta / Badge | 0.75rem | 400/600 | --text-dim |

### Body
```css
body {
    background: var( --bg );
    color: var( --text );
    font-family: 'DM Sans', system-ui, sans-serif;
    min-height: 100vh;
    padding: 32px 16px 64px;
}
```

---

## UI-Komponenten

### Surface-Karte
```css
background: var( --surface );
border-radius: var( --radius );
border: 1px solid var( --border );
padding: 14px 16px;
```

### Buttons
```css
.btn { padding: 8px 18px; border-radius: 6px; font-size: 0.85rem; font-weight: 600; }
.btn:not(:disabled):hover { opacity: 0.85; }
.btn:disabled { opacity: 0.4; cursor: default; }

.btn--accent  { background: var( --accent ); color: #1a1d23; }
.btn--ghost   { background: var( --surface-2 ); color: var( --text-dim ); border: 1px solid var( --border ); }
.btn--danger  { background: transparent; color: var( --red ); border: 1px solid var( --red ); }
.btn--remove  { background: transparent; border: none; color: var( --text-dim ); font-size: 1rem; }
.btn--remove:hover { color: var( --red ); }

/* Download-Link als Button */
.btn--download {
    background: var( --accent );
    color: #1a1d23;
    font-size: 0.82rem;
    padding: 7px 16px;
    border-radius: 6px;
    font-weight: 600;
    text-decoration: none;
}
```

### Drop-Zone
```css
border: 2px dashed var( --border );
border-radius: var( --radius );
background: var( --surface );
/* Drag-over: */
border-color: var( --accent );
background: var( --accent-dim );
```

### Controls-Leiste (Slider + Number Input)
```css
/* Wrapper */
max-width: 720px; display: flex; align-items: center; gap: 20px;
background: var( --surface ); border-radius: var( --radius ); padding: 20px 24px;

/* Range Slider */
input[type="range"] { -webkit-appearance: none; height: 4px; background: var( --border ); border-radius: 2px; }
/* Thumb: 18×18px, border-radius 50%, background var(--accent) */

/* Number Input */
input[type="number"] {
    width: 80px; background: var( --surface-2 );
    border: 1px solid var( --border ); border-radius: 6px;
    color: var( --text ); font-size: 0.9rem; padding: 5px 8px;
}
input[type="number"]:focus { border-color: var( --accent ); }
```

### File Item (Dateiliste)
Grid: `52px 1fr auto` · gap 14px

```css
.file-item {
    background: var( --surface );
    border-radius: var( --radius );
    padding: 12px 16px;
    display: grid;
    grid-template-columns: 52px 1fr auto;
    align-items: center;
    gap: 14px;
    border: 1px solid var( --border );
}
```

Aufbau pro Zeile:
- **Thumbnail** (52×52px, border-radius 6px) mit Hover-Preview-Tooltip (siehe unten)
- **Info:** Original-Dateiname (0.75rem, --text-dim) → fetter Output-Name → Meta-Zeile (Größen, %, Auflösung)
- **Actions:** `.file-type-badge` (`.webp`) + Download-Button + ✕-Button

### Thumbnail Hover-Preview
```css
.file-item__thumb { position: relative; overflow: visible; }

.file-item__preview {
    position: absolute;
    z-index: 30;
    left: 0;
    bottom: calc( 100% + 8px );
    display: none;
    padding: 4px;
    background: var( --surface-2 );
    border: 1px solid var( --border );
    border-radius: 8px;
    box-shadow: 0 12px 32px rgba( 0, 0, 0, 0.5 );
    pointer-events: none;
}
.file-item__thumb:hover .file-item__preview { display: block; }
.file-item__preview img { max-width: min( 480px, 80vw ); max-height: 70vh; border-radius: 4px; }
```

### Bottom Bar (Anzahl + Ersparnis + Buttons)
```css
.bottom-bar {
    max-width: 720px;
    margin: 16px auto 0;
    display: none; /* → display: flex wenn Dateien vorhanden */
    align-items: center;
    gap: 10px;
}
/* .bottom-bar__count: flex: 1, 0.85rem, --text-dim */
/* .bottom-bar__saving: 0.85rem, --accent, font-weight 600 */
```

Inhalt: `X Dateien` · `68 KB → 23 KB · −65%` · Button „Leeren" · Button „Alle als ZIP"

### Status-Farben
```css
.status--processing { color: var( --accent ); }
.status--done       { color: var( --accent ); }
.status--error      { color: var( --red ); }
```

### Progress Bar
```css
height: 3px; background: var( --border ); border-radius: 2px;
/* Fill: background: var( --accent ) */
```

---

## Features

- Drag & Drop + Click-to-select, mehrere Dateien gleichzeitig
- Unterstützte Formate: JPG, PNG, GIF, AVIF, BMP, HEIC, HEIF, TIFF, SVG, ICO, WebP
  - HEIC/HEIF via `heic2any` → PNG → Canvas
  - Alle anderen via `FileReader.readAsArrayBuffer` → `Image` → Canvas
- Qualitäts-Slider (1–100, Default: 70)
- Breite-Input (px, optional) – skaliert nur herunter, nie hoch; Hinweis „Nicht vergrößert"
- `canvas.toBlob('image/webp', quality)` – alles lokal, kein Upload
- Dateiliste: Thumbnail + Hover-Preview, Original-Name, WebP-Name, Größenvergleich, Auflösung
- Einzel-Download als `.webp` + Batch-Download als ZIP
- Bottom Bar: Dateianzahl + Gesamtersparnis (ab 2 Dateien)
- Fehlerbehandlung: Nicht-Bild-Dateien → Fehlereintrag in der Liste

---

## Code-Style (LS-Standard)

- **Tabs** für Einrückung
- **Spaces inside Parentheses:** `document.querySelector( '.foo' )`
- **Space nach `!`:** `if ( ! el ) return;`
- **Closing Callbacks:** `});`
- Leerzeilen zwischen logischen Blöcken
- Kommentare auf Englisch
- Vanilla JS, kein Framework

---

## Deployment

```bash
# GitHub Pages aktiv (main branch, root /)
git add index.html
git commit -m "..."
git push
# → live auf https://b0li.github.io/webp-converter/
```
