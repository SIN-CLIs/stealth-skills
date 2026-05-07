# fix.md — Bekannte Bugs & Fixes

## 🔧 Fix: "No cached AX state" (cua-driver LEGACY)

**Problem**: cua-driver Klick fehlschlägt mit "No cached AX state".
**Ursache**: `cua-driver serve` läuft nicht oder Popup noch nicht geladen.
**Fix**:
```bash
# 1. Daemon starten
cua-driver serve &

# 2. Popup laden (Wartezeit)
sleep 2

# 3. State checken VOR Klick
cua-driver call get_window_state '{"pid":PID,"window_id":WID}'
# Falls leer → noch warten
```

## 🔧 Fix: "JSON abgeschnitten" → parse-fail → "wait"

**Problem**: Omni gibt nur "wait" zurück, kein JSON.
**Ursache**: `max_tokens: 300` — JSON kommt ab Token 300+.
**Fix** in `runner/nemotron_omni.py`:
```python
"max_tokens": 1000  # war 300
```

## 🔧 Fix: API-Timeout bei großen Images

**Problem**: Request timeout, Vision liefert nichts.
**Ursache**: 1200×1006 PNG = 300KB+ payload.
**Fix**:
```python
img = Image.open(path)
img.thumbnail((960, 960), Image.LANCZOS)
# → ~67KB JPEG
```

## 🔧 Fix: skylight-cli Index Invalid nach Page-Transition

**Problem**: element-index 42 funktionierte vorher, jetzt nicht mehr.
**Ursache**: Neue Seite = neues DOM = neue Indices.
**Fix**:
```bash
# IMMER neu scannen nach Page-Transition
skylight-cli list-elements --pid $PID | python3 -c ...
```

## 🔧 Fix: Screen-Follow startet nicht

**Problem**: `screen-follow record --video` startet nicht.
**Ursache**: Screen Recording Permission fehlt.
**Fix**:
```bash
# macOS System Settings → Privacy & Security → Screen Recording
# → screen-follow.app erlauben
```

## 🔧 Fix: NVIDIA API Key ungültig

**Problem**: 401 Unauthorized von NVIDIA NIM.
**Ursache**: Key abgelaufen oder falsches Prefix.
**Fix**:
```bash
# Key prüfen
echo $NVIDIA_API_KEY | head -c 10
# Sollte mit nvapi- beginnen

# Key setzen falls fehlt
export NVIDIA_API_KEY=nvapi-...
```

## 🔧 Fix: Recovery Module startet nicht (Permission)

**Problem**: `Permission denied` beim CLI-Modul-Aufruf.
**Ursache**: CLI nicht ausführbar.
**Fix**:
```bash
chmod +x platforms/heypiggy/modules/*/cli/*
```

## 🔧 Fix: Graphify outdated nach Code-Änderungen

**Problem**: Knowledge Graph zeigt alten Code.
**Ursache**: Hooks liefen nicht (detached HEAD).
**Fix**:
```bash
graphify update ~/dev/stealth-skills
# oder
graphify update ~/dev/stealth-runner
```

## 2026-05-07 Live Debugging Fixes

### Fix #5: Balance reads 125€ instead of 2.23€
- **ROOT CAUSE**: `read_balance()` took Math.max of all numbers near €. Level progress "125" appeared near € sign.
- **FIX**: Changed to `if (val > 1.0 && val < 1000)` and check adjacent lines for "Level"/"Min" keywords
- **FILE**: survey-cli/survey/scanner.py :: read_balance()
- **VERIFIED**: Balance now reads 2.23€ consistently

### Fix #6: React form inputs not accepting .value
- **ROOT CAUSE**: React synthetic events ignore direct .value= setter
- **FIX**: Use `Object.getOwnPropertyDescriptor(HTMLInputElement.prototype, 'value').set.call(el, val)` + dispatchEvent('input') + dispatchEvent('change')
- **FILE**: survey-cli/survey/snapshot.py
- **VERIFIED**: Zip=10785, Age=53 accepted, button enables

### Fix #7: Multiple stacked modals blocking clicks
- **ROOT CAUSE**: 7-9 layered modals at identical coordinates
- **FIX**: Close all "Schließen" buttons via JS before interacting with survey
- **VERIFIED**: Survey questions visible after closing modals

### Fix #8: Modal-only element scanning
- **ROOT CAUSE**: ELEMENT_EXTRACTOR_JS scanned entire document (84+ elements)
- **FIX**: Topmost modal detection by viewport center distance
- **VERIFIED**: Element count reduced to 3-5 for modal surveys

### Fix #9: New tab detection for Qualtrics
- **ROOT CAUSE**: Survey navigates to external URL in new tab
- **FIX**: Check tab count via /json before/after clickSurvey()
- **VERIFIED**: Survey questions visible after connecting to correct tab
