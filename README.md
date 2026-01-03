# Claude Code Knowledge Base Generator

A comprehensive knowledge base generation system for [Claude Code](https://claude.ai/claude-code) that creates AI-optimized documentation using specialized agents.

## Features

- **Universal Tech Stack Support** - Auto-detects React, Python, Go, Rust, Java, and 15+ frameworks
- **Two-Layer Documentation** - AI-optimized skills + human-readable comprehensive docs
- **Specialized Agents** - Dedicated agents for analysis, skill writing, doc writing, and migration
- **Plan Mode Integration** - Preview all changes before applying
- **Parallel Execution** - Multiple agents work simultaneously for faster generation
- **Existing Docs Migration** - Automatically integrates your existing documentation
- **Incremental Updates** - Keep KB current with git-based change detection

## Requirements

- [Claude Code CLI](https://claude.ai/claude-code) installed and configured
- A codebase to document

## Installation

1. Clone or download this repository
2. Copy the folders to your project's `.claude/` directory:

```bash
# From your project root
cp -r path/to/claude-code-kb-generator/commands .claude/
cp -r path/to/claude-code-kb-generator/knowledge-base .claude/
```

Your project structure should look like:
```
your-project/
├── .claude/
│   ├── commands/
│   │   ├── init-kb.md
│   │   └── update-kb.md
│   └── knowledge-base/
│       ├── agents/
│       │   ├── kb-analyzer.md
│       │   ├── kb-skill-writer.md
│       │   ├── kb-doc-writer.md
│       │   └── kb-refactor.md
│       ├── config/
│       │   ├── tech-stacks.yaml
│       │   └── thresholds.yaml
│       └── templates/
│           └── universal/
│               ├── SKILL.md.tmpl
│               ├── domain.md.tmpl
│               ├── pattern.md.tmpl
│               ├── reference-index.md.tmpl
│               └── human-doc-system.md.tmpl
├── src/
└── ...
```

## Optional: Optimize for Your Tech Stack

After installation, you can optimize the knowledge base generator for your specific codebase. This produces more relevant documentation by understanding your project's architecture and conventions.

Run this prompt in Claude Code:

```
I just installed the kb-generator commands in .claude/. Please:

1. Analyze my codebase to detect my tech stack, architecture patterns, and existing documentation
2. Read the command files in .claude/commands/ and .claude/knowledge-base/agents/
3. Optimize each KB agent by:
   - Updating tech-stacks.yaml with my specific framework versions and patterns
   - Adjusting skill templates to match my project's domain terminology
   - Configuring documentation structure based on my existing docs organization
   - Setting complexity thresholds appropriate for my codebase size
4. Keep the agent structure, two-layer documentation approach, and output format unchanged

Show me what you'll change before applying.
```

## Usage

### Initialize Knowledge Base

Run the initialization command to create a complete KB:

```
/init-kb
```

The command will:
1. **Analyze** your codebase (tech stack, patterns, structure)
2. **Ask questions** about documentation preferences
3. **Show a plan** of what will be created
4. **Execute** with parallel agents after approval

### Update Knowledge Base

Keep your KB current after development:

```
/update-kb        # Analyze last 20 commits
/update-kb 50     # Analyze last 50 commits
/update-kb --dry-run  # Preview without applying
```

## Command Options

| Command | Description |
|---------|-------------|
| `/init-kb` | Full KB generation with plan mode |
| `/init-kb --minimal` | Core structure only |
| `/init-kb --dry-run` | Preview structure without creating |
| `/update-kb` | Update from last 20 commits |
| `/update-kb N` | Update from last N commits |
| `/update-kb --dry-run` | Preview updates without applying |

## Generated Structure

### AI Skills (`.claude/skills/`)

Concise, AI-optimized documentation (300-500 lines each):

```
.claude/skills/
├── SKILL.md              # Master navigation index
├── domains/              # Feature-specific knowledge
│   ├── audio.md
│   ├── auth.md
│   └── ...
├── patterns/             # Cross-cutting patterns
│   ├── service-pattern.md
│   └── provider-pattern.md
├── infrastructure/       # Tools & services
│   ├── supabase.md
│   └── sentry.md
├── references/           # Quick lookup tables
│   ├── services-index.md
│   └── components-index.md
└── onboarding/           # Getting started guides
    ├── quick-start.md
    └── architecture-overview.md
```

### Human Documentation (`docs/knowledge_base/`)

Comprehensive specifications (1000+ lines each):

```
docs/knowledge_base/
├── README.md             # Documentation index
├── systems/              # Core system specs
│   ├── SYS-001-audio.md
│   └── SYS-002-auth.md
├── features/             # Feature documentation
├── integrations/         # Third-party integrations
└── testing/              # Test documentation
```

## Specialized Agents

### KB-Analyzer

Discovers and analyzes your codebase:
- Tech stack detection (15+ frameworks)
- Directory structure mapping
- Pattern detection (ServiceResponse, Provider, etc.)
- Existing documentation discovery
- Complexity estimation

### KB-Skill-Writer

Creates AI-optimized skill files:
- Concise content (300-500 lines)
- YAML frontmatter with metadata
- Code examples from your codebase
- Cross-references between skills
- Reference indexes

### KB-Doc-Writer

Creates comprehensive human documentation:
- Full specifications (1000+ lines)
- Architecture diagrams (ASCII)
- Sequence diagrams for flows
- Configuration documentation
- Troubleshooting guides
- Changelog sections

### KB-Refactor

Migrates existing documentation:
- Analyzes existing docs structure
- Maps to new KB organization
- Creates deprecation notices
- Updates cross-references
- Preserves git history

## Supported Tech Stacks

The generator auto-detects and adapts to:

**Frontend:**
- React, React Native, Expo
- Next.js, Nuxt.js
- Vue, Angular, Svelte

**Backend:**
- Node.js, Express, NestJS
- Django, Flask, FastAPI
- Go (Gin, Echo, Fiber)
- Rust (Actix, Axum)
- Java (Spring Boot)

**Mobile:**
- React Native, Expo
- Flutter
- Native iOS/Android

See `knowledge-base/config/tech-stacks.yaml` for complete detection rules.

## Configuration

### Tech Stack Detection

Edit `knowledge-base/config/tech-stacks.yaml` to customize:
- Detection rules per framework
- Default directory patterns
- Template selection

### Thresholds

Edit `knowledge-base/config/thresholds.yaml` to customize:
- Project size thresholds
- Analysis depth settings
- Output limits
- When to ask for confirmation

### Templates

Customize templates in `knowledge-base/templates/universal/`:
- `SKILL.md.tmpl` - Master index template
- `domain.md.tmpl` - Domain skill template
- `pattern.md.tmpl` - Pattern skill template
- `human-doc-system.md.tmpl` - Human doc template

## How It Works

### Initialization Flow

```
/init-kb
    │
    ▼
┌─────────────────┐
│  kb-analyzer    │ ← Detect tech stack, patterns, existing docs
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  User Questions │ ← Where to put docs? Integrate existing?
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Plan Preview   │ ← Show what will be created
└────────┬────────┘
         │ (approved)
         ▼
┌─────────────────────────────────────────────┐
│            Parallel Agent Execution          │
│  ┌──────────────┐  ┌──────────────┐         │
│  │ skill-writer │  │  doc-writer  │         │
│  │  (domains)   │  │  (systems)   │         │
│  └──────────────┘  └──────────────┘         │
│  ┌──────────────┐  ┌──────────────┐         │
│  │ skill-writer │  │  kb-refactor │         │
│  │  (patterns)  │  │  (migration) │         │
│  └──────────────┘  └──────────────┘         │
└─────────────────────────────────────────────┘
         │
         ▼
┌─────────────────┐
│ Update Metadata │ ← kb-meta.json
└─────────────────┘
```

### Update Flow

```
/update-kb
    │
    ▼
┌─────────────────┐
│ Load kb-meta    │ ← Tech stack, mappings, last commit
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Analyze Changes │ ← git diff since last update
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Update Proposal │ ← Show affected docs
└────────┬────────┘
         │ (approved)
         ▼
┌─────────────────┐
│ Apply Updates   │ ← Parallel skill/doc updates
└─────────────────┘
```

## Metadata Tracking

The system maintains `kb-meta.json` to track:
- Project tech stack and framework
- File-to-documentation mappings
- Last analyzed commit
- Update history
- Integrated existing docs

## Best Practices

1. **Run `/init-kb` once** when setting up a new project
2. **Run `/update-kb` regularly** after significant development
3. **Review generated content** before committing
4. **Customize templates** to match your documentation style
5. **Add `<!-- MANUAL -->` markers** to preserve manual edits during updates

## Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

### Adding New Tech Stack Support

1. Add detection rules to `config/tech-stacks.yaml`
2. (Optional) Create stack-specific templates in `templates/stacks/`
3. Test with a project using that stack

## License

MIT License - see [LICENSE](LICENSE) file.

## References

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Agent Skills Pattern](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- [Multi-Agent Architecture](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns)
