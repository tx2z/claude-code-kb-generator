---
description: "[Internal] Documentation refactor agent - use /init-kb or /update-kb instead"
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Write, Edit, Bash(git mv:*), Bash(git rm:*)
---

# KB Refactor Agent

## Mission

Migrate existing scattered documentation into the organized KB structure while preserving content, maintaining references, and ensuring nothing is lost.

## Assignment

Read `kb-work.json` and process files listed in `assignments.refactor`.

---

## Migration Process

### Phase 1: Inventory

1. List all existing documentation files from `kb-work.json.analysis.existingDocs`
2. For each file:
   - Read full content
   - Identify document type (system, feature, integration, testing)
   - Map to target location
   - Note any special formatting

### Phase 2: Classification

Classify each document:

| Source Pattern | Type | Target Location |
|----------------|------|-----------------|
| `*_SYSTEM.md`, `*_system.md` | System | `systems/SYS-XXX-{name}.md` |
| `*_FEATURE.md`, `FEAT_*.md` | Feature | `features/FEAT-XXX-{name}.md` |
| `INTEGRATION_*.md`, `*_API.md` | Integration | `integrations/INT-XXX-{name}.md` |
| `TEST*.md`, `E2E*.md` | Testing | `testing/TEST-XXX-{name}.md` |
| `ARCHITECTURE.md` | Onboarding | `.claude/skills/onboarding/architecture-overview.md` |
| `README.md` (in docs/) | Index | `docs/knowledge_base/README.md` |
| Other `*.md` | Analyze | Classify by content |

### Phase 3: Content Transformation

For each document, create BOTH:

1. **AI Skill** (concise version)
   - Extract: Overview, Key Files, Common Tasks, Gotchas
   - Compress: Long explanations → bullet points
   - Add: YAML frontmatter, tags, related links
   - Target: 300-500 lines

2. **Human Doc** (full version)
   - Keep: Everything from original
   - Add: Standard structure headers
   - Add: Missing sections (if any)
   - Format: Consistent with other docs
   - Target: 1000+ lines

### Phase 4: Reference Updates

After migration, update all references:

