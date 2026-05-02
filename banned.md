# banned.md — Verbotene Tools & Patterns (STRENG!)

## 🚫 BANNED TOOLS

| Tool | Warum | Alternative |
|------|-------|-------------|
| `pyautogui` | Mausbewegung = Detection | `skylight-cli click --element-index` |
| `pynput` | Tastatur-Emulation = Detection | `skylight-cli type --element-index` |
| `open -na "Google Chrome"` | Manipuliert User-Chrome | `playstealth launch` (isolierte Instanz) |
| `playstealth launch (isolierte PID)` | Falsches Pattern | `playstealth launch --url` |
| `webauto-nodriver-mcp` | Veralteter Stack | skylight-cli + cua-driver |
| `openai.OpenAI()` | Wrong API | `httpx` an NVIDIA NIM |
| `recovery_mode: true` | Semgrep blockiert | Recovery-Module in stealth-skills |

## 🚫 BANNED PATTERNS

```bash
# ❌ KOORDINATEN RATEN
skylight-cli click --x 500 --y 300
skylight-cli click --pid X --x Y --y Z

# ❌ skylight-cli IN POPUPS
skylight-cli list-elements --pid X  # im OAuth Popup!
skylight-cli click --pid X --element-index Y  # falsches Fenster!

# ❌ Recovery Mode
recovery_mode: true
omni_fallback: llama

# ❌ Blind klicken
skylight-cli click --pid X --element-index Y  # ohne Vision-Prompt!

# ❌ Große Images
1200x1006 PNG direkt an API

# ❌ Max_tokens zu klein
max_tokens: 300  # JSON abgeschnitten!

# ❌ bash mit & (non-blocking)
command &  # blockiert shell!
```

## 🚫 RICHTIGE ALTERNATIVEN

```bash
# ✅ Koordinaten NIE — immer element-index
skylight-cli click --pid X --element-index Y

# ✅ Popups via cua-driver
cua-driver call click '{"pid":X,"window_id":W,"element_index":Y}'

# ✅ Vision-Prompt VOR jedem Klick
python3 runner/step.py "https://heypiggy.com/?page=dashboard"

# ✅ Image resize VOR API
Image.thumbnail((960, 960), Image.LANCZOS) → JPEG quality 40

# ✅ Max_tokens ≥ 1000
max_tokens: 1000  # für reasoning models

# ✅ Recovery Module statt recovery_mode
./modules/recovery-overquota/cli/recovery-overquota $PID
```

## 🚫 RECOVERY MODULE (nur wenn FAIL!)

Nicht blind starten. Erst Vision fragen: "Is recovery needed?"

```
modules/recovery-overquota/        # Nur bei "overquota" DQ
modules/recovery-attentioncheck/   # Nur bei Fangfrage-Detektion
```

## 🔒 SEMGREP BLOCKIERT COMMITS MIT BANNED MUSTERN

`semgrep --config=~/dev/stealth-runner/.semgrep_rules.yaml .`
Falls blocked: Patterns in `.semgrep_rules.yaml` checken.