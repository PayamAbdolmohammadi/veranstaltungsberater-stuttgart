# Veranstaltungsberater Stuttgart — Next Steps

## Roadmap Overview

```
MVP 1  ✅ COMPLETE
  Wizard (7 questions) + Checkliste + PDF-Paket + Formulare
  Barrierefreiheit + Häufige Ablehnungsgründe

MVP 2  → Budget- und Finanzierungsplan
MVP 3  → Standplanung / Standvergabe
MVP 4  → Veranstaltungsakte Lite (Laravel)
```

---

## MVP 2: Budget- und Finanzierungsplan

### Status
**Waiting.** Bärbel promised to send a sample budget (Muster). Do NOT build before receiving it.

### Why
Bärbel raised this herself (not prompted): *"Das Thema Budget — man braucht einen Kosten- und Finanzierungsplan — auch falls man Sponsoren oder Zuschüsse möchte."*

### When to start
After receiving Bärbel's Muster, analyze:
- How many line items?
- Are there fixed categories or free-form?
- Is there a specific format required by authorities?
- Does it need to support Zuschüsse/Sponsoren separately?

### Expected output
A Budget-Planer section in the app that:
- Lets user enter costs by category
- Calculates Gesamtkosten / Finanzierung / Fehlbetrag
- Exports as PDF alongside the permit documents

### Likely cost categories (from Bärbel + common sense)
- Standmiete / Platzmiete
- Bühne / Technik
- Strom
- Sanitätsdienst
- Versicherung
- Werbung / Druck
- Genehmigungsgebühren
- Sonstiges
- **Income side:** Standgebühren, Sponsoren, Zuschüsse, Eintritt

---

## MVP 3: Standplanung / Standvergabe

### Status
**Blocked.** Need to understand Bärbel's actual Excel workflow before building anything.

### Why
Bärbel said: *"Die Aufplanung und Vergabe der Stände ist sicher das Aufwendigste."*

**Important distinction:** "das Aufwendigste" ≠ "das größte Problem". May or may not be worth building.

### Required question for Bärbel
> *"Du hast erwähnt, dass die Standplanung das Aufwendigste ist. Was machst Du heute konkret in Excel? Welche Informationen trägst Du pro Stand ein?"*

### What to expect (two scenarios)

**Scenario A — Simple Excel:**
- Standnummer, Name, Telefon, Status
- → Build a simple table editor

**Scenario B — Complex Excel:**
- Standnummer, Strom (kW), Wasser (ja/nein), Gebühren, Rechnung, Zahlung, Lageplan-Position
- → Much more complex, may need significant time investment

### Expected output (if built)
- Stand allocation table/editor
- Export as Excel or PDF
- Possibly: Lageplan integration

---

## MVP 4: Veranstaltungsakte Lite

### Status
**Not started.** This is where Laravel becomes valuable.

### Trigger
Build this after MVP 3 is validated by Bärbel as complete.

### Features
- Save a Veranstaltung (name, date, location)
- Upload/attach documents (Genehmigung, Versicherung, etc.)
- Track status of each required document (ausstehend / eingereicht / genehmigt)
- Download complete Akte as ZIP

### Tech approach
- Laravel backend + MySQL
- Blade or Vue.js frontend
- File storage (local or S3)
- Simple authentication (no OAuth needed initially)

---

## Pending Actions (Non-Development)

### 1. Send to Bärbel for full test
After GitHub upload, send:
> *"Ich habe Barrierefreiheit und die häufigen Ablehnungsgründe eingebaut — genau wie du beschrieben hast. Kannst du den Wizard einmal als echtes Stadtteilfest durchspielen?"*

### 2. Ask Bärbel about Standplanung
> *"Was trägst du konkret in Excel ein — Standnummer, Strom, Wasser, Gebühren, Status?"*

### 3. Wait for Budget Muster from Bärbel
She said *"Ich schicke Dir mal ein Muster zu"* — follow up if not received within a week.

