---
description: Post-mortem on completed work - reflect, extract patterns, close the learning loop
---

Retrospective mode. After completing a task or bead, reflect on what happened and extract reusable learnings.

## Usage

```
/retro <bead-id>              # Retro on specific completed bead
/retro --session              # Retro on current session's work
/retro --extract-patterns     # Also update knowledge files with discoveries
```

The argument is: $ARGUMENTS

## Step 1: Gather Context

### If bead-id provided:

```bash
# Get bead details
bd show $BEAD_ID --json

# Get related commits
git log --oneline --all --grep="$BEAD_ID" | head -10

# Find files changed in those commits
git log --all --grep="$BEAD_ID" --name-only --pretty=format: | sort -u | head -20
```

### If --session or no argument:

```bash
# What was completed this session?
bd list --status closed --json | head -10

# What's still in progress?
bd list --status in_progress --json

# Recent commits
git log --oneline -20

# Files touched recently
git diff --stat HEAD~10 2>/dev/null || git log --name-only --pretty=format: -10 | sort -u
```

## Step 2: Reconstruct the Journey

From the gathered context, reconstruct:

1. **Starting Point** - What was the original goal/description?
2. **Path Taken** - What commits were made? What files changed?
3. **Detours** - Any discovered issues filed? Scope changes?
4. **Endpoint** - What was actually delivered?

Check for child beads or discovered-from links:

```bash
# If epic, get children
bd list --parent $BEAD_ID --json 2>/dev/null || true

# Check for discoveries
bd list --json | jq '.[] | select(.dependencies[]? | .target == "'$BEAD_ID'" and .type == "discovered-from")' 2>/dev/null || true
```

## Step 3: Structured Reflection

Answer these questions honestly:

### What was the goal?

- Original scope from the bead/task description
- Any implicit assumptions made at the start

### What actually happened?

- Final state vs intended state
- Time estimate vs actual (if known)
- Scope creep or scope reduction?

### What went well?

Look for:

- Approaches that worked smoothly
- Tools/patterns that saved time
- Good decisions made early
- Successful debugging strategies

### What went poorly?

Look for:

- Time sinks / rabbit holes
- Wrong initial assumptions
- Repeated mistakes
- Missing context that caused delays

### What was surprising?

- Unexpected complexity
- Things that were easier than expected
- Hidden dependencies discovered
- Codebase quirks encountered

### What would you do differently?

- Better starting point
- Different approach
- Earlier escalation
- More/less planning

### Patterns discovered?

- Reusable code patterns
- Error patterns to document
- Process improvements
- Tool usage insights

## Step 4: Output the Retro

Format the retrospective:

```markdown
## Retro: <bead-id or "Session YYYY-MM-DD">

### Goal

<what we set out to do - quote original description if bead>

### Outcome

<what actually happened, delta from goal>

### What Went Well

- **<thing>**: <why it worked, how to replicate>

### What Went Poorly

- **<thing>**: <what went wrong, how to avoid>

### Surprises

- <unexpected discovery or finding>

### Patterns Discovered

- **<pattern name>**: <brief description, when to apply>

### Action Items

- [ ] <concrete thing to do differently next time>
```

## Step 5: Extract Patterns (if --extract-patterns)

If `--extract-patterns` flag is present OR significant patterns were discovered:

### For Error Patterns:

If new error patterns were encountered and resolved:

```bash
# Check if pattern already exists
rg -i "<error keyword>" ~/.config/opencode/knowledge/error-patterns.md
```

If novel, offer to add:

```
Found novel error pattern: <pattern name>
Add to ~/.config/opencode/knowledge/error-patterns.md? (y/n)
```

Format for error-patterns.md:

```markdown
### <Error Name>

**Pattern:** `<searchable error fragment>`

**Common Causes:**

- <what caused it>

**Fixes:**
\`\`\`typescript
<the fix>
\`\`\`

**Prevention:** <how to avoid>
```

### For Code Patterns:

If reusable code patterns were discovered:

1. Identify the appropriate knowledge file or create new one
2. Format the pattern with:
   - Name
   - When to use
   - Example code
   - Gotchas

### For Action Items:

Create beads for non-trivial action items:

```bash
bd create "<action item>" -t chore -p 2 --tags retro-action --json
```

## Step 6: Update the Bead (if bead-id provided)

Add retro notes to the bead:

```bash
# Get current description
CURRENT=$(bd show $BEAD_ID --json | jq -r '.description')

# Append retro summary (abbreviated)
bd update $BEAD_ID -d "$CURRENT

---
**Retro Notes:**
- Went well: <key thing>
- Watch out: <key thing>
- Pattern: <if any>"

# Add tag
bd tag $BEAD_ID retro-complete 2>/dev/null || true
```

## Step 7: Sync

```bash
bd sync
git push 2>/dev/null || true
```

## Output Summary

End with:

```markdown
---

**Retro complete.**

Patterns extracted: <count or "none">
Action items filed: <count or "none">
Knowledge updated: <files or "none">

Key takeaway: <one sentence summary of the most important learning>
```

---

**Remember:** The point isn't documentation theater - it's closing the learning loop. A retro that doesn't change future behavior is worthless. Focus on actionable insights.
