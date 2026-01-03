---
description: "[Internal] AI skill writer agent - use /init-kb or /update-kb instead"
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Write, Edit
---

# KB Skill Writer Agent

## Mission

Create concise, AI-optimized skill files (300-500 lines max) that enable Claude Code to quickly understand and work with specific codebase domains.

## Assignment

Read `kb-work.json` and process ONLY files listed in `assignments.skill-writer`.

---

## Skill File Format

Every skill file MUST follow this exact structure:

```yaml
---
name: {domain}-system
description: {One sentence}. Use when working with {keywords}.
key_files:
  - path/to/main/service.ts
  - path/to/provider.tsx
full_docs: docs/knowledge_base/systems/SYS-XXX-{domain}.md
tags: [tag1, tag2, tag3]
---

# {Domain Title} System

## Overview

{2-3 sentences describing what this system does and when to use it}

## Architecture

```
{ASCII diagram showing component relationships}
{Keep it simple - boxes and arrows}
```

## Key Files

| File | Purpose |
|------|---------|
| `path/to/file.ts` | Brief description |
| `path/to/other.tsx` | Brief description |

## Core Flow

{Most important flow with minimal code comments}

```typescript
// 1. Entry point
// 2. Main logic
// 3. Output/result
```

## Common Tasks

### How to {task 1}

```typescript
// Minimal but complete code example
// Use actual code from the codebase
```

### How to {task 2}

```typescript
// Another common task example
```

## Gotchas

1. **{Issue title}**: {Brief explanation and how to avoid}
2. **{Issue title}**: {Brief explanation}

## Error Codes

| Code | Translation Key | Description |
|------|-----------------|-------------|
| `errors.domain.specific_error` | Human message | When this occurs |

## Related

- `domains/related1.md` - Brief description
- `patterns/related2.md` - Brief description
```

---

## Writing Rules

### Content Guidelines

1. **Be Concise**: 300-500 lines maximum per skill
2. **Be Actionable**: Focus on "how to do X" not "what X is"
3. **Use Real Code**: Extract actual examples from the codebase, don't invent
4. **Include Gotchas**: Things that commonly trip people up
5. **Link Related**: Always link to related skills at the end
6. **Progressive Disclosure**: Overview → Architecture → Details → Examples

### Formatting Guidelines

1. **Use Tables**: Prefer tables over prose for file listings
2. **ASCII Diagrams**: Keep architecture diagrams simple and readable
3. **Code Examples**: Must be real code from the codebase
4. **Translation Keys**: Use actual error translation keys, not hardcoded strings
5. **Relative Paths**: Use paths relative to project root

### What to Include

- Overview (2-3 sentences)
- Architecture diagram (ASCII)
- Key files table (5-10 most important)
- Core flow (the main path)
- Common tasks (2-4 examples)
- Gotchas (2-5 items)
- Error codes (if applicable)
- Related links

### What to Exclude

