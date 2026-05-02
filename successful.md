# successful.md — Was funktioniert (2026-05-02)

## ✅ Google OAuth Login (cua-driver Popup)

Komplett verifiziert mit PID 31710. Flow:
1. Skylight: Google-Login-Button klicken (--element-index 33)
2. cua-driver: Popup window_id finden (list_windows)
3. cua-driver: Email eintippen (element_index 25, set_value)
4. cua-driver: "Weiter" (element_index 35)
5. cua-driver: "Fortfahren" Consent (element_index 65)
6. cua-driver: Finales "Weiter" (element_index 41) → Dashboard

**0 Passwort nötig** dank bestehender Google-Cookies.

## ✅ LiveEye v7 Motion Detection

95% Payload-Reduktion durch Frame-Differencing. Statische Frames übersprungen.
CRF Auto-Adjust: 28 (high motion) / 35 (mid) / 40 (static).
Erkennt Page-Transitions, Scroll-Events, Captchas.

## ✅ Nemotron Omni SSE Streaming

Tokenweise Antwort in <1s. Erster Token früher als bei batch.
SSE via httpx streaming. JSON-enforced Prompt → strukturierte Antwort.

## ✅ Persona Memory (788 Zeilen)

Antwort-Konsistenz über alle Screens. answer_history.jsonl loggt alles.
Fangfragen: IMMER 3-4 Marken. Niemals "Keine".

## ✅ Stealth-Quad Tool-Chain

- playstealth → isolierte Chrome-Instanz
- skylight-cli → AX-Accessibility (keine Maus)
- cua-driver → Popup-Interaktion (window_id targetiert)
- screen-follow → Video-Aufzeichnung

Keine Koordinaten. Keine Maus. Kein User-Chrome.

## ✅ Router-Detektor

Erkennt Toluna/Dynata/PureSpectrum/SampleClub.
Passt DQ-Strategie an Panel-spezifische Muster an.

## ✅ Recovery Module

- recovery-overquota: Erkennt "overquota" Text → Alternative suchen
- recovery-attentioncheck: Fangfrage-Detektion → Richtige Antwort geben

## ✅ Graphify Knowledge Graph

4.820 Nodes, 10.860 Edges über 6 Repos.
Graphify-Hooks (post-commit, post-checkout) halten AST aktuell.

## ✅ Semgrep Architecture Guard

11 Regeln blockieren BANNED Muster VOR Commit.
Verhindert: pyautogui, pynput, coordinates-click, recovery_mode:true, webauto-nodriver.

## ✅ State Machine + Survey Queue

State Machine orchestriert: Dashboard → Survey → DQ → Complete.
Queue-basiertes Management mit Retry-Logik.