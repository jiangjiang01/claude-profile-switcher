# Claude Config Manager (CCM) - Design Specification

## Overview
A Node.js CLI tool for managing Claude Code configuration profiles, enabling quick switching between different API endpoints and keys stored in `.claude/settings.json`.

---

## System Architecture

### Component Structure
```
ccm/
├── bin/
│   └── ccm.js              # CLI entry point
├── src/
│   ├── commands/
│   │   ├── list.js         # List all profiles
│   │   ├── use.js          # Switch to a profile
│   │   ├── add.js          # Add new profile
│   │   ├── set.js          # Update existing profile
│   │   ├── delete.js       # Delete a profile
│   │   ├── show.js         # Show current active profile
│   │   └── init.js         # Initialize profile storage
│   ├── core/
│   │   ├── config.js       # Config file operations
│   │   ├── profiles.js     # Profile management
│   │   └── validator.js    # Input validation
│   └── utils/
│       ├── logger.js       # Colored console output
│       └── paths.js        # Path resolution
├── package.json
└── README.md
```

---

## Data Model

### Profile Storage Format
Location: `.claude/profiles.json`

```json
{
  "version": "1.0.0",
  "currentProfile": "default",
  "profiles": {
    "default": {
      "name": "default",
      "apiUrl": "https://api.anthropic.com",
      "apiKey": "sk-ant-xxxxx",
      "createdAt": "2026-01-16T10:00:00.000Z",
      "lastUsed": "2026-01-16T10:00:00.000Z"
    },
    "work": {
      "name": "work",
      "apiUrl": "https://work-proxy.company.com",
      "apiKey": "sk-ant-work-xxxxx",
      "createdAt": "2026-01-16T11:00:00.000Z",
      "lastUsed": null
    }
  }
}
```

### Settings File Format
Location: `.claude/settings.json`

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "your-api-key",
    "ANTHROPIC_BASE_URL": "https://api.anthropic.com"
  },
  "enabledPlugins": {
    "document-skills@anthropic-agent-skills": true,
    "code-review@claude-plugins-official": true,
    "superpowers@superpowers-marketplace": true
  },
  "apiUrl": "",
  "apiKey": ""
}
```

---

## CLI Commands Specification

### 1. `ccm list` (alias: `ls`)
**Purpose**: Display all configured profiles with status indicators

**Output**:
```
Available profiles:

  ✓ default      https://api.anthropic.com        (active)
    work         https://work-proxy.company.com
    staging      https://staging-api.example.com

Use 'ccm use <name>' to switch profiles
```

**Features**:
- ✓ checkmark for currently active profile
- Masked API keys (show last 4 chars only)
- Color coding: green for active, white for inactive
- Sort by last used, then alphabetically

---

### 2. `ccm use <name>`
**Purpose**: Switch to a different profile

**Arguments**:
- `<name>`: Profile name (required)

**Behavior**:
1. Validate profile exists
2. Read current `.claude/settings.json`
3. Update `apiUrl` and `apiKey` from selected profile
4. Write back to `.claude/settings.json`
5. Update `lastUsed` timestamp in profiles.json
6. Update `currentProfile` in profiles.json

**Output**:
```
✓ Switched to profile 'work'
  API URL: https://work-proxy.company.com
  API Key: ****xxxxx
```

**Error Handling**:
- Profile not found → suggest similar names
- Settings file missing → offer to create
- Permission denied → clear error message

---

### 3. `ccm add <name> <url> <key>`
**Purpose**: Add a new profile

**Arguments**:
- `<name>`: Profile name (required, alphanumeric + dash/underscore)
- `<url>`: API endpoint URL (required, must be valid HTTPS URL)
- `<key>`: API key (required)

**Validation**:
- Name: 1-50 chars, alphanumeric, dash, underscore only
- URL: Valid HTTPS URL format
- Key: Minimum 10 chars

**Behavior**:
1. Validate all inputs
2. Check for duplicate profile name
3. Add to profiles.json
4. Optionally switch to new profile (prompt user)

**Output**:
```
✓ Profile 'staging' added successfully
  API URL: https://staging-api.example.com
  API Key: ****xxxxx

