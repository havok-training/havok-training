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

## Nested Quoting and Command Composition

**This is the most common source of errors and repeated iterations!**

When commands are nested (calling one shell from another, or embedding commands within commands), quoting becomes exponentially complex. Here are the patterns that work:

### PowerShell.exe Called from CMD

**Problem:** Running PowerShell commands from CMD requires careful escaping.

```cmd
REM WRONG - Will fail with quotes
powershell.exe -Command "Get-Content 'file.txt'"

REM CORRECT - Escape inner quotes
powershell.exe -Command "Get-Content 'file.txt'"

REM CORRECT - Use backslash for paths with spaces
powershell.exe -Command "Get-Content 'C:\Program Files\file.txt'"

REM CORRECT - Complex nested example
powershell.exe -Command "& {Get-ChildItem | Where-Object {$_.Name -like '*test*'}}"
```

### PowerShell.exe Called from Bash/Zsh

**Problem:** Bash interprets quotes and variables before passing to PowerShell.

```bash
# WRONG - Bash interprets $_ and quotes
powershell.exe -Command "Get-Content 'file.txt' | Where-Object {$_.Length -gt 10}"

# CORRECT - Escape or use single quotes to protect from Bash
powershell.exe -Command 'Get-Content "file.txt" | Where-Object {$_.Length -gt 10}'

# CORRECT - Alternative with escaping
powershell.exe -Command "Get-Content 'file.txt' | Where-Object {\$_.Length -gt 10}"

# CORRECT - For paths with spaces, use double escaping
powershell.exe -Command "Get-Content 'C:\Program Files\My App\file.txt'"
```

### Bash/sh Called from PowerShell

**Problem:** PowerShell's quoting and escaping rules differ from Bash.

```powershell
# WRONG - PowerShell interprets the quotes incorrectly
bash -c "echo 'Hello World'"

# CORRECT - Use single quotes in PowerShell, double inside bash
bash -c 'echo "Hello World"'

# CORRECT - Or escape quotes properly
bash -c "echo \`"Hello World\`""

# CORRECT - Complex nested example
bash -c 'grep "pattern" /path/to/file.txt | awk ''{print $1}'''

# CORRECT - Even better, use here-string
$command = @'
grep "pattern" /path/to/file.txt | awk '{print $1}'
'@
bash -c $command
```

### Git Commit Messages with Quotes

**Problem:** Commit messages often contain quotes and need proper escaping.

**Bash - Use HEREDOC (BEST PRACTICE):**
```bash
# CORRECT - Heredoc avoids all quoting issues
git commit -m "$(cat <<'EOF'
Add feature: User's "profile" page

- Implemented user's profile view
- Added "Edit Profile" button
- Fixed issue with apostrophes in names like O'Brien
EOF
)"
```

**PowerShell - Use here-string:**
```powershell
# CORRECT - Here-string
$message = @"
Add feature: User's "profile" page

- Implemented user's profile view
- Added "Edit Profile" button
- Fixed issue with apostrophes in names like O'Brien
"@
git commit -m $message
```

**CMD - Escape quotes:**
```cmd
REM CORRECT - Escape with backslash
git commit -m "Add feature: User's \"profile\" page"
```

### Docker Exec with Nested Commands

**Problem:** Docker exec runs commands inside a container, requiring multiple levels of quoting.

```bash
# WRONG - Quotes not properly nested
docker exec container bash -c "echo 'Hello World'"

# CORRECT - Single quotes around the command
docker exec container bash -c 'echo "Hello World"'

# CORRECT - Complex example with variables
docker exec container bash -c 'grep "error" /var/log/app.log | tail -n 10'

# CORRECT - With file paths containing spaces
docker exec container bash -c 'cat "/app/My Files/config.txt"'
```

**From PowerShell:**
```powershell
# CORRECT - Use single quotes in PowerShell
docker exec container bash -c 'echo "Hello World"'

# CORRECT - Complex nested
docker exec container bash -c 'mysql -e "SELECT * FROM users WHERE name=''John''"'
```

### SSH Commands with Nested Quotes

**Problem:** SSH executes commands on remote hosts, requiring quote preservation.

```bash
# WRONG - Quotes interpreted locally
ssh user@host "grep 'pattern' /var/log/syslog"

# CORRECT - Escape quotes for remote execution
ssh user@host 'grep "pattern" /var/log/syslog'

# CORRECT - Complex nested example
ssh user@host 'docker exec container bash -c "echo \"Hello\""'

# CORRECT - With variables (use backslash escaping)
local_var="pattern"
ssh user@host "grep '$local_var' /var/log/syslog"
```

### JSON Strings as Command Arguments

**Problem:** JSON contains quotes and braces that conflict with shell syntax.

**Bash:**
```bash
# WRONG - Unescaped quotes and braces
curl -X POST -d {"name": "value"} http://api.example.com

# CORRECT - Single quotes protect JSON
curl -X POST -d '{"name": "value", "nested": {"key": "val"}}' http://api.example.com

# CORRECT - With variables, use heredoc
json=$(cat <<EOF
{
  "name": "value",
  "path": "/home/user/My Files"
}
EOF
)
curl -X POST -d "$json" http://api.example.com
```

