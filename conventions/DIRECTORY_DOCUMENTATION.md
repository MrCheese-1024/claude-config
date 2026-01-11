# Directory Documentation Strategy

This document guides documentation synchronization (doc-sync) and CLAUDE.md creation for different directory types in this repository.

## Core Principle

**Directory type determines CLAUDE.md depth and content style.**

Different directories have different purposes and change frequencies. Documentation depth should match the stability of the directory content.

## Three Directory Types

### Type 1: STABLE (Code, Architecture, Core Systems)

**Examples:** `scripts/`, `scenes/`, `systems/`, `components/`, `spells/`

**Characteristics:**
- Code files rarely change structure
- Dependencies and relationships are stable
- Needs discoverable index for navigation
- Integration points matter

**CLAUDE.md Treatment:**
- ✅ DO create comprehensive table-based index
- ✅ DO enumerate all files and subdirectories
- ✅ DO explain integration points and relationships
- ✅ DO update when files are added/removed
- ⚠️ COST: Moderate maintenance burden, but prevents knowledge loss

**Template:**
```markdown
# [directory]/

[One-sentence purpose]

## Files

| File | What | When to read |
|------|------|--------------|
```

---

### Type 2: FLUID (Planning, Specifications, Project Tracking)

**Examples:** `project/features/`, `project/plans/`, `project/features/draft/`

**Characteristics:**
- Content changes frequently
- Items are added/removed/reorganized constantly
- Enumeration becomes stale quickly
- Workflow and process matter more than inventory

**CLAUDE.md Treatment:**
- ✅ DO keep lightweight and instruction-focused
- ✅ DO document the workflow (how to add/maintain items)
- ✅ DO reference single sources of truth for status tracking
- ❌ DO NOT enumerate current items (list becomes stale immediately)
- ❌ DO NOT maintain detailed index tables
- ⚠️ COST: Minimal—rely on folder structure and naming conventions

**Template:**
```markdown
# [directory]/

[One-sentence purpose]

## Workflow

[Steps to add/maintain items]

## Single Source of Truth

Use `[reference file]` for tracking status.
```

---

### Type 3: MINIMAL (Assets, Resources, Generated)

**Examples:** `audio/`, `resources/roguelike/artifacts/`, `assets/icons/`, `bin/`

**Characteristics:**
- Reference materials (sounds, textures, data files)
- May be generated or imported from external sources
- Enumeration less useful than description

**CLAUDE.md Treatment:**
- ✅ DO create minimal reference CLAUDE.md
- ✅ DO explain what files do and how they're used
- ❌ DO NOT enumerate every file
- ❌ DO NOT try to maintain detailed indexes
- ⚠️ COST: Very low—few updates needed

**Template:**
```markdown
# [directory]/

[Purpose and file format]

## Integration
How files are used in the codebase.
```

---

## Decision Tree for Doc-Sync

**Is the directory's content primarily CODE?**
- YES → STABLE type (comprehensive index)
- NO → Continue

**Does the directory content change frequently?**
- YES → FLUID type (workflow-focused, no enumeration)
- NO → Continue

**Is this asset/resource reference material?**
- YES → MINIMAL type (light reference)
- NO → Treat as STABLE (comprehensive index)

---

## Examples by Type

### STABLE Examples

| Directory | Reason | Treatment |
|-----------|--------|-----------|
| `scripts/systems/` | Core spell and game systems | Comprehensive index, enum all files |
| `scripts/components/` | Reusable AI components | Comprehensive index, explain relationships |
| `scenes/characters/` | Character scene definitions | Comprehensive index, integration notes |

### FLUID Examples

| Directory | Reason | Treatment |
|-----------|--------|-----------|
| `project/features/` | Feature specs change constantly | Workflow instructions |
| `project/plans/` | Plans move between active/done | Workflow instructions, no enumeration |
| `project/features/draft/` | Draft ideas come and go | Lightweight, point to /define-feature skill |

### MINIMAL Examples

| Directory | Reason | Treatment |
|-----------|--------|-----------|
| `audio/` | Sound effects (imported/generated) | Reference list, integration notes |
| `assets/icons/` | UI icons (many placeholders) | Format description, no enumeration |
| `resources/roguelike/artifacts/` | .tres data files (auto-discovered) | Brief description, integration pattern |

---

## Implementation Guidelines for Doc-Sync

When running doc-sync:

1. **Classify each directory** using the decision tree above
2. **Apply appropriate CLAUDE.md template** for that type
3. **For STABLE:** Create/update comprehensive indexes
4. **For FLUID:** Create/update workflow instructions only
5. **For MINIMAL:** Create brief reference documentation

**Critical:** Do NOT force stable-type documentation on fluid directories. This creates stale, misleading documentation and wastes maintenance effort.

---

## Anti-Patterns for FLUID Directory Documentation

When creating/maintaining CLAUDE.md for fluid directories like `project/features/` and `project/plans/`, avoid these patterns:

### ❌ DO NOT enumerate current items

```markdown
# Features

- Feature A: Description
- Feature B: Description
- Feature C: Description
```

**Why:** Items are constantly added, removed, reorganized. This list becomes stale within hours.

**Instead:** Reference the canonical status tracking file:

```markdown
We do NOT maintain a detailed enumeration of features in this CLAUDE.md. Individual features track their status by file location and in the file
```

### ❌ DO NOT maintain detailed index tables for active items

```markdown
| Feature | Status | Owner | Link |
| ... | ... | ... | ... |
```

**Why:** This duplicates the work of tracking tools and drifts from source of truth.

**Instead:** Keep index tables ONLY in the canonical files.

### ❌ DO NOT duplicate status information

Don't track it in `project/plans/CLAUDE.md` or `project/features/CLAUDE.md`. Document once, reference everywhere.

### ✅ DO focus on workflow and process

```markdown
## When Creating a Plan
1. Create markdown file with task breakdown
2. Track progress as tasks complete
...

## When Completing a Plan
1. Move completed plan to `done/`
2. Add 1-2 sentence summary
```

**Why:** Workflow instructions don't become stale. They guide behavior regardless of current inventory.

---

## Updating This Strategy

If a directory's characteristics change (e.g., code moves from `project/` to `scripts/`):
- Reclassify the directory type
- Migrate CLAUDE.md to appropriate template
- Update this reference document