Switch to this profile now? (y/N):
```

**Interactive Mode**:
```bash
ccm add
# Prompts for:
# Profile name: _
# API URL: _
# API Key: _ (hidden input)
```

---

### 4. `ccm set <name> <url> <key>`
**Purpose**: Update an existing profile

**Arguments**: Same as `add`

**Behavior**:
1. Validate profile exists
2. Validate new inputs
3. Update profile in profiles.json
4. If profile is currently active, update settings.json
5. Preserve createdAt timestamp

**Output**:
```
✓ Profile 'work' updated successfully
  API URL: https://work-proxy.company.com (updated)
  API Key: ****xxxxx (updated)

⚠ This profile is currently active. Changes applied to settings.json
```

---

### 5. `ccm delete <name>` (alias: `rm`)
**Purpose**: Remove a profile

**Arguments**:
- `<name>`: Profile name (required)

**Safeguards**:
- Cannot delete currently active profile (must switch first)
- Confirmation prompt (bypass with `--force` or `-f`)

**Output**:
```
⚠ Delete profile 'staging'? This cannot be undone. (y/N): y
✓ Profile 'staging' deleted
```

---

### 6. `ccm show` (alias: `current`)
**Purpose**: Display current active profile details

**Output**:
```
Current profile: work

  API URL: https://work-proxy.company.com
  API Key: ****xxxxx
  Created: 2026-01-16 11:00 AM
  Last used: 2026-01-16 02:30 PM
```

---

### 7. `ccm init`
**Purpose**: Initialize profile management (first-time setup)

**Behavior**:
1. Check if profiles.json exists
2. If settings.json exists, import as "default" profile
3. Create profiles.json with initial structure
4. Set up default profile

**Output**:
```
Initializing Claude Config Manager...

✓ Found existing settings.json
✓ Created profiles.json
✓ Imported current config as 'default' profile

You can now use 'ccm add' to create additional profiles.
```

---

### 8. `ccm export [file]`
**Purpose**: Export profiles to a file (for backup/sharing)

**Arguments**:
- `[file]`: Output file path (default: `./claude-profiles-backup.json`)

**Options**:
- `--no-keys`: Export without API keys (for sharing)

**Output**:
```
✓ Exported 3 profiles to claude-profiles-backup.json
```

---

### 9. `ccm import <file>`
**Purpose**: Import profiles from a file

**Arguments**:
- `<file>`: Input file path (required)

**Options**:
- `--merge`: Merge with existing profiles (default: replace)

**Behavior**:
1. Validate file format
2. Check for name conflicts
3. Import profiles
4. Preserve current active profile

**Output**:
```
Importing profiles from backup.json...

  ✓ 2 profiles imported
  ⚠ 1 profile skipped (name conflict: 'work')