**PowerShell:**
```powershell
# CORRECT - Use here-string for JSON
$json = @'
{
  "name": "value",
  "nested": {
    "key": "val"
  }
}
'@
Invoke-RestMethod -Method Post -Body $json -Uri 'http://api.example.com'

# CORRECT - Or use ConvertTo-Json
$data = @{
    name = "value"
    path = "C:\Program Files\App"
}
$json = $data | ConvertTo-Json
Invoke-RestMethod -Method Post -Body $json -Uri 'http://api.example.com'
```

### Multiple Levels of Nesting

**Problem:** Commands within commands within commands...

```bash
# Example: SSH to host, run docker, execute bash inside container
# Level 1: Local Bash
# Level 2: SSH command
# Level 3: Docker exec
# Level 4: Bash inside container

# WRONG - Quotes completely broken
ssh user@host "docker exec container bash -c "echo 'Hello'""

# CORRECT - Progressive escaping
ssh user@host 'docker exec container bash -c "echo \"Hello\""'

# CORRECT - For very complex cases, use intermediate scripts
# Create script on remote host first, then execute it
cat > /tmp/script.sh << 'EOF'
docker exec container bash -c 'echo "Hello from container"'
EOF

ssh user@host 'bash /tmp/script.sh'
```

### Heredoc and Here-String Patterns (BEST PRACTICE)

**Use heredocs to avoid quoting hell entirely:**

**Bash Heredoc:**
```bash
# CORRECT - No escaping needed inside heredoc
cat > script.sh << 'EOF'
#!/bin/bash
echo "User's name: O'Brien"
grep "pattern" 'file with spaces.txt'
docker exec container bash -c 'echo "nested command"'
EOF

chmod +x script.sh
./script.sh
```

**PowerShell Here-String:**
```powershell
# CORRECT - No escaping needed inside here-string
$script = @'
Write-Host "User's name: O'Brien"
Get-Content "file with spaces.txt" | Select-String "pattern"
docker exec container bash -c 'echo "nested command"'
'@

$script | Out-File script.ps1
& .\script.ps1
```

### Quick Decision Tree for Nested Quoting

```
Is the command nested?
├─ No → Use normal quoting rules
└─ Yes → How many levels?
    ├─ 1 level (e.g., bash -c "...")
    │   └─ Use opposite quotes: outer double, inner single (or vice versa)
    ├─ 2 levels (e.g., ssh ... docker exec ...)
    │   └─ Use: outer single, middle double, inner escaped double
    └─ 3+ levels or contains JSON/complex strings
        └─ Use heredoc/here-string and avoid inline quoting entirely
```

### Common Nested Quoting Patterns Reference

| Scenario | Shell | Pattern | Example |
|----------|-------|---------|---------|
| Call PowerShell from Bash | Bash | Single quotes around entire command | `powershell.exe -Command 'Get-Content "file.txt"'` |
| Call Bash from PowerShell | PowerShell | Single quotes in PS, double in bash | `bash -c 'echo "Hello"'` |
| Git commit with quotes | Bash | Heredoc | `git commit -m "$(cat <<'EOF'...)"` |
| Git commit with quotes | PowerShell | Here-string | `$msg = @"..."@; git commit -m $msg` |
| Docker exec | Bash | Single outer, double inner | `docker exec c bash -c 'echo "Hi"'` |
| SSH command | Bash | Single quotes | `ssh host 'command "with quotes"'` |
| JSON in curl | Bash | Single quotes | `curl -d '{"key":"val"}' url` |
| JSON in PowerShell | PowerShell | ConvertTo-Json | `$obj | ConvertTo-Json` |

### Anti-Patterns to AVOID

```bash
# ❌ NEVER: Mix quotes without planning nesting
powershell.exe -Command "Get-Content "file.txt""

# ❌ NEVER: Forget to escape $ in bash when calling PowerShell
powershell.exe -Command "Get-Process | Where {$_.Name -eq 'foo'}"
# Should be: '\$_' or use single quotes

# ❌ NEVER: Use complex inline JSON
curl -d "{\"key\":\"value with spaces\",\"nested\":{\"key2\":\"val\"}}"
# Use heredoc or file instead

# ❌ NEVER: Chain multiple levels without testing each level
ssh host "docker exec c bash -c "mysql -e "SELECT * FROM t"""
# Build up from innermost to outermost
```

### Best Practices for Avoiding Quoting Issues

1. **Use heredoc/here-string for anything complex** (3+ lines or 2+ nesting levels)
2. **Test each nesting level separately** before combining
3. **Prefer opposite quote types** at each level (outer double, inner single)
4. **Use intermediate files/scripts** for very complex cases
5. **Escape $ and ` in bash when passing to other shells**
6. **Use ConvertTo-Json in PowerShell** instead of manual JSON strings
7. **Document the nesting levels** with comments when unavoidable

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
