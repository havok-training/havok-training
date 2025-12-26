---
name: shell-environment-detector
description: Detects the current shell environment (PowerShell, Bash, Zsh, Fish, CMD, etc.) and automatically adjusts command syntax for quoting, escape symbols, German language characters (Umlauts), and file permissions. Use when executing shell commands to ensure compatibility across different operating systems and shell environments.
allowed-tools:
  - Bash
  - Read
  - Grep
  - Write
  - Edit
---

# Shell Environment Detector

This skill ensures commands are compatible with the user's shell environment by detecting the active shell and adjusting syntax for quoting, escaping, special characters (including German Umlauts), and permissions.

## When to Use This Skill

- Executing shell commands that may differ between PowerShell, Bash, CMD, and other shells
- Working with file paths containing spaces or special characters
- Handling German language characters (ä, ö, ü, ß) in filenames or content
- Setting file permissions on Unix/Linux vs Windows systems
- Writing cross-platform scripts
- Dealing with environment variables and PATH configuration

## Shell Detection Strategy

### 1. Detect the Active Shell

**On Unix/Linux/macOS:**
```bash
# Method 1: Check SHELL environment variable
echo $SHELL  # Shows the user's login shell (e.g., /bin/bash, /usr/bin/zsh)

# Method 2: Check current process
ps -p $$ -o comm=  # Shows the actual running shell

# Method 3: Check shell-specific variables
if [ -n "$BASH_VERSION" ]; then echo "bash"; fi
if [ -n "$ZSH_VERSION" ]; then echo "zsh"; fi
```

**On Windows:**
```powershell
# Method 1: Check environment variable
$env:SHELL  # May not be set on Windows

# Method 2: Check PowerShell version
$PSVersionTable.PSVersion  # PowerShell specific

# Method 3: Check if running in CMD
echo %COMSPEC%  # CMD specific
```

### 2. Platform Detection

```bash
# Detect OS
uname -s  # Linux, Darwin (macOS), MINGW64_NT (Git Bash on Windows)

# Or use Python for cross-platform detection
python -c "import platform; print(platform.system())"  # Linux, Windows, Darwin
```

## Syntax Adjustments by Shell

### Bash/Zsh (Unix/Linux/macOS)

**Quoting:**
- Single quotes: Literal strings, no variable expansion: `'$HOME'` → `$HOME`
- Double quotes: Allow variable expansion: `"$HOME"` → `/home/user`
- Escape special chars: `\"`, `\'`, `\\`, `\$`

**German Characters:**
- Ensure UTF-8 encoding: `export LANG=de_DE.UTF-8`
- Filenames with Umlauts: `"Übersicht.txt"` (always quote)

**Permissions:**
```bash
# Read permissions
ls -la file.txt

# Set permissions
chmod 755 script.sh      # rwxr-xr-x
chmod u+x script.sh      # Add execute for user
chmod go-w file.txt      # Remove write for group and others

# Change ownership
chown user:group file.txt
```

**Examples:**
```bash
# File with spaces and Umlauts
filename="Meine Übersicht.txt"
touch "$filename"
cat "$filename"

# Escape special characters
echo "Price: \$100"
echo 'Price: $100'  # Literal

# Command substitution
current_date=$(date +%Y-%m-%d)
```

### PowerShell (Windows)

**Quoting:**
- Single quotes: Literal strings: `'$env:HOME'` → `$env:HOME`
- Double quotes: Variable expansion: `"$env:HOME"` → `C:\Users\username`
- Escape character: Backtick `` ` ``
- Special chars: `` `n`` (newline), `` `t`` (tab), `` `$`` (literal $)

**German Characters:**
- PowerShell uses UTF-16LE by default
- Set encoding: `[Console]::OutputEncoding = [System.Text.Encoding]::UTF8`
- For files: `-Encoding UTF8` parameter

**Permissions (ACLs):**
```powershell
# Read permissions
Get-Acl file.txt | Format-List

# Set permissions
$acl = Get-Acl file.txt
$permission = "DOMAIN\User","FullControl","Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule $permission
$acl.SetAccessRule($accessRule)
Set-Acl file.txt $acl

# Simple attribute changes
Set-ItemProperty file.txt -Name IsReadOnly -Value $true
```

