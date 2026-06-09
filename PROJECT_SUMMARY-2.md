# Veranstaltungsberater Stuttgart — Project Summary

## Vision & Business Goal

A digital advisor that guides event organizers in Stuttgart through the entire permit process for public events under §29 StVO. The tool replaces hours of research and back-and-forth with city authorities by generating a personalized checklist, timeline, contacts, and automatically filled official PDF forms — all in one download.

**Target user:** Anyone organizing a public event in Stuttgart (Stadtteilfest, Weihnachtsmarkt, Flohmarkt, etc.)

**Core value proposition:** From "I don't know where to start" to "I have everything I need to submit" in under 5 minutes.

---

## Domain Expert

**Bärbel** — 30 years of experience organizing and administering public events in Stuttgart. She has reviewed every version of the product and provided direct feedback. Her input is the primary source of domain knowledge.

Key validated statements from Bärbel:
- *"Mir fällt nichts ein, was noch fehlt"* — MVP 1 validated
- *"Für die Beantragung brauchst Du Excel nicht"* — confirms Antrag/Checkliste scope is complete
- *"Das Aufwendigste"* — about Standplanung (Stand allocation)
- *"oft ein Grund für eine Genehmigung oder eben keine Genehmigung"* — about Barrierefreiheit
- Promised to send a Budget Muster (sample budget template) — not yet received

---

## Live URL

```
https://payamabdolmohammadi.github.io/veranstaltungsberater-stuttgart/
```

---

## Current Status: MVP 1 Complete

All core features are built, validated by domain expert, and deployed.

---

## Completed Features

### Phase 1 — Wizard (7 Questions)
1. **Veranstaltungsort** — Straße / Platz / Parkplatz / Privatgelände
2. **Besucherzahl** — Klein / Mittel / Groß / Sehr groß
3. **Erfahrung** — Erstveranstaltung / Wiederkehrend
4. **Gastronomie** — Nein / Ja / Ja mit Alkohol
5. **Aufbauten & Programm** — Bühne / Musik / Hüpfburg / Sonstiges
6. **Verkehr** — Straßensperrung ja/nein
7. **Barrierefreiheit** — Ja geplant / Noch nicht geklärt / Ich weiß nicht welche Anforderungen gelten *(NEU — from Bärbel feedback)*

Each question shows a green "sofort" info box explaining what the answer means legally/practically.

### Phase 2 — Personalized Output
- **Mappe** (summary card): Fläche, Besucherzahl, Antragsfrist, Gastronomie
- **Checkliste** with sections:
  - Pflichtunterlagen (always)
  - Gastronomie (conditional)
  - Fliegende Bauten (conditional)
  - Musik & Lärm (conditional)
  - Verkehrsmaßnahmen (conditional)
  - Barrierefreiheit (always, content varies by answer) *(NEU)*
- **⚠ Häufige Rückfragen der Behörden** — red warning box *(NEU — from Bärbel feedback)*:
  - Fluchtwege nicht freigehalten
  - Feuergassen fehlen oder zu schmal
  - Hygiene-Auflagen nicht erfüllt
  - Lautstärke / Nachtruhe missachtet
  - Barrierefreiheit nicht berücksichtigt
- **Noch nicht automatisch erstellt** — items user must gather manually
- **Zeitplan** — timeline with weeks before event
- **Kontakte** — relevant city offices with phone/email/hours

### Phase 3 — Formulare (4-step form filling)
User enters their data once, tool fills both official PDF forms.

### Phase 4 — ZIP Download
Contains 3 PDFs:
1. `Checkliste_[EventName].pdf` — personalized checklist (mirrors the on-screen Beratung output, incl. Barrierefreiheit + Häufige Rückfragen)
2. `Antrag_Verkehrsflaeche.pdf` — filled §29 StVO Antrag (official Stuttgart form)
3. `Veranstaltererklarung.pdf` — filled Veranstaltererklärung (official Stuttgart form)

---

## Important Domain Knowledge

### Legal Requirements
- **§29 StVO** — the main law governing public events on public roads/spaces in Stuttgart
- **LGastG BW** — Landesgaststättengesetz Baden-Württemberg (replaced old §12 GastG) — each food stand needs separate Anzeige nach §2 LGastG
- **MVAS** — Merkblatt über Rahmenbedingungen für erforderliche Fachkenntnisse zur Verkehrsabsicherung — NEW 2026, required for traffic safety responsible person
- **Reisegewerbekarte** — required for commercial vendors at public events
- **Versammlungsstättenverordnung BW** — applies at 1000+ persons (written safety concept required)

### Key City Contacts
- **Amt für öffentliche Ordnung — Bürgerservice Veranstaltungen**: Tel. 0711 216-91131, veranstaltungsmanagement@stuttgart.de, Eberhardstr. 35, Mo/Mi/Do/Fr 9–12, Do 14–15:30
- **Gewerbe- und Gaststättenrecht**: Tel. 0711 216-98905, gaststaettenrecht@stuttgart.de, Schloßstr. 73, Mo/Mi/Fr 8:30–12:30, Do 13–18
- **Baurechtsamt**: Tel. 0711 216-60100, BSBauen@stuttgart.de, Eberhardstr. 33
- **Tiefbauamt**: responsible for traffic measures / MVAS

### Timelines
- First-time event: ~12 weeks before
- Repeat event: ~8 weeks before
- Insurance: at least 2 weeks before
- Summer peak season can require longer

---

## What Is Working

- Complete 7-step Wizard with conditional logic
- Dynamic Checkliste based on all 7 answers
- Barrierefreiheit section with 3 distinct response paths — on-screen AND in the Checkliste PDF
- Häufige Rückfragen warning box (5 items from Bärbel) — on-screen AND in the Checkliste PDF
- Both official Stuttgart PDF forms filled via pdf-lib (browser-side)
- Checkboxes set via the native AcroForm checkbox API (`form.getCheckBox(name).check()`) — renders the form's own centered X, works even on boxes with a white-fill (`/BG`) appearance
- ZIP download with all 3 PDFs
- Both PDFs embedded as base64 in index.html (no external fetch needed)

---

## What Is Not Implemented Yet

- **Budget- und Finanzierungsplan** (MVP 2) — waiting for Bärbel's Muster
- **Standplanung / Standvergabe** (MVP 3) — need to understand Bärbel's Excel workflow first
- **Veranstaltungsakte Lite** (MVP 4) — save/upload documents, track status — this is where Laravel makes sense
