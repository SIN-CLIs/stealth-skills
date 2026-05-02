# anti-learn.md — ANTI-PATTERNS (NIEMALS MACHEN!)

## ❌ Blind klicken ohne Vision-Prompt

**FALSCH**: Einfach `skylight-cli click --pid X --element-index Y` aufrufen.
**RICHTIG**: Erst Omni fragen: "Was sehe ich? Welche Action?" → JSON → execute.

## ❌ skylight-cli in Popups

**FALSCH**: `skylight-cli list-elements --pid X` im Google OAuth Popup.
**PROBLEM**: skylight-cli cached NUR das Hauptfenster. Popup-Indices sind INVALID.
**RICHTIG**: `cua-driver call get_window_state '{"pid":X,"window_id":W}'` für Popups.

## ❌ Koordinaten raten (--x --y)

**FALSCH**: `skylight-cli click --pid X --x 500 --y 300`
**PROBLEM**: Chrome-Fenster verschiebt sich, Layout ändert sich.
**RICHTIG**: Immer `skylight-cli click --pid X --element-index Y` (AX-Accessibility).

## ❌ Recovery-Module blind starten

**FALSCH**: `./modules/recovery-overquota/cli/recovery-overquota $PID` sofort.
**PROBLEM**: Verschwendet API-Calls, kann Survey killen wenn nicht nötig.
**RICHTIG**: Erst `python3 runner/step.py` → FAIL → Recovery.

## ❌ Ohne Video-Recording arbeiten

**FALSCH**: Survey ohne screen-follow laufen lassen.
**PROBLEM**: Kein Post-Mortem möglich. Gleiche Fehler wiederholen sich.
**RICHTIG**: `screen-follow record --video --output /tmp/session.mp4 &`

## ❌ Max_tokens zu klein (300 statt 1000)

**FALSCH**: `max_tokens: 300` für reasoning models.
**PROBLEM**: JSON abgeschnitten → parse-fail → "wait" → Survey stuck.
**RICHTIG**: `max_tokens: 1000` minimum.

## ❌ Image ohne Resize an API senden

**FALSCH**: 1200×1006 PNG direkt an API.
**PROBLEM**: 300KB+ payload → Timeout → Vision-Fail.
**RICHTIG**: `Image.thumbnail((960, 960), Image.LANCZOS)` → JPEG quality 40.

## ❌ Recovery Mode aktivieren (semgrep blockiert)

`recovery_mode: true` ist BANNED. Nutze Recovery-Module in stealth-skills stattdessen.

## ❌ webauto-nodriver-mcp nutzen

BANNED Stack. Nutze skylight-cli + cua-driver + playstealth.

## ❌ User-Chrome manipulieren

`open -na "Google Chrome"` manipuliert den Browser des Nutzers.
Nur `playstealth launch` (isolierte Chrome-Instanz).

## ❌ Nur klicken, keine Textfelder beachten

Umfragen haben Textfelder (Beruf, Einkommen, PLZ).
Nur klicken = Survey disqualified. Immer erst fragen: click oder type?