**Examples:**
```powershell
# File with spaces and Umlauts
$filename = "Meine Übersicht.txt"
New-Item -Path $filename -ItemType File
Get-Content $filename

# Escape special characters
Write-Host "Price: `$100"
Write-Host 'Price: $100'  # Literal

# Command substitution
$currentDate = Get-Date -Format "yyyy-MM-dd"

# Paths with spaces
$path = "C:\Program Files\My App\data.txt"
Test-Path "$path"  # Always quote paths
```

### CMD (Windows Command Prompt)

**Quoting:**
- Double quotes for paths with spaces: `"C:\Program Files\app.exe"`
- Escape character: Caret `^`
- Special chars: `^&`, `^|`, `^<`, `^>`, `^^`

**German Characters:**
- Use `chcp 65001` to set UTF-8 code page
- Default is often Windows-1252 or CP850 (German)

**Examples:**
```cmd
REM File with spaces
set filename="Meine Datei.txt"
type %filename%

REM Escape special characters
echo Price: ^$100
echo 5 ^& 6

REM Use variables
set "PATH_TO_APP=C:\Program Files\MyApp"
"%PATH_TO_APP%\app.exe"
```

### Fish Shell

**Quoting:**
- Single quotes: Literal
- Double quotes: Variable expansion
- No escape character in strings (use quotes)

**Examples:**
```fish
# File with spaces
set filename "Meine Übersicht.txt"
touch $filename
cat $filename

# Variables
set current_date (date +%Y-%m-%d)
```

## Cross-Platform Command Translation

### Common Operations

| Operation | Bash/Zsh | PowerShell | CMD |
|-----------|-----------|------------|-----|
| List files | `ls -la` | `Get-ChildItem` or `ls` | `dir` |
| Copy file | `cp src dst` | `Copy-Item src dst` | `copy src dst` |
| Move file | `mv src dst` | `Move-Item src dst` | `move src dst` |
| Delete file | `rm file` | `Remove-Item file` | `del file` |
| Echo | `echo "text"` | `Write-Host "text"` | `echo text` |
| Environment var | `$HOME` | `$env:HOME` | `%USERPROFILE%` |
| Path separator | `/` | `\` or `/` | `\` |
| Line continuation | `\` | `` ` `` | `^` |

### File Permissions

| Platform | Command | Example |
|----------|---------|---------|
| Unix/Linux | `chmod` | `chmod 755 script.sh` |
| macOS | `chmod` | `chmod +x app` |
| Windows (PowerShell) | `Set-Acl` or `icacls` | `icacls file.txt /grant User:F` |
| Windows (CMD) | `icacls` | `icacls file.txt /grant User:F` |

## Implementation Guidelines

### Before Executing Commands

1. **Detect the shell:**
   ```bash
   # In Bash
   CURRENT_SHELL=$(basename "$SHELL")

   # Or check specific shell
   if [ -n "$BASH_VERSION" ]; then
       SHELL_TYPE="bash"
   elif [ -n "$ZSH_VERSION" ]; then
       SHELL_TYPE="zsh"
   fi
   ```

2. **Detect the platform:**
   ```bash
   case "$(uname -s)" in
       Linux*)     PLATFORM=linux;;
       Darwin*)    PLATFORM=mac;;
       MINGW*)     PLATFORM=windows;;
       *)          PLATFORM=unknown;;
   esac
   ```

3. **Apply appropriate syntax:**
   - Use proper quoting for the detected shell
   - Escape special characters correctly
   - Use correct path separators
   - Apply proper permission commands

### Handling German Language Specifics

**File Names with Umlauts:**
```bash
# Bash - always quote
filename="Größe_Übersicht.txt"
touch "$filename"

# PowerShell
$filename = "Größe_Übersicht.txt"
New-Item -Path $filename -ItemType File

# Ensure UTF-8
export LC_ALL=de_DE.UTF-8  # Bash
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8  # PowerShell
```