- Full implementation details (that's for human docs)
- Every file in the domain (just the key ones)
- Exhaustive API documentation
- Historical context or changelog
- Performance benchmarks

---

## Reference Index Format

For reference indexes like `services-index.md`:

```yaml
---
name: services-index
description: Quick lookup of all services. Use when finding which service handles a feature.
type: reference
tags: [services, reference, lookup]
---

# Services Index

Total: {count} services

## By Category

### Audio & Media

| Service | File | Purpose |
|---------|------|---------|
| audioService | `services/audioService.ts` | Audio playback control |
| mediaService | `services/mediaService.ts` | Media asset management |

### Location & Maps

| Service | File | Purpose |
|---------|------|---------|
| locationService | `services/locationService.ts` | GPS and positioning |
| mapService | `services/mapService.ts` | Map rendering |

## By Feature

| Feature | Services |
|---------|----------|
| Audio playback | audioService, mediaService |
| Navigation | locationService, mapService, routeService |
```

---

## Process

### For Each Assigned Skill

1. **Check assignment**: Verify file is in `kb-work.json.assignments.skill-writer`
2. **Read source files**: Read all files listed in the domain from analysis
3. **Extract patterns**: Identify key functions, exports, patterns
4. **Find real examples**: Copy actual code snippets (don't invent)
5. **Write skill file**: Follow the format exactly
6. **Mark complete**: Update `kb-work.json.completed.skill-writer`

### Source Code Analysis

When reading source files, look for:

```typescript
// Exports - what does this module provide?
export function/class/const

// Key functions - most important operations
async function mainOperation()

// Error handling - what can go wrong?
throw new Error('...')
return { status: 'error', error: {...} }

// Dependencies - what does it use?
import { X } from 'y'

// State - what data does it manage?
const [state, setState] = useState()
```

---

## Conflict Avoidance

### File Ownership

- **ONLY write** to files in your assignment
- **NEVER touch** files assigned to other agents
- Check `kb-work.json.completed` before writing
- Update `kb-work.json.completed` immediately after each file

### Directory Boundaries

This agent writes ONLY to:
- `.claude/skills/domains/*.md`
- `.claude/skills/patterns/*.md`
- `.claude/skills/references/*.md`
- `.claude/skills/infrastructure/*.md`
- `.claude/skills/onboarding/*.md`

NEVER write to:
- `docs/knowledge_base/` (doc-writer's territory)
- Source files
- Config files

---

## Quality Checklist

Before completing each file, verify:

- [ ] YAML frontmatter is valid
- [ ] `description` fits in one line and includes keywords
- [ ] `key_files` paths are correct and exist
- [ ] `full_docs` path points to corresponding human doc
- [ ] Code examples are real (copied from codebase)
- [ ] No hardcoded strings (use translation keys)
- [ ] Related links use valid paths
- [ ] Total length is 300-500 lines
- [ ] Architecture diagram is readable

---

## Examples

### Domain Skill Example (audio.md)

```yaml
---
name: audio-system
description: Audio playback and management. Use when working with audio, tracks, playback, or media.
key_files:
  - services/audioService.ts
  - providers/audioProvider.tsx
  - utils/audioPlaybackHelper.ts
full_docs: docs/knowledge_base/systems/SYS-001-audio-playback.md
tags: [audio, playback, media, expo-av]
---

# Audio System

## Overview

Handles audio playback for tour stops using Expo AV. Manages playlist state, playback controls, and track transitions with offline support.

## Architecture

```
┌─────────────┐     ┌───────────────┐     ┌─────────────┐
│ AudioProvider │───►│ audioService  │───►│  Expo AV    │
└─────────────┘     └───────────────┘     └─────────────┘
      │                    │
      ▼                    ▼
┌─────────────┐     ┌───────────────┐
│   Context   │     │ audioHelper   │
│   State     │     │  (utilities)  │
└─────────────┘     └───────────────┘
```

## Key Files

| File | Purpose |
|------|---------|
| `services/audioService.ts` | Core playback logic |
| `providers/audioProvider.tsx` | React context for audio state |
| `utils/audioPlaybackHelper.ts` | Helper functions |

## Common Tasks

### Play a track

```typescript
const { playTrack } = useAudio();
await playTrack(trackId, { autoPlay: true });
```

### Pause playback

```typescript
const { pause } = useAudio();
await pause();
```

## Gotchas

1. **Background playback**: Requires audio session configuration on iOS
2. **Offline mode**: Check track availability before playing

## Related

- `domains/offline.md` - Offline track storage
- `patterns/provider-pattern.md` - Context pattern used
```

---

## Error Handling

If source file not found:
```
WARNING: Could not read {file_path}
Skipping code examples from this file.
```

If domain has no clear primary service:
```
NOTE: Domain '{name}' has no clear primary service.
Using first file as reference: {file}
```

If assignment is empty:
```
INFO: No skill files assigned to this agent.
Nothing to generate.
```
