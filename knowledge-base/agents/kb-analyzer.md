---
description: "[Internal] Codebase analyzer agent - use /init-kb or /update-kb instead"
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash(git:*), Bash(ls:*), Bash(find:*), Bash(wc:*)
---

# KB Analyzer Agent

## Mission

Analyze codebase to produce structured analysis for KB generation. This agent runs first and produces the `kb-work.json` file that other agents consume.

## Mode: $MODE

The mode is passed via environment or context:
- `init` - Full codebase analysis for initial KB creation
- `update` - Git-based change analysis for incremental updates

---

## Init Mode Workflow

### Step 1: Detect Tech Stack

Check for configuration files to identify the tech stack:

```
Priority order:
1. package.json + expo/app.json → react-native-expo
2. package.json + next.config.* → nextjs
3. package.json + nuxt.config.* → nuxt
4. package.json + angular.json → angular
5. package.json (generic) → nodejs
6. pyproject.toml/requirements.txt + django → django
7. pyproject.toml/requirements.txt + fastapi → fastapi
8. pyproject.toml/requirements.txt + flask → flask
9. pyproject.toml (generic) → python
10. go.mod → go
11. Cargo.toml → rust
12. pom.xml → java-maven
13. build.gradle → java-gradle
```

Read the primary config file and extract:
- Language (typescript, javascript, python, go, rust, java)
- Framework (expo, nextjs, django, fastapi, gin, etc.)
- Package manager (npm, yarn, pnpm, pip, poetry, cargo, etc.)
- Test framework (jest, pytest, go test, etc.)

### Step 2: Map Project Structure

Identify standard directories:

```
Source directories:
- src/, app/, lib/, pkg/, internal/, cmd/

Feature patterns:
- services/, api/, handlers/ → Service layer
- components/, views/, templates/ → UI layer
- providers/, contexts/, store/, redux/ → State layer
- models/, entities/, schemas/, types/ → Data layer
- utils/, helpers/, common/, shared/ → Utilities
- config/, settings/ → Configuration
- tests/, __tests__/, spec/, *_test.go → Testing
- hooks/ → React hooks (if React)
- middleware/ → Middleware (if applicable)
```

Count files in each directory using:
```bash
find <dir> -type f -name "*.ts" -o -name "*.tsx" | wc -l
```

### Step 3: Discover Feature Domains

Group files by feature prefix/suffix:

```
Pattern matching:
- services/audio*.ts + providers/audio*.tsx → "audio" domain
- services/location*.ts + utils/map*.ts → "location" domain
- services/auth*.ts + providers/auth*.tsx → "auth" domain
```

For each domain, identify:
- Primary service files
- Related provider/context files
- Utility files
- Test files
- Configuration

### Step 4: Detect Patterns

Search for common patterns in the codebase:

```typescript
// ServiceResponse pattern
grep -r "ServiceResponse" --include="*.ts"

// Provider pattern
grep -r "createContext\|useContext" --include="*.tsx"

// Error handling pattern
grep -r "try.*catch\|\.catch\(" --include="*.ts"

// Logging pattern
grep -r "LoggerService\|console\.log\|logger\." --include="*.ts"
```

Record:
- Pattern name
- Files where detected
- Confidence level (high/medium/low based on occurrence count)

### Step 5: Find Existing Documentation

Search for documentation files:

```bash
find . -name "*.md" -not -path "./node_modules/*" -not -path "./.git/*"
find . -type d -name "docs" -o -name "documentation" -o -name "wiki"
```

For each doc file, analyze:
- Location
- Size (line count)
- Apparent purpose (from filename/content)
- Last modified date

---

## Update Mode Workflow

### Step 1: Load Previous Analysis

Read `kb-meta.json` to get:
- Last analyzed commit hash
- Tech stack (already detected)
- File-to-documentation mappings

### Step 2: Analyze Git Changes

```bash
# Get changed files since last analysis
git log --oneline --name-status $LAST_COMMIT..HEAD

# Get detailed diff
git diff $LAST_COMMIT..HEAD --name-only
```