**String Content:**
```bash
# Bash
echo "Größe: 10 cm"
printf "Preis: %.2f €\n" 19.99

# PowerShell
Write-Host "Größe: 10 cm"
"{0:C2}" -f 19.99  # Currency formatting
```

### Permission Handling Template

```bash
# Detect platform and set permissions
set_permissions() {
    local file=$1
    local mode=$2

    if command -v chmod &> /dev/null; then
        # Unix/Linux/macOS
        chmod "$mode" "$file"
        echo "Set permissions to $mode on $file"
    elif command -v icacls &> /dev/null; then
        # Windows
        case "$mode" in
            755|rwxr-xr-x)
                icacls "$file" /grant:r "%USERNAME%:F" /grant:r "Users:RX"
                ;;
            644|rw-r--r--)
                icacls "$file" /grant:r "%USERNAME%:W" /grant:r "Users:R"
                ;;
        esac
        echo "Set Windows ACL on $file"
    else
        echo "Unable to set permissions - unknown platform"
        return 1
    fi
}

# Usage
set_permissions "script.sh" "755"
```

## Complete Example: Cross-Platform Script

```bash
#!/usr/bin/env bash

# Detect shell and platform
detect_environment() {
    # Detect shell
    if [ -n "$BASH_VERSION" ]; then
        SHELL_TYPE="bash"
    elif [ -n "$ZSH_VERSION" ]; then
        SHELL_TYPE="zsh"
    elif [ -n "$FISH_VERSION" ]; then
        SHELL_TYPE="fish"
    else
        SHELL_TYPE="unknown"
    fi

    # Detect platform
    case "$(uname -s 2>/dev/null || echo Windows)" in
        Linux*)     PLATFORM="linux";;
        Darwin*)    PLATFORM="mac";;
        MINGW*|CYGWIN*|MSYS*)  PLATFORM="windows";;
        Windows*)   PLATFORM="windows";;
        *)          PLATFORM="unknown";;
    esac

    echo "Detected: $SHELL_TYPE on $PLATFORM"
}

# Set encoding for German characters
set_encoding() {
    if [ "$PLATFORM" = "windows" ]; then
        # Windows UTF-8
        chcp 65001 2>/dev/null || true
    else
        # Unix/Linux/Mac UTF-8
        export LANG=de_DE.UTF-8
        export LC_ALL=de_DE.UTF-8
    fi
}

# Create file with proper quoting
create_file() {
    local filename="Projekt_Übersicht.txt"

    if [ "$SHELL_TYPE" = "bash" ] || [ "$SHELL_TYPE" = "zsh" ]; then
        # Bash/Zsh quoting
        touch "$filename"
        echo "Größe: 100 MB" > "$filename"
    fi
}

# Main execution
detect_environment
set_encoding
create_file
```

## Quick Reference

### Quoting Rules
- **Bash/Zsh**: Use `"$var"` for expansion, `'literal'` for literal
- **PowerShell**: Use `"$var"` for expansion, `'literal'` for literal
- **CMD**: Use `%var%` for variables, `"path with spaces"`

### Escape Characters
- **Bash/Zsh**: `\` (backslash)
- **PowerShell**: `` ` `` (backtick)
- **CMD**: `^` (caret)

### Path Separators
- **Unix/Linux/macOS**: `/`
- **Windows**: `\` (but PowerShell accepts both `/` and `\`)

### Encoding for German
- **Bash**: `export LANG=de_DE.UTF-8`
- **PowerShell**: `[Console]::OutputEncoding = [System.Text.Encoding]::UTF8`
- **CMD**: `chcp 65001`

## Best Practices

1. **Always detect before executing** - Never assume the shell type
2. **Quote all variables** - Prevents word splitting and globbing
3. **Use UTF-8 encoding** - Essential for German characters
4. **Test on target platform** - Cross-platform issues are common
5. **Provide fallbacks** - Handle unknown shells gracefully
6. **Document shell requirements** - Make dependencies clear
7. **Avoid shell-specific features** - When portability is needed
8. **Check command availability** - Use `command -v` before execution
