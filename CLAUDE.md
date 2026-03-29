# RTCA Project — Claude Code Instructions & Checklists

## Project Context
VoxBharat — Real-Time Multilingual Conversational AI Platform (B2B SaaS)
- MVP: Cascaded STS (Speech→ASR→LLM→TTS→Speech)
- Industries: BFSI + Automotive (Phase 1), others (Phase 2)
- Docs location: `RTCA/docs/`

---

## CRITICAL CHECKLIST — Before Creating Multi-File Deliverables

When asked to create multiple HTML/file deliverables in one request, follow this checklist:

### ✅ Pre-Creation
- [ ] Create the target directory first (`mkdir -p`) and verify it exists
- [ ] List any existing files in that directory before starting
- [ ] Plan ALL file names upfront and confirm with user if uncertain

### ✅ File Creation
- [ ] **Use Write tool directly** for files < 50KB — do NOT rely on background agents for file creation
- [ ] **Use foreground agents** (not background) when delegating large file writes — background agents time out silently
- [ ] If using parallel foreground agents: run all in a single message, wait for ALL to complete before responding
- [ ] After each Write/agent completion: immediately verify file exists with `ls`

### ✅ Post-Creation Verification (ALWAYS run before telling user it's done)
```bash
# 1. Confirm all expected files exist
ls -lh /path/to/docs/*.html

# 2. Confirm no file is 0 bytes or suspiciously small
# (< 5KB for a "comprehensive" doc = likely failed/truncated)

# 3. Confirm all HTML files are properly closed
for f in /path/*.html; do
  echo -n "$f: "
  tail -3 "$f" | grep -q "</html>" && echo "COMPLETE" || echo "INCOMPLETE ⚠️"
done

# 4. For documents with numbered content (scripts, requirements), count them
grep -c "SCRIPT [0-9]" file.html  # should match expected count
```

### ✅ Index / Navigation Pages
- [ ] Only add links in index.html AFTER confirming target files exist
- [ ] Documents must be listed in sequential numeric order (01, 02, 03... not 01, 02, 06, 03)
- [ ] Test that href values exactly match actual filenames (case-sensitive on Linux)
- [ ] "File not found" errors = href mismatch or file not yet created

### ✅ Anchor Link Validation (ALWAYS run for HTML with Table of Contents)
When an HTML file has a TOC with `href="#anchorId"` links AND script/section divs with `id="anchorId"`,
the IDs **must match exactly** — especially when content is written in multiple parts or by different agents.

**Root cause of broken anchors:** Part 1 uses `id="s1"` style; Part 2 appended later uses `id="script-13"` style → TOC links `#s13` break.

**Validation command — run after every multi-part HTML file is assembled:**
```bash
FILE="/path/to/file.html"

# Extract TOC hrefs (e.g. href="#s13" → s13)
TOC=$(grep -o 'href="#[^"]*"' "$FILE" | sed 's/href="#//;s/"//' | sort)

# Extract div/section IDs
IDS=$(grep -o 'id="[^"]*"' "$FILE" | sed 's/id="//;s/"//' | sort)

# Find TOC links with no matching ID
MISSING=$(comm -23 <(echo "$TOC") <(echo "$IDS"))
if [ -z "$MISSING" ]; then
  echo "ALL ANCHOR LINKS VALID ✔"
else
  echo "BROKEN ANCHORS ⚠️: $MISSING"
fi
```

**Fix pattern:** If TOC uses `#s13` but div has `id="script-13"`, pick one convention and align both.
Prefer the shorter `id="s{N}"` form (matches existing Part 1 style in this project).

### ✅ Content Completeness
- [ ] If an agent returns "exceeded token limit" or "connection refused": the file may be incomplete
  - Check file size vs expected
  - Check for cut-off markers like `<!-- PART 2 CONTINUES BELOW -->`
  - Append missing content manually with Edit tool
- [ ] For numbered sequences (20 scripts, 18 requirements): always grep-count after creation

---

## Document Map — RTCA/docs/

| File | Purpose | Size |
|------|---------|------|
| `index.html` | Hub / navigation | ~14KB |
| `01_FRD.html` | Functional Requirements (F1–F18) | ~102KB |
| `02_Design_Doc.html` | Technical Architecture & Stack | ~81KB |
| `03_Call_Templates.html` | Call Flow Templates (Universal + BFSI + Auto) | ~62KB |
| `04_Call_Scripts.html` | 20 Call Scripts (12 BFSI + 8 Automotive) | ~95KB |
| `05_Knowledge_Bank.html` | RAG Training Data (BFSI + Automotive) | ~61KB |
| `06_Business_Document.html` | Executive/Investor Pitch | ~73KB |

---

## Agent Usage Rules (Lessons Learned)

1. **Background agents (`run_in_background: true`)** — Do NOT use for file creation tasks. They silently time out on large outputs. Use only for research/read-only tasks.
2. **Foreground agents** — Safe for file creation, but run all in parallel in one message to avoid sequential blocking.
3. **Token limit errors** — If an agent hits "exceeded 32000 output tokens", the file will be truncated. Always verify completeness post-run.
4. **Connection refused errors** — Transient; retry the specific agent/Write that failed.
5. **Large files (>50KB HTML)** — Better to Write directly with content than delegate to an agent.

---

## Coding Style for This Project

- All HTML documents: light theme, white cards, navy-to-blue gradient header (#1a3a5c → #2563eb)
- Agent speech: blue bg (#dbeafe), blue left border (#3b82f6)
- User speech: green bg (#dcfce7), green left border (#22c55e)
- System actions: gray bg (#f3f4f6), italic text
- Hinglish variations: purple bg (#fdf4ff), purple border (#a855f7)
- All CSS inline (no external stylesheets) — documents must be self-contained
