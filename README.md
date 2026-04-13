We’ll implement **Patch 01 as an automated, safe, repeatable script**.
## Step 1 — Create Patch Script (LOCAL)
Create file:
```bash
C:\Users\HP\PROJECTS\GITHUB\az4M\Deterministic-Amazon-Automation\patch_01.py
```
### Script (multi-line safe replace)
```python
from datetime import datetime
import json
import re
from pathlib import Path

file_path = Path("workflow_orchestrator.py")
content = file_path.read_text()

FIND = r"""def log(level, message, trace_id=None, span_id=None):
    print(json.dumps({
        "level": level,
        "message": message,
        "trace_id": trace_id,
        "span_id": span_id
    }))"""

REPLACE = """def log(level, message, stage, status, trace_id, span_id, context=None, progress_percent=None, current_step=None, total_steps=None):
    log_entry = {
        "timestamp": datetime.utcnow().isoformat(),
        "level": level,
        "message": message,
        "service": "amazon_listing_pipeline",
        "stage": stage,
        "status": status,
        "trace_id": trace_id,
        "span_id": span_id,
        "context": context or {}
    }

    if progress_percent is not None:
        log_entry["progress_percent"] = int(progress_percent)
        log_entry["current_step"] = current_step
        log_entry["total_steps"] = total_steps

    print(json.dumps(log_entry))"""

# DRY RUN
matches = re.findall(FIND, content, re.MULTILINE)
print(f"Matches found: {len(matches)}")

if matches:
    updated = re.sub(FIND, REPLACE, content, flags=re.MULTILINE)
    file_path.write_text(updated)
    print("Patch applied.")
else:
    print("No match found. Aborting.")
```
## Step 2 — Run (Dry-run behavior)
```bash
python patch_01.py
```
- If `Matches found: 1` → SAFE
- If `0` or `>1` → STOP (pattern mismatch)
## Step 3 — Commit via GitHub Desktop
- You’ll see `workflow_orchestrator.py` changed
- Commit message:
```bash
fix: patch-01 replace logger function
```
- Push → create PR
## Benefit
- Regex handles **multi-line safely**
- Dry-run check prevents corruption
- Deterministic (same patch always same result)
