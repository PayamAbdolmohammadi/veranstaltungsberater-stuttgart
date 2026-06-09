# Veranstaltungsberater Stuttgart — Architecture

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML/CSS/JavaScript (single file) |
| PDF manipulation | pdf-lib (loaded from CDN) |
| ZIP creation | JSZip (loaded from CDN) |
| Hosting | GitHub Pages |
| Fonts | Google Fonts (IBM Plex Mono, Inter) |
| Future backend | Laravel (planned for MVP 4+) |

**No build step. No npm. No framework.** Everything runs in the browser.

---

## Repository

```
https://github.com/payamabdolmohammadi/veranstaltungsberater-stuttgart
```

### Files on GitHub
```
/
├── index.html              # Entire application (~250KB with embedded PDFs)
├── antrag_source.pdf       # Original Stuttgart Antrag §29 StVO form
├── erklaerung_source.pdf   # Original Stuttgart Veranstaltererklärung form
├── NotoSans-Regular.ttf    # Font for PDF generation
├── NotoSans-Bold.ttf       # Font for PDF generation
```

### Local Development Files (NOT on GitHub)
```
/home/claude/
├── antrag_decrypted.pdf    # Decrypted version of antrag_source.pdf (pypdf)
├── erklaerung_decrypted.pdf
└── pdftest/                # Node.js test environment with pdf-lib

/mnt/user-data/outputs/
└── index.html              # Working copy — copy this to GitHub
```

---

## Application Structure (index.html)

The entire app is one HTML file (~1,238 lines). Structure:

```
<html>
  <head>
    CSS styles (lines ~1-200)
    External CDN links (pdf-lib, JSZip, Google Fonts)
  </head>
  <body>
    <header>              Phase tabs (① Checkliste ② Formulare ③ Download)
    <div#phase1>          Wizard + Results
      <div.prog-wrap>     Progress bar (7 dots + 6 lines)
      <div.sofort>        Green info box (changes per question)
      <div#q1>            Question 1: Veranstaltungsort
      <div#q1privat>      Dead-end for Privatgelände
      <div#q1unsure>      Help for unsure users
      <div#q2>            Question 2: Besucherzahl
      <div#q3>            Question 3: Erfahrung
      <div#q4>            Question 4: Gastronomie
      <div#q4b>           Question 4b: Alkohol (conditional)
      <div#q5>            Question 5: Aufbauten
      <div#q6>            Question 6: Verkehr
      <div#q7>            Question 7: Barrierefreiheit (NEW)
      <div#resultWrap>    Results page
        <div#mappe>       Summary card
        <div#clBody>      Full checklist
        <div#ablehnungBox> ⚠ Häufige Rückfragen (red warning box)
        <div#nochFehltBox> Items not auto-created
        <div#tlBox>       Timeline
        <div#ctWrap>      Contacts
        <div#pdfPaketBox> CTA to Phase 2
    <div#phase2>          Form filling (4 steps)
    <div#phase3>          ZIP download
    <script>
      PDF base64 constants (ANTRAG_PDF_B64, ERKL_PDF_B64)
      Wizard state (W object)
      SF object (sofort info box content)
      pick() function (wizard logic)
      buildChecklist() (generates all output)
      fillAntrag() / fillErklaerung() (PDF filling)
      buildPaket() (ZIP creation)
```

---

## Wizard State

```javascript
const W = {};
// W[1] = ort: 'strasse' | 'platz' | 'parkplatz' | 'privat' | 'unsure'
// W[2] = besucher: 'klein' | 'mittel' | 'gross' | 'sehr_gross' | 'unsure'
// W[3] = erfahrung: 'neu' | 'wieder'
// W[4] = gastro: 'nein' | 'ja' | 'ja_alkohol' | 'ja_kein_alkohol'
// W[5] = aufbau: 'buehne' | 'musik' | 'huepfburg' | 'sonstiges' | 'nein'
// W[6] = traffic: 'ja' | 'nein'
// W[7] = barriere: 'ja' | 'nein' | 'unsure'  ← NEW
```

Results stored in `window._paketData` for Phase 2/3:
```javascript
window._paketData = {
  ort, besucher, erstmal, food, alkohol, aufbau, traffic, barriere,
  weeks, hasBuilds, hasMusik, bigEvent, veryBig,
  ortLabel, bLabel, nochFehlt
}
```

---

## PDF Filling Architecture

### Why base64 embedded?
Original PDFs on GitHub Pages were served with wrong Content-Type, causing pdf-lib to fail silently. Solution: embed both PDFs as base64 strings directly in index.html. No network request needed.

```javascript
const ANTRAG_PDF_B64 = "JVBERi0x..."; // ~97KB base64
const ERKL_PDF_B64 = "JVBERi0x...";   // ~82KB base64
```