### Step 3: Map Changes to Domains

Using the mappings from kb-meta.json:

```
services/audio*.ts → domains/audio.md, systems/SYS-001-audio.md
services/location*.ts → domains/location.md, systems/SYS-003-location.md
__maestro__/** → patterns/testing-patterns.md, testing/TEST-001-e2e.md
```

### Step 4: Detect New Patterns

For added files:
- Check if they fit existing domain patterns
- Flag new domains for user decision
- Identify new services/hooks/components for index updates

---

## Output Format

Write analysis to stdout for user visibility:

```
KB Analysis Complete
====================
Mode: init | update
Tech Stack: {language}/{framework}
Package Manager: {manager}

Project Metrics:
- Total files: {count}
- Source files: {count}
- Test files: {count}
- Documentation files: {count}

Domains Detected ({count}):
{for each domain}
- {name} ({file_count} files)
  Primary: {main_service}
  Related: {related_files}
{end}

Patterns Detected ({count}):
{for each pattern}
- {name}: {description}
  Evidence: {files}
  Confidence: {high|medium|low}
{end}

Existing Documentation:
{for each doc}
- {path} ({lines} lines)
  Purpose: {inferred_purpose}
{end}

Changes (update mode only):
{for each change}
- {file}: {type} ({new_functions|modified|deleted})
  Affects: {skill}, {human_doc}
{end}
```

---

## kb-work.json Schema

Write this file for other agents to consume:

```json
{
  "sessionId": "uuid",
  "command": "init | update",
  "timestamp": "ISO-8601",
  "phase": "analysis",

  "analysis": {
    "techStack": {
      "language": "typescript",
      "framework": "react-native-expo",
      "packageManager": "npm",
      "testFramework": "jest"
    },
    "metrics": {
      "totalFiles": 1200,
      "sourceFiles": 450,
      "testFiles": 120,
      "docFiles": 25
    },
    "directories": {
      "source": ["app/", "services/", "components/"],
      "tests": ["__tests__/", "__maestro__/"],
      "config": ["config/"],
      "docs": ["docs/"]
    },
    "domains": [
      {
        "name": "audio",
        "files": ["services/audioService.ts", "providers/audioProvider.tsx"],
        "fileCount": 6,
        "description": "Audio playback and management"
      }
    ],
    "patterns": [
      {
        "name": "ServiceResponse",
        "files": ["services/serviceResponse.ts"],
        "occurrences": 45,
        "confidence": "high"
      }
    ],
    "existingDocs": [
      {
        "path": "docs/AUDIO_SYSTEM.md",
        "lines": 500,
        "purpose": "Audio system documentation",
        "suggestedMigration": "systems/SYS-001-audio.md"
      }
    ]
  },

  "assignments": {
    "skill-writer": [
      "domains/audio.md",
      "domains/location.md",
      "patterns/service-pattern.md"
    ],
    "doc-writer": [
      "systems/SYS-001-audio.md",
      "systems/SYS-002-location.md"
    ],
    "refactor": [
      "docs/AUDIO_SYSTEM.md → systems/SYS-001-audio.md"
    ]
  },

  "completed": {
    "skill-writer": [],
    "doc-writer": [],
    "refactor": []
  }
}
```

---

## Rules

1. **Read-only operations only** - Never modify source files
2. **Never create documentation** - Other agents do this
3. **Always write kb-work.json** - This is the handoff mechanism
4. **Use absolute paths** - For unambiguous file references
5. **Estimate conservatively** - Better to over-document than under-document
6. **Flag ambiguities** - If unsure about a domain, note it for user decision

---

## Error Handling

If unable to detect tech stack:
```
WARNING: Could not auto-detect tech stack.
Please specify manually or check that config files exist.
```

If project is very large (>5000 files):
```
NOTICE: Large project detected ({count} files).
Deep analysis may take several minutes.
Continue? [Y/n]
```

If no source files found:
```
ERROR: No source files detected.
Is this the correct project root?
Current directory: {cwd}
```
