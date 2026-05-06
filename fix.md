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