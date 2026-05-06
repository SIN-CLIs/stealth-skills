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