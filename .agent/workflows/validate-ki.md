---
description: Validate Knowledge Items
mode: VERIFICATION
exceptions: [READ_ONLY, EXTERNAL_TARGET]
read_when:
  - Checking KI accuracy against codebase
  - KIs may be stale or outdated
---
# Validate KI

Verify Knowledge Item artifacts are accurate. **Read-only—creates validation report.**

## 1. Locate KIs
```bash
# Auto-discover KI path
KI_PATH=""
[ -d ".agent/KI" ] && KI_PATH=".agent/KI"
[ -d ".gemini/ki" ] && KI_PATH=".gemini/ki"
[ -z "$KI_PATH" ] && echo "⛔ No KI directory found" && exit 1
ls -la "$KI_PATH/"
```

## 2. Audit Each KI
```bash
grep -oE '{SRC_DIR}/[a-zA-Z0-9/_.-]+' "$KI_PATH"/*.md 2>/dev/null | while read f; do
  [ ! -f "$f" ] && echo "STALE: $f"
done
```
Check for: terminology drift, contradictions (KI says X, code does Y), missing info.

## 3. Report
```markdown
## KI Validation Report
### {KI Name}
**Path**: `$KI_PATH/{ki_folder}`
| Issue | Type | Details |
|-------|------|---------|
| `oldFunction` | Stale reference | Renamed to `newFunction` |
```

## 4. After Validation
Issues → Update KI manually | Clean → Document

## 5. Post-Fix Validation
```bash
grep -oE '\[.*\]\(file://[^)]+\)' "$KI_PATH"/*.md 2>/dev/null | while read link; do
  path=$(echo "$link" | grep -oE 'file://[^)]+' | sed 's|file://||')
  [ ! -f "$path" ] && echo "BROKEN LINK: $path"
done
```