### 4. Validation question (after MVP 2+3)
> *"Fällt Dir spontan eine Veranstaltung ein, bei der dieses Tool die Vorbereitung deutlich erleichtert hätte? Und eine, bei der es heute noch nicht ausreichen würde?"*

---

## Known Issues / Technical Debt

### Resolved (June 2026)
- **Checkbox rendering & alignment** — checkboxes now use the native AcroForm API (`getCheckBox(name).check()`) instead of manually drawn `drawLine` X-marks. Fixes the misalignment where the X floated *above* the boxes. Root cause: a `+12` Y offset that had been added specifically to clear the white-fill (`/BG`) boxes so the drawn X stayed visible. Verified end-to-end by rasterizing the filled PDF. Covers Verkehrsart, all 8 Aufbauten, Flüssiggas, Anwohnerbenachrichtigung, gemnü (Antrag) and gemeinnützig (Erklärung).
- **Barrierefreiheit + Häufige Rückfragen now in the Checkliste PDF** — both sections were previously browser-only. `buildChecklistePDF()` now renders them: Barrierefreiheit with the same ja/nein/unsure logic and status tags, Häufige Rückfragen as a red warning box with all 5 items. Order matches the on-screen Beratung page.

### Open
1. **Duplicate `dl()` function** — `dl()` is defined twice in a row in index.html (harmless, last definition wins). Remove one copy.
2. **No input validation in Phase 2** — form filling has no validation. Empty fields produce blank PDFs. Add required-field indicators and basic validation before allowing PDF generation.
3. **Progress bar label hardcoded** — `setProgress()` hardcodes "von 7". Adding/removing questions needs updating in two places: the HTML dots/lines and the JS function.
4. **Single HTML file getting large** — ~1,250 lines plus ~180KB of base64 PDF data. Fine for current scope; will need splitting at the MVP 4 Laravel migration.

### PDF text rendering note
The Checkliste PDF uses NotoSans if `NotoSans-Regular.ttf` / `NotoSans-Bold.ttf` load, otherwise it falls back to Helvetica (WinAnsi). Keep new PDF text ASCII-safe (no emoji; use hyphens, not en-dash/middot) so the Helvetica fallback can never throw on an unencodable character.

---

## Recommended Next Implementation Steps

### Step 1 — GitHub upload (immediate)
Push current index.html to GitHub. MVP 1 is complete and ready for Bärbel review.

### Step 2 — Fix Ablehnungsgründe in PDF ✅ DONE (June 2026)
The 5 Häufige Rückfragen and the full Barrierefreiheit section are now rendered in the Checkliste PDF by `buildChecklistePDF()`.

### Step 3 — Wait for Bärbel's Budget Muster
Do not start Budget-Planer until Muster arrives.

### Step 4 — Build Budget-Planer (MVP 2)
Based on Muster analysis, build a budget table in Phase 1 output with PDF export.

### Step 5 — Ask Bärbel about Standplanung
Before writing a single line of code for Standplanung, get the Excel details.

### Step 6 — Build Standplanung (MVP 3)
Based on Scenario A or B from Bärbel's answer.

### Step 7 — Full validation with Bärbel
Show complete MVP 1+2+3, ask the full validation question.

### Step 8 — Migrate to Laravel (MVP 4)
With full feature scope confirmed, rebuild as a proper web app with database and authentication.

---

## Context for Next Conversation

If starting a new Claude conversation, provide:

1. This file (NEXT_STEPS.md)
2. PROJECT_SUMMARY.md
3. ARCHITECTURE.md
4. The current index.html from `/mnt/user-data/outputs/index.html`

Key things to know:
- The app is a **single HTML file** — all changes go in index.html
- PDFs are **base64 embedded** — do not try to fetch them externally
- Checkboxes use the **native AcroForm checkbox API** (`form.getCheckBox(name).check()`) — no manual coordinates
- **Bärbel** is the domain expert — her word overrides any assumption
- **Do not build Budget-Planer or Standplanung** until Bärbel's inputs are received
