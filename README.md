# Claude Config Manager (CCM)

A CLI tool for managing Claude Code configuration profiles, enabling quick switching between different API endpoints and keys.

## Features

- 🔄 **Quick Profile Switching**: Switch between configurations in seconds
- 📦 **Profile Management**: Add, update, delete, and rename profiles
- 🔒 **Secure Storage**: API keys stored with restricted file permissions
- 💾 **Import/Export**: Backup and share profiles (with optional key masking)
- 🎨 **Beautiful CLI**: Colored output with clear status indicators
- 🛡️ **Safe Operations**: Atomic file writes with automatic backups

## Installation

### Global Installation (Recommended)

```bash
npm install -g @haiyangj/ccs
```

### Local Installation

```bash
git clone https://github.com/haiyangj/ccs.git
cd ccs
npm install
npm link
```

## Quick Start

1. **Initialize profile management**
   ```bash
   ccm init
   ```

2. **Add a new profile**
   ```bash
   ccm add work https://work-api.company.com sk-ant-work-key
   ```

3. **Switch to a profile**
   ```bash
   ccm use work
   ```

4. **List all profiles**
   ```bash
   ccm list
   ```

## Commands

### `ccm init`
Initialize profile management. If you have an existing `settings.json`, it will be imported as the "default" profile.

```bash
ccm init
```

### `ccm list` (alias: `ls`)
Display all configured profiles with status indicators.

```bash
ccm list
```

**Output:**
```
Available profiles:

  ✓ default      https://api.anthropic.com        ****xxxx (active)
    work         https://work-proxy.company.com   ****yyyy
    staging      https://staging-api.example.com  ****zzzz

Use 'ccm use <name>' to switch profiles
```

### `ccm show` (alias: `current`)
Display detailed information about the currently active profile.

```bash
ccm show
```

**Output:**
```
Current profile: work

  API URL:    https://work-proxy.company.com
  API Key:    ****yyyy
  Created:    2026-01-16 11:00 AM
  Last used:  2026-01-16 02:30 PM
```

### `ccm add [name] [url] [key]`
Add a new profile. Can be used with arguments or in interactive mode.

**With Arguments:**
```bash
ccm add staging https://staging-api.example.com sk-ant-staging-key
```

**Interactive Mode:**
```bash
ccm add
# Prompts for:
# - Profile name
# - API URL
# - API Key (hidden input)
```

**Options:**
- `--no-switch`: Don't prompt to switch to the new profile

### `ccm use <name>`
Switch to a different profile. Updates both `profiles.json` and `settings.json`.

```bash
ccm use work
```

**Features:**
- Suggests similar profile names if not found
- Updates last used timestamp
- Applies changes immediately to settings.json

### `ccm set [name] [url] [key]`
Update an existing profile's API URL and key.

**With Arguments:**
```bash
ccm set work https://new-work-api.com sk-ant-new-work-key
```

**Interactive Mode:**
```bash
ccm set
```

**Notes:**
- If updating the currently active profile, changes are immediately applied to settings.json
- Preserves profile creation timestamp

### `ccm delete <name>` (alias: `rm`)
Delete a profile with confirmation prompt.

```bash
ccm delete staging
```

**Options:**
- `-f, --force`: Skip confirmation prompt

**Safeguards:**
- Cannot delete the currently active profile
- Requires switching to another profile first

### `ccm rename <old-name> <new-name>` (alias: `mv`)
Rename an existing profile.

```bash
ccm rename staging stage
```

**Notes:**
- Updates current profile reference if renaming the active profile
- Validates new name format

### `ccm export [file]`
Export profiles to a JSON file for backup or sharing.

```bash
# Default location
ccm export

# Custom location
ccm export ~/backups/claude-profiles.json
```

**Options:**
- `--no-keys`: Export without API keys (for sharing profile structure)

**Output:**
```
✓ Exported 3 profiles to ./claude-profiles-backup.json
```

### `ccm import <file>`
Import profiles from a previously exported file.

```bash
ccm import ~/backups/claude-profiles.json
```

**Options:**
- `--merge`: Merge with existing profiles (overwrite conflicts)

**Behavior:**
- Skips profiles with conflicting names by default
- Validates profile structure before importing
- Preserves current active profile

## Configuration Files

### `.claude/profiles.json`
Stores all profile configurations:

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
    }
  }
}
```

### `.claude/settings.json`
Claude Code's active configuration (updated by `ccm use`):

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

## Directory Resolution

CCM looks for the `.claude` directory in the following order:

1. **Environment Variable**: `$CLAUDE_CONFIG_DIR`
2. **Current Directory**: `./claude`
3. **Git Root**: `$(git rev-parse --show-toplevel)/.claude`
4. **Home Directory**: `~/.claude`

## Security

- Profile files are created with `0600` permissions (owner read/write only)
- API keys are masked in all output (shows last 4 characters only)
- Atomic file writes with automatic backups
- No logging of sensitive information

## Examples

### Complete Workflow

```bash
# Initialize
ccm init

# Add profiles for different environments
ccm add personal https://api.anthropic.com sk-ant-personal-key
ccm add work https://work-api.company.com sk-ant-work-key
ccm add staging https://staging-api.example.com sk-ant-staging-key

# List all profiles
ccm list

# Switch between profiles
ccm use work
ccm use staging
ccm use personal

# Show current profile
ccm show

# Update a profile
ccm set work https://new-work-api.com sk-ant-new-key

# Rename a profile
ccm rename staging stage

# Export for backup
ccm export ~/backups/claude-profiles.json

# Delete old profile
ccm delete stage

# Import on another machine
ccm import ~/backups/claude-profiles.json
```

### Interactive Mode

```bash
# Add profile interactively
$ ccm add
? Profile name: demo
? API URL: https://demo-api.example.com
? API Key: ************
✓ Profile 'demo' added successfully
  API URL: https://demo-api.example.com
  API Key: ****xxxx

? Switch to this profile now? (y/N)
```

## Troubleshooting

### Profile not found
```bash
✗ Profile 'wrk' does not exist
ℹ Did you mean: work?
```

### Cannot delete active profile
```bash
✗ Failed to delete profile: Cannot delete active profile 'work'. Switch to another profile first.
```

### Import conflicts
```bash
⚠ 1 profile skipped (name conflict: 'work')
ℹ Use --merge flag to overwrite existing profiles
```

## Development

```bash
# Install dependencies
npm install

# Link for local testing
npm link

# Run directly
node bin/ccm.js list
```

## Contributing

Contributions are welcome! Please ensure:
- Code follows existing patterns
- Security considerations are maintained
- Error handling is comprehensive

## License

MIT

## Author

Built for Claude Code users who frequently switch between configurations.

---

**Questions or Issues?** Please open an issue on GitHub.
