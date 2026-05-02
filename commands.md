# commands.md — Alle CLI-Befehle für stealth-skills

## Google Login Flow (cua-driver Popup)

```bash
# 1. Chrome starten
playstealth launch --url 'https://heypiggy.com/?page=dashboard'
PID=$!

# 2. cua-driver Daemon (einmalig pro Session)
cua-driver serve &

# 3. Google Login Button (Hauptfenster)
skylight-cli click --pid $PID --element-index 33
sleep 2

# 4. Popup Window-ID finden
WID=$(cua-driver call list_windows '{}' | python3 -c "
import sys,json
data=json.load(sys.stdin)
for w in data.get('windows',[]):
    if w.get('pid')==$PID:
        t=(w.get('title','')).lower()
        if 'anmelden' in t or 'sign' in t or 'google' in t:
            print(w['window_id'])
" 2>/dev/null | head -1)
echo "Popup WID=$WID"

# 5. Email eintippen (element_index 25)
cua-driver call set_value "{\"pid\":$PID,\"window_id\":$WID,\"element_index\":25,\"value\":\"zukunftsorientierte.energie@gmail.com\"}"

# 6. "Weiter" klicken (element_index 35)
cua-driver call click "{\"pid\":$PID,\"window_id\":$WID,\"element_index\":35,\"action\":\"press\"}"
sleep 2

# 7. "Fortfahren" klicken (Consent, element_index 65)
cua-driver call click "{\"pid\":$PID,\"window_id\":$WID,\"element_index\":65,\"action\":\"press\"}"
sleep 2

# 8. Finales "Weiter" → Dashboard (element_index 41)
cua-driver call click "{\"pid\":$PID,\"window_id\":$WID,\"element_index\":41,\"action\":\"press\"}"
```

## Dashboard Survey Discovery

```bash
# Elements scannen
skylight-cli list-elements --pid $PID | python3 -c "
import json,sys
data=json.load(sys.stdin)
for e in data.get('elements',[]):
    l=e.get('label','')
    if any(x in l for x in ['Umfrage','€','EUR','starten','teilnehmen']):
        print(f'[{e[\"index\"]}] {e[\"role\"]}: {l[:80]}')
"

# Screenshot
skylight-cli screenshot --pid $PID --mode som --output /tmp/dashboard.png
```

## Module Commands

```bash
# Router-Detektor (welches Panel?)
./platforms/heypiggy/modules/router-detector/cli/router-detector $PID

# Question Modules
./platforms/heypiggy/modules/question-slider/cli/question-slider-set $PID <element_index> <value>
./platforms/heypiggy/modules/question-ranking/cli/question-ranking-solve $PID
./platforms/heypiggy/modules/question-matrix/cli/question-matrix-solve $PID
./platforms/heypiggy/modules/question-opentext/cli/question-opentext-answer $PID

# Recovery Modules
./platforms/heypiggy/modules/recovery-overquota/cli/recovery-overquota $PID
./platforms/heypiggy/modules/recovery-attentioncheck/cli/recovery-attentioncheck $PID

# History
./platforms/heypiggy/modules/heypiggy-history/cli/heypiggy-history $PID
```

## Video Recording

```bash
# Screen aufnehmen
screen-follow record --video --output /tmp/session.mp4

# Video analysieren (nach Session)
python3 -m runner.video_analyzer --last flow
```

## Live Vision (stealth-runner)

```bash
# Live Eye v7
PYTHONPATH=~/dev/stealth-runner python3 -c "
from runner.live_eye import LiveEye
eye = LiveEye(fps=5)
eye.start()
"

# Live Omni Monitor
PYTHONPATH=~/dev/stealth-runner python3 -c "
from runner.live_omni_monitor import LiveOmniMonitor
m = LiveOmniMonitor(fps=1.0, debug=True)
m.start('https://heypiggy.com/?page=dashboard')
m.run_continuous(max_steps=100)
"

# Step Orchestrator
PYTHONPATH=~/dev/stealth-runner python3 runner/step.py "https://heypiggy.com/?page=dashboard"
```

## Knowledge Graph

```bash
graphify query "Google OAuth flow heypiggy"
graphify path "LiveOmniMonitor" "StealthExecutor"
graphify update ~/dev/stealth-skills
```

## /doctor Audit

```bash
python3 ~/dev/stealth-runner/runner/doctor_cli.py
```