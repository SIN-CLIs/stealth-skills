# AGENTS.md — Stealth-Skills Agenten-Anleitung

> **Private Skill Library** — Plattformspezifische Automatisierung für die Stealth-Quad.
> Enthält Betriebsgeheimnisse. NICHT öffentlich.

---

## 🎯 WAS IST stealth-skills?

```
platforms/heypiggy/
├── google-login/      → Google OAuth Login (cua-driver Popup)
├── heypiggy-survey/   → Dashboard → Survey → EUR
└── modules/           → 8 spezialisierte Sub-Skills
    ├── router-detector
    ├── question-slider
    ├── question-ranking
    ├── question-matrix
    ├── question-opentext
    ├── recovery-attentioncheck
    ├── recovery-overquota
    └── heypiggy-history
```

---

## 🔑 AKTIVES MODELL (NVIDIA NIM)

- **Model**: `nvidia/nemotron-3-nano-omni-30b-a3b-reasoning`
- **API**: `POST https://integrate.api.nvidia.com/v1/chat/completions`
- **Key**: `$NVIDIA_API_KEY` (Prefix: `nvapi-...`)
- **SSE**: `stream: true` → tokenweise Antwort
- **Antwort**: `msg.get("content") or msg.get("reasoning")`

---

## 🛠️ TOOL-CHAIN (STRIKT BEACHTEN!)

### Tool-Rollen (KRITISCH!)

| Tool | Wann | Command |
|------|------|---------|
| `playstealth` | Chrome starten | `playstealth launch --url 'https://heypiggy.com/?page=dashboard'` |
| `skylight-cli` | **Hauptfenster** interagieren | `skylight-cli click --pid X --element-index Y` |
| `cua-driver` | **Popups** interagieren | `cua-driver call click '{"pid":X,"window_id":W,"element_index":N}'` |
| `screen-follow` | Video aufnehmen | `screen-follow record --video --output /tmp/session.mp4` |

### NIE DIESE TOOLS (BANNED)

- ❌ `pyautogui` — Mausbewegung verboten
- ❌ `pynput` — Tastatur-Emulation verboten
- ❌ `open -na "Google Chrome"` — Nutzer-Chrome manipulieren
- ❌ `playstealth launch (isolierte PID)` — falsches Pattern
- ❌ `skylight-cli click --x --y` — Koordinaten raten verboten
- ❌ `webauto-nodriver-mcp` — BANNED Stack
- ❌ skylight-cli in **Popups** — falsches Fenster!

---

## 📋 HEYPIGGY SURVEY WORKFLOW

### Schritt 1: Chrome starten
```bash
playstealth launch --url 'https://heypiggy.com/?page=dashboard'
# → {"pid": 12345, "status": "ok"}
```

### Schritt 2: Google Login (cua-driver für Popup!)
```bash
PID=<CHROME_PID>
cua-driver serve &

# Google Login Button (Hauptfenster via skylight)
skylight-cli click --pid $PID --element-index 33

# Popup: Email eintippen
WID=$(cua-driver call list_windows '{}' | python3 -c "...")
cua-driver call set_value '{"pid":$PID,"window_id":$WID,"element_index":25,"value":"email@gmail.com"}'

# Popup: Weiter klicken
cua-driver call click '{"pid":$PID,"window_id":$WID,"element_index":35,"action":"press"}'

# Popup: Consent "Fortfahren" klicken
cua-driver call click '{"pid":$PID,"window_id":$WID,"element_index":65,"action":"press"}'

# Popup: Finales "Weiter" → LOGGED IN
cua-driver call click '{"pid":$PID,"window_id":$WID,"element_index":41,"action":"press"}'
```

### Schritt 3: Dashboard scannen
```bash
skylight-cli list-elements --pid $PID | python3 -c "
import json,sys
for e in json.load(sys.stdin)['elements']:
    l=e.get('label','')
    if 'Umfrage' in l or '€' in l:
        print(f'[{e[\"index\"]}] {e[\"role\"]}: {l[:80]}')
"
```

### Schritt 4: Survey starten + Routing
```bash
# Survey klicken
skylight-cli click --pid $PID --element-index <N>

# Router-Detektor (modular)
./platforms/heypiggy/modules/router-detector/cli/router-detector $PID
# → Erkennt Toluna/Dynata/SampleClub/PureSpectrum
```

### Schritt 5: Frage-Handling (Module)

| Fragetyp | Modul | CLI |
|----------|-------|-----|
| Slider | question-slider | `./modules/question-slider/cli/question-slider-set $PID <index> <value>` |
| Ranking | question-ranking | `./modules/question-ranking/cli/question-ranking-solve $PID` |
| Matrix | question-matrix | `./modules/question-matrix/cli/question-matrix-solve $PID` |
| Open Text | question-opentext | `./modules/question-opentext/cli/question-opentext-answer $PID` |

### Schritt 6: Recovery (wenn was schief geht)

| Problem | Modul |
|---------|-------|
| Overquota | `./modules/recovery-overquota/cli/recovery-overquota $PID` |
| Attention Check | `./modules/recovery-attentioncheck/cli/recovery-attentioncheck $PID` |

---

## 🧠 SKILL STRUKTUR PRO MODUL

Jedes Modul hat:
```
<module>/SKILL.md       → Was macht dieses Modul? Wann nutzen?
<module>/cli/<binary>  → Ausführbare CLI
<module>/recovery.md   → Was tun wenn es fehlschlägt?
```

Kein SKILL.md = nicht nutzen. Immer zuerst SKILL.md lesen.

---

## 🔄 PANEL-SPECIFIC ROUTING

| Panel | Erkennung | Specific |
|-------|-----------|----------|
| Toluna | URL contains toluna | Survey startet in neuem Tab |
| Dynata | URL contains dynata | Andere Fangfragen-Muster |
| PureSpectrum | URL contains purespectrum | Matrix-Fragen häufig |
| SampleClub | URL contains sampleclub | Schnell, wenig DQ |

Routing-Detektor nutzen: `./modules/router-detector/cli/router-detector $PID`

---

## ⚠️ GOLDEN RULES

1. **NIE** ohne Vision-Prompt klicken
2. **NIEMALS** Koordinaten raten — immer `--element-index`
3. **IMMER** cua-driver für Popups (Google OAuth, Consent)
4. **IMMER** skylight-cli für Hauptfenster
5. **NIE** Recovery-Module ohne FAIL-Status starten
6. **IMMER** Video aufnehmen (screen-follow) für Post-Mortem

---

## 📚 VERWANDTE DOKUMENTATION

- `brain.md` — Systemwissen & Architektur-Entscheidungen
- `commands.md` — Alle CLI-Befehle im Detail
- `platforms/heypiggy/google-login/SKILL.md` — Google OAuth Flow
- `platforms/heypiggy/heypiggy-survey/SKILL.md` — Survey-Ablauf
- `platforms/heypiggy/modules/*/SKILL.md` — Modul-spezifisch