Use 'ccm list' to see all profiles.
```

---

### 10. `ccm rename <old-name> <new-name>`
**Purpose**: Rename an existing profile

**Arguments**:
- `<old-name>`: Current profile name (required)
- `<new-name>`: New profile name (required)

**Output**:
```
✓ Profile renamed: 'staging' → 'stage'
```

---

## Technical Implementation Details

### Dependencies
```json
{
  "dependencies": {
    "commander": "^11.1.0",      // CLI framework
    "chalk": "^5.3.0",            // Terminal colors
    "inquirer": "^9.2.12",        // Interactive prompts
    "ora": "^8.0.1",              // Spinners
    "conf": "^12.0.0"             // Config storage (alternative to custom JSON)
  }
}
```

### Path Resolution Strategy
```javascript
// Priority order for finding .claude directory:
1. Current working directory: process.cwd() + '/.claude'
2. Git root directory: git rev-parse --show-toplevel + '/.claude'
3. Home directory: os.homedir() + '/.claude'
4. Environment variable: process.env.CLAUDE_CONFIG_DIR
```

### Error Handling Principles
- **Graceful degradation**: Don't crash, show helpful messages
- **Validation first**: Check inputs before file operations
- **Atomic operations**: Use temp files for updates, rename on success
- **Backup on modify**: Keep `.bak` file when updating settings.json

### Security Considerations
1. **File Permissions**: Set profiles.json to 0600 (owner read/write only)
2. **API Key Display**: Always mask except last 4 characters
3. **Input Sanitization**: Validate all user inputs
4. **No Logging**: Never log API keys, even to debug logs

---

## User Experience Enhancements

### Auto-completion Support
Provide shell completion scripts:
```bash
ccm completion bash > /etc/bash_completion.d/ccm
ccm completion zsh > ~/.zsh/completion/_ccm
```

### Configuration Validation
On every operation, validate settings.json structure:
- Required fields present
- API URL is reachable (optional check with --validate)
- API key format correct

### Color Scheme
- **Green**: Success messages, active profile
- **Yellow**: Warnings, prompts
- **Red**: Errors
- **Blue**: Info, headers
- **Gray**: Secondary text, masked keys

### Interactive Mode Triggers
Commands support both argument-based and interactive modes:
```bash
ccm add                    # Interactive prompts
ccm add name url key       # Direct arguments
```

---

## Migration Strategy

### First-time Users
1. Run `ccm init` to create profiles.json
2. Current settings.json becomes "default" profile
3. Ready to add more profiles

### Existing Multi-config Users
1. Run `ccm import` with existing config files
2. Assign meaningful names during import
3. Use `ccm use` to switch between them

---

## Testing Strategy

### Unit Tests
- Config file read/write operations
- Profile validation logic
- Path resolution
- Input sanitization

### Integration Tests
- Full command workflows (add → use → delete)
- File permission handling
- Concurrent access (file locking)

### Manual Testing Checklist
- [ ] Create profile with special characters in URL
- [ ] Switch profiles and verify settings.json updates
- [ ] Delete active profile (should fail)
- [ ] Import with name conflicts
- [ ] Run without .claude directory (should guide setup)

---

## Future Enhancements (v2.0)

1. **Environment Variables**: Support for env var substitution in profiles
2. **Profile Groups**: Organize profiles by project/environment
3. **Cloud Sync**: Optional encrypted cloud backup of profiles
4. **API Key Encryption**: Encrypt keys at rest in profiles.json
5. **Health Checks**: Validate API connectivity before switching
6. **Profile Templates**: Quick setup for common configurations
7. **Diff Command**: Compare two profiles
8. **Audit Log**: Track profile switches with timestamps

---

## Installation & Distribution

### NPM Package
```bash
npm install -g @haiyangj/ccs
```

### Manual Installation
```bash
git clone https://github.com/haiyangj/ccs.git
cd ccs
npm install
npm link
```

### Binary Distribution
Use `pkg` to create standalone binaries for:
- macOS (ARM64, x64)
- Linux (x64)
- Windows (x64)

---

## Documentation Requirements

1. **README.md**: Quick start guide, command reference
2. **ARCHITECTURE.md**: This design document
3. **API.md**: Programmatic usage (if used as library)
4. **CHANGELOG.md**: Version history
5. **CONTRIBUTING.md**: Development setup, testing

---

## Success Metrics

- **Time to switch**: < 1 second for profile switching
- **First-time setup**: < 2 minutes from install to first profile switch
- **Error recovery**: Clear actionable messages for all error states
- **Zero data loss**: Atomic operations, backup files maintained

---

## Command Summary Table

| Command | Aliases | Arguments | Description |
|---------|---------|-----------|-------------|
| `init` | - | - | Initialize profile management |
| `list` | `ls` | - | Show all profiles |
| `show` | `current` | - | Show active profile details |
| `use` | - | `<name>` | Switch to profile |
| `add` | - | `<name> <url> <key>` | Add new profile |
| `set` | - | `<name> <url> <key>` | Update profile |
| `delete` | `rm` | `<name>` | Delete profile |
| `rename` | `mv` | `<old> <new>` | Rename profile |
| `export` | - | `[file]` | Export profiles |
| `import` | - | `<file>` | Import profiles |

---

## Example Workflow

```bash
# First time setup
$ ccm init
✓ Initialized with 'default' profile

# Add work profile
$ ccm add work https://work.api.com sk-ant-work-key
✓ Profile 'work' added

# List profiles
$ ccm list
  ✓ default    https://api.anthropic.com         (active)
    work       https://work.api.com

# Switch to work
$ ccm use work
✓ Switched to profile 'work'

# Show current
$ ccm show
Current profile: work
  API URL: https://work.api.com
  API Key: ****key

# Back to default
$ ccm use default
✓ Switched to profile 'default'

# Export for backup
$ ccm export ~/backups/claude-profiles.json
✓ Exported 2 profiles
```

---

## End of Specification

**Document Version**: 1.0.0
**Last Updated**: 2026-01-16
**Status**: Ready for Implementation
