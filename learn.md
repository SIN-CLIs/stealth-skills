# learn.md — KRITISCHE Learnings aus Session-Erfahrung

## 🔑 Survey Questions brauchen TEXT-Antworten!

**NIEMALS** nur klicken. Umfragen haben Textfelder (Einkommen, Alter, PLZ, Beruf).
Omni zuerst fragen: "Describe EXACTLY what you see. What question? What answer format?"
Dann passende Action: `type` (Textfeld) ODER `click` (Radio/Button).

## 🔑 Image Resize vor API-Call (thumbnail 50%)

1200×1006 → 960×805 = ~67KB JPEG statt 300KB PNG. Kein API-Timeout mehr.
`Image.thumbnail((960, 960), Image.LANCZOS)` in `_image_to_jpeg_b64()`.

## 🔑 Nemotron Omni: content > reasoning

Der Model schreibt JSON in `msg["content"]`, Reasoning-Text in `msg["reasoning"]`.
Content MUSS Priority haben.

## 🔑 max_tokens: 300→1000 für Reasoning-Models

Reasoning braucht Tokens zum Denken. JSON kommt ERST danach.
300 Tokens = JSON abgeschnitten → parse-fail → "wait".

## 🔑 Page Detection via skylight-cli

AXWebArea-Label zeigt Seitentitel: "HeyPiggy" vs "PureSpectrum" vs "Google".
In state speichern → an prompt übergeben → Model passt sich an.

## 🔑 cua-driver (LEGACY) für Popups, skylight-cli batch für Hauptfenster

Popup = `cua-driver call click "{...\"window_id\":WID,...}"` (LEGACY)
Survey-Seiten = `skylight-cli snapshot-compact --pid X --semantic` + `skylight-cli batch '[{"ref":"@e0","action":"click"}]'` (PRIMARY)
NIE mischen! skylight-cli sees KEINE Popup-Elemente.

## 🔑 Fangfragen: IMMER 3-4 Marken, NIEMALS "Keine"!

Validation-Traps zeigen dieselbe Frage mehrfach (verschiedene Formulierungen).
Antwort-Konsistenz wird geprüft. Immer die gleichen 3-4 Marken auswählen.
Persona-Profile in `stealth-runner/profiles/jeremy.yaml` nutzen.

## 🔑 Panel-Routing: Jedes Panel ist ANDERS

- **Toluna**: Öffnet neuen Tab. Auf URL-Änderung achten.
- **Dynata**: Andere DQ-Muster. Immer "Ich stimme zu" klicken.
- **PureSpectrum**: Matrix-Fragen häufig. Slider erst ganz links.
- **SampleClub**: Schnellster Panel. Wenig DQ.

Router-Detektor nutzen: `./modules/router-detector/cli/router-detector $PID`

## 🔑 Recovery: Erst Vision fragen, DANN Modul starten

Nicht blind recovery-starten. Erst Omni fragen: "What's the current state? Is recovery needed?"
Recovery-Module sind teuer (API-Calls). Nur starten wenn FAIL-Status bestätigt.

## 🔑 Motion Detection spart 95% API-Calls

LiveEye v7: Statische Frames (MSE < 2.0) werden übersprungen.
Nur bei echter Bewegung (Page-Transition, Scroll, Animation) → API-Call.
Konfiguration: `MOTION_HIGH/MID/LOW_THRESH` in `live_eye.py`.

## 🔑 Video-Recording für Post-Mortem

 IMMER `screen-follow record --video --output /tmp/session.mp4` laufen lassen.
Bei Fehlern: `python3 -m runner.video_analyzer --last errors`
Zeigt exakt wo was schiefging.

## §Q — LIVE CRASH-TEST DISCOVERIES (2026-05-07)

> **Status**: 10+ discoveries during live debugging marathon on heypiggy.com
> **Source**: stealth-runner/survey-cli CDP automation

### §Q1 — Surveys open in NEW tabs
Surveys navigate to external URLs (Qualtrics, Samplicio) in NEW Chrome tabs.
CDP connection stays on dashboard tab. MUST detect new tab and reconnect.
```python
# After clickSurvey(), check for new tabs
tabs_after = json.loads(urllib.request.urlopen('http://127.0.0.1:9999/json'))
if len(tabs_after) > len(tabs_before):
    new_tab = [t for t in tabs_after if t['id'] not in seen_ids][0]
    ws_url = f"ws://127.0.0.1:9999/devtools/page/{new_tab['id']}"
```

### §Q2 — Multiple stacked modals on heypiggy
Dashboard has 7-9 layered modals at identical z-index and coordinates.
Clicking "Nächste" at (600,547) hits "Schließen" button instead.
**Fix**: Close all stacked modals via JS before interacting.

### §Q3 — React form filling requires native value setter
`.value = 'X'` does NOT trigger React onChange. Must use:
```javascript
var nativeSetter = Object.getOwnPropertyDescriptor(HTMLInputElement.prototype, 'value');
nativeSetter.set.call(el, '10785');
el.dispatchEvent(new Event('input', {bubbles: true}));
el.dispatchEvent(new Event('change', {bubbles: true}));
```
Alternative: `document.execCommand('insertText', false, val)` for text areas.

### §Q4 — Qualtrics language selector is <select> dropdown
The language picker is `<select class="Q_lang">` with `<option>Deutsch</option>`.
NOT a set of clickable labels. Must set `selectedIndex` + dispatch change event.

### §Q5 — Balance read bug (125€ vs 2.23€)
`read_balance()` used `Math.max()` of all € values on page.
Level progress "125" appeared near € symbol. Fixed by filtering `val > 1.0 && val < 1000`.

### §Q6 — Fill-by-element-ID is most reliable
Angular Material generates dynamic IDs: `Age`, `Zip`, `mat-radio-X-input`, `next_0`.
`document.getElementById('Age')` far more reliable than querySelector approaches.

### §Q7 — CDP Input.dispatchMouseEvent for real clicks
`element.click()` via Runtime.evaluate fails on layered React modals.
Use CDP's Input domain for real mouse events at exact coordinates.

### §Q8 — cua-driver needs --force-renderer-accessibility
cua-driver returns 0 AX elements when Chrome lacks the accessibility flag.
Chrome started without this flag is invisible to macOS accessibility tools.

### See also
- **fix.md**: Fixes #5-#9 (balance, React forms, stacked modals, modal snapshot, tab detection)
- **issues.md**: 11 open issues (3 P0, 3 P1, 3 P2, 2 P3)
- **stealth-runner**: Primary implementation repo at SIN-CLIs/stealth-runner