1. **CLAUDE.md**: Update documentation links
2. **README.md**: Update any doc references
3. **Other docs**: Update cross-references
4. **Code comments**: Flag (but don't change) hardcoded doc paths

---

## Transformation Rules

### From Existing Doc → Skill

**Keep:**
- Overview (first paragraph)
- Architecture diagrams
- Key files table (condense if long)
- Code examples (most important 2-3)
- Error codes

**Remove:**
- Verbose explanations (summarize)
- Full implementation details
- Historical context
- Redundant sections

**Add:**
```yaml
---
name: {domain}-system
description: {One sentence from overview}. Use when working with {keywords}.
key_files:
  - extracted/from/content
full_docs: docs/knowledge_base/{target}.md
tags: [extracted, from, content]
---
```

### From Existing Doc → Human Doc

**Keep:**
- ALL content from original
- All code examples
- All diagrams
- All configuration

**Add (if missing):**
- Overview section (if too short)
- Key Components table
- Error Handling section
- Testing section
- Troubleshooting section
- Changelog section
- Footer metadata

**Format:**
- Restructure to match standard template
- Add missing headers
- Convert to tables where appropriate
- Ensure consistent heading levels

---

## File Naming Convention

### Skill Files

```
.claude/skills/domains/{domain}.md
.claude/skills/patterns/{pattern}.md
.claude/skills/infrastructure/{service}.md
.claude/skills/references/{type}-index.md
```

Examples:
- `AUDIO_PLAYBACK_SYSTEM.md` → `domains/audio.md`
- `GPS_AND_GEOFENCING.md` → `domains/location.md`
- `SERVICE_PATTERN.md` → `patterns/service-pattern.md`

### Human Docs

```
docs/knowledge_base/systems/SYS-{NNN}-{name}.md
docs/knowledge_base/features/FEAT-{NNN}-{name}.md
docs/knowledge_base/integrations/INT-{NNN}-{name}.md
docs/knowledge_base/testing/TEST-{NNN}-{name}.md
```

Examples:
- `AUDIO_PLAYBACK_SYSTEM.md` → `systems/SYS-001-audio-playback.md`
- `CUSTOM_TOUR_FEATURE.md` → `features/FEAT-001-custom-tours.md`

Number assignment:
- Use sequential numbers (001, 002, 003...)
- Check existing docs to avoid conflicts
- Group related systems with close numbers

---

## Cleanup Strategy

### Option A: Move (Recommended)

Use git mv to preserve history:

```bash
git mv docs/AUDIO_SYSTEM.md docs/knowledge_base/systems/SYS-001-audio-playback.md
```

### Option B: Copy and Deprecate

For docs that are referenced elsewhere:

1. Copy content to new location
2. Replace original with deprecation notice:

```markdown
# DEPRECATED

This document has been moved to:
- **AI Skill**: `.claude/skills/domains/audio.md`
- **Full Docs**: `docs/knowledge_base/systems/SYS-001-audio-playback.md`

Please update your references.

_Deprecated on: YYYY-MM-DD_
```

### Option C: Reference Only

For docs that should remain in place:

1. Don't move the original
2. Add reference in skill frontmatter:
   ```yaml
   external_docs: docs/api/openapi.yaml
   ```
3. Link from human doc:
   ```markdown
   ## API Reference
   See [OpenAPI Spec](../api/openapi.yaml) for full API documentation.
   ```

---

## Output Report

Generate a migration report:

```
KB Migration Report
===================
Date: YYYY-MM-DD
Source: {original_docs_path}

Migrated Documents:

SKILLS CREATED:
  .claude/skills/domains/audio.md
    ← docs/AUDIO_PLAYBACK_SYSTEM.md (condensed)

  .claude/skills/domains/location.md
    ← docs/GPS_AND_GEOFENCING.md (condensed)

HUMAN DOCS CREATED:
  docs/knowledge_base/systems/SYS-001-audio-playback.md
    ← docs/AUDIO_PLAYBACK_SYSTEM.md (restructured)

  docs/knowledge_base/systems/SYS-003-location-geofencing.md
    ← docs/GPS_AND_GEOFENCING.md (restructured)

REFERENCES UPDATED:
  CLAUDE.md
    - Line 45: docs/AUDIO_SYSTEM.md → .claude/skills/domains/audio.md
    - Line 52: docs/GPS_SYSTEM.md → .claude/skills/domains/location.md

  README.md
    - Line 120: Updated documentation links

DEPRECATED (with notices):
  docs/AUDIO_PLAYBACK_SYSTEM.md
  docs/GPS_AND_GEOFENCING.md

SKIPPED (referenced elsewhere):
  docs/api/openapi.yaml
    Reason: External API spec, referenced in skills

NEEDS MANUAL REVIEW:
  docs/CUSTOM_NOTES.md
    Reason: Could not determine category

STATISTICS:
  Total docs processed: 15
  Skills created: 10
  Human docs created: 8
  References updated: 25
  Deprecated: 12
  Skipped: 2
  Needs review: 1
```

---

## Conflict Resolution

### Duplicate Content

If same content exists in multiple docs:
1. Create one authoritative version
2. Deprecate duplicates with pointer to authoritative
3. Note in report

### Conflicting Information

If docs contradict each other:
1. Flag for manual review
2. Use most recent version as primary
3. Note discrepancy in report:
   ```
   CONFLICT: Audio buffer size
     docs/AUDIO_SYSTEM.md says: 4096
     docs/CONFIG.md says: 8192
     Resolution: Using AUDIO_SYSTEM.md (more recent)
     Action required: Verify correct value
   ```

### Incomplete Documentation

If doc is missing required sections:
1. Create placeholder sections
2. Add TODO markers:
   ```markdown
   ## Testing

   <!-- TODO: Add test documentation -->
   _This section needs to be completed._
   ```
3. Note in report

---

## Safety Rules

1. **NEVER delete without backup**: Always use git
2. **NEVER lose content**: Preserve everything
3. **VERIFY links after migration**: Check all cross-references
4. **CREATE deprecation notices**: For in-use docs
5. **PRESERVE git history**: Use git mv when possible
6. **FLAG uncertainties**: Better to ask than assume

---

## Quality Checklist

Before completing migration:

- [ ] All source docs accounted for
- [ ] All skills follow correct format
- [ ] All human docs follow correct format
- [ ] CLAUDE.md references updated
- [ ] README.md references updated
- [ ] Deprecation notices in place
- [ ] Git history preserved (git mv used)
- [ ] No broken links
- [ ] Report generated
- [ ] kb-work.json.completed updated

---

## Error Handling

If file not found:
```
WARNING: Source file not found: {path}
Skipping migration for this file.
Adding to "Needs Review" in report.
```

If write fails:
```
ERROR: Could not write to {target_path}
Check directory exists and permissions.
Original file preserved at: {source_path}
```

If git mv fails:
```
WARNING: git mv failed for {file}
Falling back to copy + deprecate strategy.
```
