# Quotefix Skill

A Claude Code skill that automatically detects shell environments (PowerShell, Bash, Zsh, Fish, CMD) and adjusts command syntax for quoting, escape symbols, German language characters, and file permissions.

## Features

- **Shell Detection**: Automatically detects Bash, Zsh, Fish, PowerShell, CMD, and other shells
- **Platform Detection**: Identifies Linux, macOS, and Windows environments
- **Quote Handling**: Applies correct quoting rules for each shell type
- **Escape Characters**: Uses the right escape sequences (backslash, backtick, caret)
- **German Language Support**: Properly handles Umlauts (ä, ö, ü, ß) and special characters
- **Permission Management**: Cross-platform file permission handling (chmod, icacls)
- **UTF-8 Encoding**: Ensures correct encoding for international characters

## Installation

This skill is already installed in your project at:
```
.claude/skills/quotefix/
```

Claude Code will automatically detect and use this skill when relevant.

## When This Skill Activates

The skill will be used automatically when:
- Executing shell commands that differ between shells
- Working with file paths containing spaces or special characters
- Handling German language characters in filenames or content
- Setting file permissions on different platforms
- Writing cross-platform scripts

## File Structure

```
quotefix/
├── SKILL.md      # Main skill definition with detection logic
├── examples.md   # Practical examples and use cases
└── README.md     # This file
```

## Quick Start Examples

### Detect Current Shell (Bash)

```bash
# Method 1: Check environment variable
echo $SHELL

# Method 2: Check running process
ps -p $$ -o comm=

# Method 3: Check version variables
if [ -n "$BASH_VERSION" ]; then echo "bash"; fi
```

### Detect Current Shell (PowerShell)

```powershell
# Check PowerShell version
$PSVersionTable.PSVersion

# Check if running in PowerShell
if ($PSVersionTable) { Write-Host "PowerShell" }
```

### Create File with German Characters

**Bash:**
```bash
export LANG=de_DE.UTF-8
filename="Größenübersicht.txt"
touch "$filename"
echo "Größe: 100 cm" > "$filename"
```

**PowerShell:**
```powershell
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$filename = "Größenübersicht.txt"
New-Item -Path $filename -ItemType File
"Größe: 100 cm" | Out-File -FilePath $filename -Encoding UTF8
```

### Set File Permissions

**Unix/Linux/macOS (Bash):**
```bash
chmod 755 script.sh
chmod u+x script.sh
ls -la script.sh
```

**Windows (PowerShell):**
```powershell
$acl = Get-Acl script.ps1
$permission = "$env:USERNAME","FullControl","Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule $permission
$acl.SetAccessRule($accessRule)
Set-Acl script.ps1 $acl
```

**Windows (CMD with icacls):**
```cmd
icacls script.bat /grant %USERNAME%:F
```

## Quoting Quick Reference

| Shell | Variable Expansion | Literal String | Escape Char |
|-------|-------------------|----------------|-------------|
| Bash/Zsh | `"$var"` | `'literal'` | `\` |
| PowerShell | `"$var"` | `'literal'` | `` ` `` |
| CMD | `%var%` | N/A | `^` |
| Fish | `"$var"` | `'literal'` | N/A (use quotes) |

## German Character Encoding

| Shell | Command |
|-------|---------|
| Bash/Zsh | `export LANG=de_DE.UTF-8` |
| PowerShell | `[Console]::OutputEncoding = [System.Text.Encoding]::UTF8` |
| CMD | `chcp 65001` |

## Platform-Specific Path Separators

- **Unix/Linux/macOS**: `/` (forward slash)
- **Windows**: `\` (backslash) - but PowerShell accepts both `/` and `\`

## Common Pitfalls

1. **Not quoting variables**: Always use `"$var"` in Bash to prevent word splitting
2. **Wrong escape character**: Bash uses `\`, PowerShell uses `` ` ``, CMD uses `^`
3. **Missing encoding setup**: Set UTF-8 before working with German characters
4. **Platform assumptions**: Always detect the platform before using platform-specific commands
5. **Path separators**: Use the correct separator for the target platform

## Testing the Skill

Create a test script to verify the skill works:

```bash
# test_skill.sh
#!/usr/bin/env bash

echo "Testing Quotefix..."

# Test 1: Shell detection
if [ -n "$BASH_VERSION" ]; then
    echo "✓ Detected Bash"
fi

# Test 2: Create file with Umlauts
filename="Test_Größe.txt"
touch "$filename"
echo "✓ Created file with German characters: $filename"

# Test 3: Set permissions
chmod 644 "$filename"
echo "✓ Set permissions"

# Cleanup
rm "$filename"
echo "✓ Cleanup complete"
```

## See Also

- [SKILL.md](./SKILL.md) - Complete skill definition
- [examples.md](./examples.md) - Detailed examples
- [Claude Code Documentation](https://github.com/anthropics/claude-code)

## License

This skill is part of the havok-training project.

## Contributing

To improve this skill:
1. Edit `SKILL.md` for skill behavior
2. Add examples to `examples.md`
3. Update this README with any new features
4. Test on multiple platforms and shells

## Support

For issues or questions:
- Check the [examples.md](./examples.md) file
- Review the [SKILL.md](./SKILL.md) documentation
- Refer to Claude Code documentation

---

**Created for**: havok-training project
**Branch**: claude/detect-shell-environment-G4vQY
**Purpose**: Cross-platform shell command compatibility
