# brain.md — Systemwissen & Architektur-Entscheidungen

## 🔑 KRITISCH: Tool-Rollen-Trennung

**skylight-cli (RE-ACTIVATED)**: `snapshot-compact` + `batch` für Survey-Seiten (PRIMARY). `click --element-index` ist DEPRECATED.**

**cua-driver (LEGACY)** für Popup-Interaktion (Google OAuth, Consent-Dialoge):

```bash
# Popup Window-ID finden
cua-driver call list_windows '{}'

# Popup-Elemente laden (cached pro window_id)
cua-driver call get_window_state '{"pid":PID,"window_id":WID}'

# Im Popup klicken/tippen
cua-driver call click '{"pid":PID,"window_id":WID,"element_index":N,"action":"press"}'
cua-driver call set_value '{"pid":PID,"window_id":WID,"element_index":N,"value":"text"}'
```

**cua-driver Daemon MUSS laufen**: `cua-driver serve &` vor allen Popup-Interaktionen.
Ohne Daemon: "No cached AX state" → Klick schlägt fehl.

**NEMO ist PRIMARY**: Für Survey-Seiten `skylight-cli snapshot-compact` + `src/stealth_survey/` nutzen.
cua-driver ist LEGACY Fallback für Popups/Sheets.

## AKTIVES MODELL
- **Name**: `nvidia/nemotron-3-nano-omni-30b-a3b-reasoning`
- **API**: `POST https://integrate.api.nvidia.com/v1/chat/completions`
- **Key**: `$NVIDIA_API_KEY` (Prefix: `nvapi-...`)
- **SSE**: `stream: true` → tokenweise Antwort
- **Antwort-Feld**: `msg.get("content") or msg.get("reasoning")`
- **max_tokens**: 300 → 1000 (Reasoning braucht Denk-Tokens, JSON kommt danach)

## AKTIVER CODE
- `stealth-runner/runner/live_eye.py` — LiveEye v7 (Motion Detection, Frame-Skipping, CRF Auto-Adjust)
- `stealth-runner/runner/live_omni_monitor.py` — Hybrid Screenshot + Rolling Video + SSE
- `stealth-runner/runner/live_agent.py` — TRIO: Retina (Motion) + Cortex (Omni) + Hands (skylight/cua)
- `stealth-runner/runner/stealth_executor.py` — Executor mit hold/drag/verify
- `stealth-runner/runner/persona_memory.py` — 788 Zeilen, Konsistenz-Log für Antworten
- `stealth-runner/runner/answer_logic.py` — Frage-Klassifizierung + Persona-Antworten

## ARCHIVIERT (NICHT NUTZEN!)
- `A2A-SIN-Worker-heypiggy` — Legacy Worker (brain.md sagt "ARCHIVIERT")
- `webauto-nodriver-mcp` — BANNED Stack
- `mistralai/mistral-small-latest` — veraltet, nutze Nemotron Omni

## MOTION DETECTION (LiveEye v7)
```python
MOTION_HIGH_THRESH = 20.0  # Scroll/Page-Transition → CRF 28, -1 frames
MOTION_MID_THRESH = 3.0    # Mid motion → CRF 35, 8 frames
MOTION_LOW_THRESH = <3.0   # Static frames → CRF 40, 4 frames, SKIP
```
Frame-Differencing: statische Frames (MSE < 2.0) werden übersprungen → 95% Payload-Reduktion.

## Jpeg Quality Optimization
`live_omni_monitor.py`: `quality=40` → 90% Payload-Reduktion.
Thumbnail vor API-Call: 1200×1006 → 960×805 (~67KB statt 300KB).

## GOOGLE OAUTH FLOW (cua-driver LEGACY)
1. Skylight: Klick auf Google-Login-Button (Hauptfenster)
2. cua-driver (LEGACY): Popup window_id finden
3. cua-driver (LEGACY): Email eintippen (element_index 25)
4. cua-driver (LEGACY): "Weiter" klicken (element_index 35)
5. cua-driver (LEGACY): "Fortfahren" klicken (element_index 65) — Consent
6. cua-driver (LEGACY): Finales "Weiter" (element_index 41) → Dashboard

Bei bestehenden Google-Cookies: KEIN Passwort nötig!

## PANEL ROUTING
- **Toluna**: Neuer Tab nach "Umfrage starten"
- **Dynata**: Andere Fangfragen-Muster
- **PureSpectrum**: Matrix-Fragen häufig
- **SampleClub**: Schnell, wenig Disqualifikationen

Router-Detektor: `./modules/router-detector/cli/router-detector $PID`

## PERSONA KONSISTENZ
- Antworten werden in `answer_history.jsonl` geloggt
- Wiederholte Fragen (semantisch ähnlich) → gleiche Antwort
- Profil-Daten: `profiles/jeremy.yaml` → Alter 34, Stadt München, PLZ 80331
- Fangfrage: IMMER 3-4 Marken auswählen, NIEMALS "Keine"!

## TMUX NON-BLOCKING PATTERN
Für long-running Commands NUR `interactive_bash` (tmux):
```python
interactive_bash(tmux_command="new-session -d -s mysession")
interactive_bash(tmux_command='send-keys -t mysession "command" Enter')
```
NIE `bash` mit `&` — das blockiert den shell.

## SEMGREP RULES (in stealth-runner active)
Alle Skills laufen im stealth-runner Kontext, wo 11 semgrep Regeln BANNED Muster blockieren.
Keine Recovery-Module starten wenn nicht nötig — lieber Vision fragen.
## Captcha Solving (2026-05-03)
Simple Text Captcha gelöst via NVIDIA Reasoning Extraktion + Playwright
Siehe: ~/dev/skills/opencode-captcha-simple-text-skill/SKILL.md
