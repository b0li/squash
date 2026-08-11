# Squash – Handover

## Projekt
- **Lokaler Pfad:** `/Users/lakestudio/localhost/claude/apps/squash/`
- **Repo:** https://github.com/b0li/squash
- **Live:** https://b0li.github.io/squash/
- **Stack:** Single `index.html` – Vanilla JS, kein Build-Step
- **CDNs:** JSZip 3.10.1, heic2any 0.0.4 (beide über cdnjs/jsdelivr)
- **Herkunft:** Weiterentwicklung des WebP-Converters (`../webp-converter/`) – gleiche Basis, erweitert um freie Output-Format-Wahl.

---

## Was Squash vom WebP-Converter unterscheidet

Der WebP-Converter konvertiert **immer** nach WebP. Squash lässt das **Zielformat frei wählen**: WebP, JPEG, PNG, AVIF. Alles andere (Drag & Drop, Breiten-Resize, Qualitäts-Slider, lokal ohne Upload, Design) ist identisch.

Drei neue Mechaniken:

1. **Format-Selektor** (`<select id="formatSelect">`) in der Controls-Leiste. Jede Option trägt `data-ext` + `data-lossy`.
2. **AVIF-Feature-Detection** (`detectAvifSupport()`): Beim Init encodiert ein 2×2-Canvas testweise nach `image/avif`. Kommt kein `image/avif`-Blob zurück (Browser kann AVIF nicht encodieren), wird die AVIF-Option per `.remove()` entfernt. → Auf `https://` in Chrome sichtbar, auf `file://` bzw. in Browsern ohne AVIF-Encoder ausgeblendet. Kein Bug, gewolltes Verhalten.
3. **Qualität an Format gekoppelt** (`applyFormatUiState()`): Bei verlustfreiem Format (PNG, `data-lossy="0"`) wird der Slider `disabled` + Label/Value bekommen `.is-disabled` (Opacity 0.4). Bei lossy aktiv.

### JPEG-Sonderfall
JPEG hat keinen Alpha-Kanal. In `processImageBlob()` wird bei `mime === 'image/jpeg'` **vor** `drawImage` der Canvas weiß gefüllt (`ctx.fillStyle = '#ffffff'; ctx.fillRect(...)`), sonst würde Transparenz schwarz. WebP/PNG/AVIF behalten Alpha.

### Encoding-Call
```javascript
const encodeQuality = format.lossy ? quality : undefined; // PNG ignoriert Quality
canvas.toBlob( cb, format.mime, encodeQuality );
```
Output-Name: `originalName.replace( /\.[^.]+$/, '' ) + '.' + format.ext`. Jede Datei im `files`-Array trägt eigene `ext`/`name`, daher funktionieren gemischte Format-Batches (User wechselt Selektor zwischen Drops) und ZIP automatisch.

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

### Controls-Leiste
Drei `.control-group` (flex, gap 10px) in `.controls` (max-width 720px, flex, gap 20px, wrap):
1. **Format:** Label + `<select>`
2. **Qualität:** Label + Range-Slider (`flex:1`) + Value – `.control-group--quality` ist `flex:1`
3. **Breite:** Label + Number-Input (px, optional)

### Select / Number Input (gleiche Optik)
```css
input[type="number"], select {
    background: var( --surface-2 );
    border: 1px solid var( --border );
    border-radius: 6px;
    color: var( --text );
    font-size: 0.9rem;
    padding: 5px 8px;
}
input[type="number"]:focus, select:focus { border-color: var( --accent ); }
select option { background: var( --surface-2 ); color: var( --text ); }
```

### Range Slider
`height 4px`, `background var(--border)`, Thumb 18×18px, `border-radius 50%`, `background var(--accent)`. `:disabled` → Opacity 0.4.

### Buttons
```css
.btn { padding: 8px 18px; border-radius: 6px; font-size: 0.85rem; font-weight: 600; }
.btn--accent  { background: var( --accent ); color: #1a1d23; }
.btn--ghost   { background: var( --surface-2 ); color: var( --text-dim ); border: 1px solid var( --border ); }
.btn--danger  { background: transparent; color: var( --red ); border: 1px solid var( --red ); }
.btn--remove  { background: transparent; border: none; color: var( --text-dim ); }
.btn--remove:hover { color: var( --red ); }
.btn--download { background: var( --accent ); color: #1a1d23; font-size: 0.82rem; padding: 7px 16px; text-decoration: none; }
```

### Drop-Zone
```css
border: 2px dashed var( --border );
border-radius: var( --radius );
background: var( --surface );
/* .drag-over: */ border-color: var( --accent ); background: var( --accent-dim );
```

### File Item
Grid `52px 1fr auto`, gap 14px. Aufbau pro Zeile:
- **Thumbnail** (52×52px) mit Hover-Preview-Tooltip (`.file-item__preview`, `position:absolute`, `bottom: calc(100% + 8px)`, `max-width: min(480px, 80vw)`)
- **Info:** Original-Name (0.75rem, --text-dim) → Output-Name (0.9rem, 600) → Meta (Größen, %, Auflösung)
- **Actions:** `.file-type-badge` (dynamisch `.webp`/`.jpg`/`.png`/`.avif`) + Download-Button + ✕

### Bottom Bar
`X Dateien` · `68 KB → 23 KB · −65%` (ab 2 Dateien) · Button „Leeren" · Button „Alle als ZIP" (→ `squash.zip`).

---

## Features

- Drag & Drop + Click-to-select, mehrere Dateien gleichzeitig
- **Input-Formate:** JPG, PNG, GIF, AVIF, BMP, HEIC, HEIF, TIFF, SVG, ICO, WebP
  - HEIC/HEIF via `heic2any` → PNG → Canvas
  - Alle anderen via `FileReader.readAsArrayBuffer` → `Image` → Canvas
- **Output-Formate:** WebP, JPEG, PNG, AVIF (AVIF nur bei Encoder-Support)
- Qualitäts-Slider (1–100, Default 70) – bei PNG ausgegraut
- Breite-Input (px, optional) – nur Downscale, nie Upscale; Hinweis „Nicht vergrößert"
- Dateiliste: Thumbnail + Hover-Preview, Original-Name, Output-Name, Größenvergleich, Auflösung
- Einzel-Download + Batch-Download als ZIP (`squash.zip`)
- Bottom Bar: Dateianzahl + Gesamtersparnis (ab 2 Dateien)
- Fehlerbehandlung: Nicht-Bild-Dateien → Fehlereintrag in der Liste

---

## Zentrale JS-Funktionen

| Funktion | Aufgabe |
|---|---|
| `getSelectedFormat()` | liest `{ mime, ext, lossy }` aus dem Selektor |
| `applyFormatUiState()` | graut Slider bei lossless (PNG) aus |
| `detectAvifSupport()` | entfernt AVIF-Option bei fehlendem Encoder |
| `handleFiles()` | friert quality/width/format beim Drop ein, dispatcht pro Datei |
| `convertImage()` | HEIC-Weiche, sonst FileReader → `processImageBlob` |
| `processImageBlob()` | Resize + JPEG-Weiß-Fill + `toBlob` |
| `markDone()` | rendert Thumbnail, Meta, Badge, Download-Link |

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

GitHub Pages aktiv (main branch, root `/`, HTTPS erzwungen).

```bash
cd /Users/lakestudio/localhost/claude/apps/squash
git add index.html
git commit -m "..."
git push
# → live auf https://b0li.github.io/squash/
```
