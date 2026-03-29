# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Context

**Vaani AI** (वाणी — Voice of Intelligence) — Real-Time Multilingual Conversational AI Platform (B2B SaaS)

- **Product:** Cascaded STS pipeline — Speech → ASR → LLM → TTS → Speech
- **MVP verticals:** BFSI + Automotive (Phase 1); Government, Healthcare (Phase 2)
- **Target:** B2B customers — Startups, SMBs, Enterprise across India + global markets
- **Docs location:** `docs/` (all HTML files are self-contained, no build step)
- **Git remote:** `https://github.com/shubhamg2710/Vaani-AI.git`

---

## Repository Structure

```
docs/
├── index.html                        # Hub / navigation (dark Indic Minimalist theme)
├── 01_FRD.html                       # Functional Requirements (F1–F18)          ~103KB
├── 02_Design_Doc.html                # Technical Architecture & Tech Stack         ~82KB
├── 03_Call_Templates.html            # Call Flow Templates (Universal+BFSI+Auto)   ~63KB
├── 04_Call_Scripts.html              # 20 Call Scripts (12 BFSI + 8 Automotive)    ~96KB
├── 05_Knowledge_Bank.html            # RAG Training Data (BFSI + Automotive)       ~62KB
├── 06_Business_Document.html         # Executive / Investor Pitch                  ~75KB
├── 07_Competitive_Analysis.html      # 11-competitor analysis, live data           ~64KB
└── scripts/                          # 35 call scripts in plain-text (.txt)
    ├── 04_Call_Scripts_BFSI_01to12.txt
    ├── 04_Call_Scripts_Automotive_13to20.txt
    ├── BFSI_Loan_PreQualification_Scripts.txt
    ├── Automotive_Routine_Service_Booking_Scripts.txt
    └── Automotive_Sales_Lead_TestDrive_Scripts.txt
```

---

## Design System (all CSS must be inline — no external stylesheets)

**index.html** uses a dark Indic Minimalist AI theme:
- Background: `#061A1E` (deep teal-black)
- Cards: `#0D2B32` / `#112F37`
- Accent 1 — Saffron: `#F4923A`
- Accent 2 — Teal/Peacock: `#26C6DA`
- Accent 3 — Gold: `#E9C46A`
- Logo mark: `वा` in saffron-gold gradient

**All 6 doc pages** use a light Indic theme:
- Body background: `#FAF8F4` (warm ivory)
- Header gradient: `#0D3B47 → #0A8EA0` (peacock teal)
- Agent speech: `#DCF0F3` bg, `#0A8EA0` left border
- User speech: `#dcfce7` bg, `#22c55e` left border
- System actions: `#f3f4f6` bg, italic
- Hinglish variations: `#fdf4ff` bg, `#a855f7` border

**Floating nav bar** (`.vb-nav`) is present on all pages — fixed bottom-right:
- Doc pages: Home (`index.html`) + Back (`history.back()`) + Top (scroll)
- `index.html`: Back + Top only (no Home — it is home)

---

## Scripting on This Machine

Python is available as `python` (not `python3`):
```bash
python script.py
```
Windows paths use forward slashes in bash: `C:/Users/...`

Bulk HTML operations (colour replacement, nav injection, name rename) use Python directly:
```python
# Pattern for reading/writing HTML files
with open("C:/Users/shubhamgoel/Documents/AI_Coding/Personal/RTCA/Vaani-AI/docs/file.html", "r", encoding="utf-8") as f:
    content = f.read()
# ... modify content ...
with open(path, "w", encoding="utf-8") as f:
    f.write(content)
# Use print() with ASCII only — Windows cp1252 console chokes on ✔ ✗ etc.
```

---

## Multi-File HTML Checklist (run before telling user it's done)

### Creation
- Use `Write` tool directly for files — do **not** use `run_in_background: true` agents for file creation (they silently time out on large outputs)
- Foreground agents are safe; run all in parallel in one message
- After every Write: verify with `ls -lh docs/*.html`

### Post-creation verification
```bash
# All files exist and are non-trivial in size
ls -lh docs/*.html

# All HTML files are properly closed
for f in docs/*.html; do
  tail -3 "$f" | grep -q "</html>" && echo "OK: $f" || echo "INCOMPLETE: $f"
done

# Count numbered items (scripts, requirements) to catch truncation
grep -c "SCRIPT [0-9]" docs/04_Call_Scripts.html   # expect 20+
```

### Anchor link validation (run after any multi-part HTML assembly)
```bash
FILE="docs/04_Call_Scripts.html"
MISSING=$(comm -23 \
  <(grep -o 'href="#[^"]*"' "$FILE" | sed 's/href="#//;s/"//' | sort) \
  <(grep -o 'id="[^"]*"'   "$FILE" | sed 's/id="//;s/"//'   | sort))
[ -z "$MISSING" ] && echo "ALL ANCHORS VALID" || echo "BROKEN: $MISSING"
```
Root cause of broken anchors: Part 1 uses `id="s1"` but Part 2 appended later uses `id="script-13"`. Always use the shorter `id="s{N}"` convention.

### Index ordering
- `index.html` doc cards must be in sequential numeric order: 01, 02, 03 … 07
- Only add a card to `index.html` **after** confirming the target file exists

---

## Content Rules (no hallucination)

Call scripts and Knowledge Bank entries must use **real, verifiable data**:
- Real company names: SBI, HDFC Bank, Maruti Suzuki, Tata Motors, Hyundai India, etc.
- Real approximate rates: SBI home loan ~8.50% p.a., FD 7.00% (2-year), Nexon EV from ₹14.49L
- Real regulatory references: RBI chargeback SLA (7 working days), IRDAI survey SLA (7 working days), Section 80E
- Competitive data sourced from public disclosures — cite sources in the document footer

---

## Git Workflow

Repo root for git operations: `C:/Users/shubhamgoel/Documents/AI_Coding/Personal/RTCA/Vaani-AI`

```bash
git -C "C:/Users/shubhamgoel/Documents/AI_Coding/Personal/RTCA/Vaani-AI" add docs/ CLAUDE.md
git -C "C:/Users/shubhamgoel/Documents/AI_Coding/Personal/RTCA/Vaani-AI" commit -m "message"
git -C "C:/Users/shubhamgoel/Documents/AI_Coding/Personal/RTCA/Vaani-AI" push origin main
```

Do **not** add `.claude/` (local settings) to git.
