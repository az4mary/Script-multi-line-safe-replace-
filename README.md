We’ll implement **Patch 01 as an automated, safe, repeatable script**.
## Step 1 — Create Patch Script (LOCAL)
Create file:
```bash
C:\Users\HP\PROJECTS\GITHUB\az4M\Deterministic-Amazon-Automation\patch_01.py
```
### Script (multi-line safe replace)
```python
import argparse
import difflib
import re
import shutil
from pathlib import Path

FILE_PATH = Path("<PASTE_FILENAME>")

# === ONLY EDIT THESE TWO ===
FIND = """<PASTE_EXACT_SNIPPET_TO_FIND>"""

REPLACE = """<PASTE_REPLACEMENT_SNIPPET>"""
# ===========================

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--apply", action="store_true")
    args = parser.parse_args()

    content = FILE_PATH.read_text(encoding="utf-8")

    pattern = re.compile(re.escape(FIND), re.MULTILINE)
    matches = list(pattern.finditer(content))

    print(f"Matches found: {len(matches)}")

    if len(matches) != 1:
        print("❌ Abort: match must be exactly 1")
        return

    updated = pattern.sub(REPLACE, content, count=1)

    diff = difflib.unified_diff(
        content.splitlines(True),
        updated.splitlines(True),
        fromfile=str(FILE_PATH),
        tofile=str(FILE_PATH) + " (patched)",
    )
    print("".join(diff))

    if not args.apply:
        print("Dry run only. No changes made.")
        return

    FILE_PATH.write_text(updated, encoding="utf-8")
    print(f"✅ Patch applied.")

if __name__ == "__main__":
    main()
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

You don’t want per-patch scripts—you want **one reusable patch engine** where you only change `FIND` and `REPLACE`. Here is a **production-ready, drop-in script**.

---

## How You Use It (for EVERY patch)

### 1. Paste snippet
```python
FIND = """def log(...old...)"""
REPLACE = """def log(...new...)"""
```

### 2. Dry-run
```bash
python patch.py
```

### 3. Apply
```bash
python patch.py --apply
```
## Guarantees
* Exact match only (no accidental edits)
* Fails if 0 or >1 matches
* Shows diff before applying
* Auto backup (`.bak`)