### PDF Filling Flow
```
User fills form (Phase 2)
  → fillAntrag(data) + fillErklaerung(data)
    → PDFDocument.load(base64, {ignoreEncryption: true})
    → page.drawText(value, ...) for text fields (positioned via annotation rects)
    → form.getCheckBox(name).check() for checkboxes (native AcroForm appearance)
    → doc.save() → Uint8Array → Blob
  → buildPaket() creates ZIP with 3 PDFs
  → JSZip.generateAsync() → download
```

### Checkbox Rendering (Important!)
The Stuttgart PDF forms use real AcroForm checkboxes. They are set via pdf-lib's native checkbox API — no coordinates involved:

```javascript
const form = doc.getForm();
form.getCheckBox('Garn').check();   // sets /AS to the on-state ('Ja')
```

This renders the field's own appearance stream (a centered X) and is reliable even for boxes carrying a white-fill appearance (`/BG [1 1 1]`), which would otherwise paint over anything drawn onto the page content stream.

**History / why this matters:** Earlier versions drew the X manually with `page.drawLine()` using hand-calibrated coordinates (an `x-2, y+12` offset from each annotation rect). The `+12` Y offset existed only to push the X *above* the white-fill boxes so it stayed visible — which is exactly why the marks floated above the boxes instead of sitting inside them. Switching to native `check()` removed all coordinates and fixed the alignment for every checkbox: Verkehrsart (Strfl/Fußg/Gehw), the 8 Aufbauten (Garn/Verk/ZelPav/Bühne/KarFahr/Hüpf/Spiel/Tier), Flüssiggas (Flüj/Flün), Anwohnerbenachrichtigung (BenBr/BenAm/BenTa), gemnü (Antrag) and gemeinnützig (Erklärung).

### AcroForm Field Names

**Antrag (antrag_source.pdf)**:
- `NameVer` — Name der Veranstaltung
- `NameInst` — Name der Institution
- `Anschrift` — Adresse
- `AnsprNam` — Ansprechpartner Name
- `gemnü` — gemeinnützig (checkbox)
- `Garn`, `Verk`, `ZelPav`, `Bühne`, `KarFahr`, `Hüpf`, `Spiel`, `Tier` — Aufbauten checkboxes
- `Flüj`, `Flün` — Flüssiggas ja/nein
- `BenBr`, `BenAm`, `BenTa` — Anwohnerbenachrichtigung
- `Strfl`, `Fußg`, `Gehw` — Verkehrsart
- `AufDat`, `UhrAuf` — Aufbau Datum/Zeit
- `VerDat`, `VerUhrAuf` — Veranstaltung Datum/Zeit
- `AbDat`, `Abuhr` — Abbau Datum/Zeit
- `gleichzanw`, `gesamtanw` — Besucherzahl

**Erklaerung (erklaerung_source.pdf)**:
- `Name der Veranstaltung`
- `BeaInst` — Institution
- `Anschrift Straße Hausnummer Postleitzahl Ort`
- `Ansprechpartnerin Zuname Vorname`
- `Datum und Unterschrift`
- `gemeinnützig` (checkbox group with Off/On states)

---

## PDF Decryption

Original PDFs from Stuttgart were encrypted (PDF 1.6). Decrypted using pypdf:

```bash
pip install pypdf --break-system-packages
python3 -c "
import pypdf
r = pypdf.PdfReader('antrag_source.pdf')
w = pypdf.PdfWriter()
for page in r.pages:
    w.add_page(page)
w.write('antrag_decrypted.pdf')
"
```

Decrypted files: `/home/claude/antrag_decrypted.pdf`, `/home/claude/erklaerung_decrypted.pdf`

---

## Sofort Info Box System

The green info box above each question uses a key-value lookup:

```javascript
const SF = {
  strasse:        [['Title', 'Description'], ...],
  platz:          [...],
  verkehr_ja:     [...],
  barriere_ja:    [...],   // NEW
  barriere_nein:  [...],   // NEW
  barriere_unsure:[...],   // NEW
  // ... etc
};
```

`showSofort(key)` renders the box. `hideSofort()` hides it.

---

## Deployment

1. Edit `/mnt/user-data/outputs/index.html`
2. Copy to GitHub repo (replace existing index.html)
3. Git commit + push
4. GitHub Pages auto-deploys in ~30 seconds

No build step. No CI/CD. Direct file replacement.

---

## Design Decisions

1. **Single HTML file** — No dependencies to manage, no build tools, works offline, easy to share and inspect
2. **Base64 embedded PDFs** — Eliminates GitHub Pages CORS/Content-Type issues with PDF loading
3. **Native AcroForm checkboxes** — `form.getCheckBox(name).check()` renders the form's own centered X reliably (incl. white-fill `/BG` boxes). Replaces an earlier `drawLine` X-mark workaround whose manual `x-2, y+12` coordinates left the marks floating above the boxes.
4. **7 questions** — Minimal viable information to generate a personalized, useful output
5. **No Laravel yet** — Building in plain HTML allows rapid iteration. Laravel migration planned after MVP 3 when full feature scope is known
6. **Barrierefreiheit as Q7** — Placed last because it's new and experimental; doesn't affect permit type, only checklist content
