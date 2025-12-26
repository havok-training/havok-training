# Shell Environment Detector - Examples

This file contains practical examples for using the shell environment detector skill.

## Example 1: Simple Shell Detection and Adaptation

**Scenario:** Create a file with German Umlauts that works on both Windows and Unix systems.

```bash
#!/usr/bin/env bash

# Detect the current shell
if [ -n "$BASH_VERSION" ]; then
    echo "Running in Bash"
    SHELL_TYPE="bash"
elif [ -n "$ZSH_VERSION" ]; then
    echo "Running in Zsh"
    SHELL_TYPE="zsh"
else
    echo "Unknown shell - using POSIX sh compatibility"
    SHELL_TYPE="posix"
fi

# Set UTF-8 encoding for German characters
export LANG=de_DE.UTF-8
export LC_ALL=de_DE.UTF-8

# Create file with Umlauts (properly quoted)
filename="Größenübersicht_2024.txt"
touch "$filename"
echo "Datei erstellt: $filename"

# Write content with German characters
cat > "$filename" << 'EOF'
Größenübersicht
===============

Äpfel: 10 Stück
Größe: 5-7 cm
Preis: 2,50 €/kg
EOF

echo "Inhalt geschrieben nach: $filename"
```

**PowerShell equivalent:**

```powershell
# Detect PowerShell version
Write-Host "Running in PowerShell $($PSVersionTable.PSVersion)"

# Set UTF-8 encoding
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$PSDefaultParameterValues['*:Encoding'] = 'utf8'

# Create file with Umlauts
$filename = "Größenübersicht_2024.txt"
New-Item -Path $filename -ItemType File -Force | Out-Null
Write-Host "Datei erstellt: $filename"

# Write content with German characters
@"
Größenübersicht
===============

Äpfel: 10 Stück
Größe: 5-7 cm
Preis: 2,50 €/kg
"@ | Out-File -FilePath $filename -Encoding UTF8

Write-Host "Inhalt geschrieben nach: $filename"
```

## Example 2: Cross-Platform Path Handling

**Scenario:** Handle file paths with spaces and special characters across platforms.

```bash
#!/usr/bin/env bash

# Detect platform
case "$(uname -s)" in
    Linux*)
        PLATFORM="linux"
        BASE_PATH="/home/user/Dokumente"
        ;;
    Darwin*)
        PLATFORM="mac"
        BASE_PATH="/Users/user/Documents"
        ;;
    MINGW*|CYGWIN*|MSYS*)
        PLATFORM="windows"
        BASE_PATH="/c/Users/user/Documents"
        ;;
    *)
        PLATFORM="unknown"
        BASE_PATH="."
        ;;
esac

echo "Platform: $PLATFORM"

# Create directory with German name and spaces
dir_name="Meine Übungen 2024"
full_path="$BASE_PATH/$dir_name"

# Create directory (properly quoted)
mkdir -p "$full_path"
echo "Created: $full_path"

# Create files inside
cd "$full_path" || exit 1

files=(
    "Übung 1 - Einführung.txt"
    "Übung 2 - Größen.txt"
    "Übung 3 - Maße.txt"
)

for file in "${files[@]}"; do
    touch "$file"
    echo "Erstellt: $file"
done

# List files
ls -la
```

**PowerShell equivalent:**

```powershell
# Detect Windows version
$PLATFORM = "windows"
Write-Host "Platform: $PLATFORM"

# Base path for Windows
$BasePath = "$env:USERPROFILE\Documents"

# Create directory with German name and spaces
$dirName = "Meine Übungen 2024"
$fullPath = Join-Path $BasePath $dirName

# Create directory
New-Item -Path $fullPath -ItemType Directory -Force | Out-Null
Write-Host "Created: $fullPath"

# Create files inside
Set-Location $fullPath

$files = @(
    "Übung 1 - Einführung.txt",
    "Übung 2 - Größen.txt",
    "Übung 3 - Maße.txt"
)

foreach ($file in $files) {
    New-Item -Path $file -ItemType File -Force | Out-Null
    Write-Host "Erstellt: $file"
}

# List files
Get-ChildItem
```

## Example 3: Permission Management Across Platforms

**Scenario:** Set executable permissions on a script across Unix and Windows.

```bash
#!/usr/bin/env bash

script_name="deploy.sh"

# Create the script
cat > "$script_name" << 'EOF'
#!/usr/bin/env bash
echo "Deployment script running..."
EOF

# Detect platform and set permissions
if command -v chmod &> /dev/null; then
    # Unix/Linux/macOS
    chmod +x "$script_name"
    echo "Set executable permission with chmod"
    ls -la "$script_name"
elif command -v icacls &> /dev/null; then
    # Windows - grant execute permission
    icacls "$script_name" /grant:r "%USERNAME%:(RX)"
    echo "Set Windows ACL for execute permission"
    icacls "$script_name"
else
    echo "Warning: Unable to set permissions"
fi

# Try to execute
if [ -x "$script_name" ]; then
    ./"$script_name"
else
    bash "$script_name"
fi
```

**PowerShell equivalent:**

```powershell
$scriptName = "deploy.ps1"

# Create the script
@"
Write-Host "Deployment script running..."
"@ | Out-File -FilePath $scriptName -Encoding UTF8

# Windows doesn't need execute permissions for .ps1 files
# But we can set the execution policy or ACLs

# Set ACL to allow current user full control
$acl = Get-Acl $scriptName
$user = [System.Security.Principal.WindowsIdentity]::GetCurrent().Name
$permission = "$user","FullControl","Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule $permission
$acl.SetAccessRule($accessRule)
Set-Acl $scriptName $acl

Write-Host "Set Windows ACL for full control"
Get-Acl $scriptName | Format-List

# Execute (if execution policy allows)
& ".\$scriptName"
```

## Example 4: Escaping Special Characters

**Scenario:** Handle strings with special characters in different shells.

```bash
#!/usr/bin/env bash

# String with various special characters
price='$100'
path='C:\Program Files\App'
regex='^\d+\.\d+$'

# Bash: Use single quotes for literal strings
echo "Bash literal price: $price"

# Use double quotes for expansion
greeting="Hello"
echo "Bash expanded: $greeting, World!"

# Escape special characters in double quotes
echo "Escaped dollar: \$100"
echo "Escaped backslash: C:\\Path\\To\\File"

# For regex, prefer single quotes
if [[ "123.45" =~ $regex ]]; then
    echo "Regex matched"
fi

# Storing command output
current_date=$(date +%Y-%m-%d)
echo "Date: $current_date"
```

**PowerShell equivalent:**

```powershell
# String with special characters
$price = '$100'  # Literal in single quotes
$path = 'C:\Program Files\App'  # Literal
$regex = '^\d+\.\d+$'  # Literal

# PowerShell: Single quotes for literal
Write-Host "PowerShell literal price: $price"

# Double quotes for expansion
$greeting = "Hello"
Write-Host "PowerShell expanded: $greeting, World!"

# Escape special characters in double quotes
Write-Host "Escaped dollar: `$100"
Write-Host "Path with backslashes: C:\Program Files\App"  # No escaping needed in single quotes

# For regex
if ("123.45" -match $regex) {
    Write-Host "Regex matched"
}

# Storing command output
$currentDate = Get-Date -Format "yyyy-MM-dd"
Write-Host "Date: $currentDate"
```

**CMD equivalent:**

```cmd
REM String with special characters
set price=$100
set path=C:\Program Files\App

REM CMD: Use quotes for paths with spaces
echo Literal price: %price%
echo Path: "%path%"

REM Escape special characters with caret
echo Escaped: ^$100
echo Pipe: 5 ^| 6
echo Redirect: ^> file.txt

REM Command substitution is limited in CMD
for /f "tokens=*" %%i in ('date /t') do set current_date=%%i
echo Date: %current_date%
```

## Example 5: Complete Cross-Platform Utility Script

**Scenario:** A utility script that detects environment and adapts accordingly.

```bash
#!/usr/bin/env bash

# ============================================
# Cross-Platform File Manager
# Handles German characters and permissions
# ============================================

# Colors (if supported)
if [ -t 1 ] && command -v tput &> /dev/null; then
    RED=$(tput setaf 1)
    GREEN=$(tput setaf 2)
    YELLOW=$(tput setaf 3)
    RESET=$(tput sgr0)
else
    RED="" GREEN="" YELLOW="" RESET=""
fi

# Detect environment
detect_environment() {
    # Shell detection
    if [ -n "$BASH_VERSION" ]; then
        SHELL_TYPE="bash"
    elif [ -n "$ZSH_VERSION" ]; then
        SHELL_TYPE="zsh"
    else
        SHELL_TYPE="sh"
    fi

    # Platform detection
    case "$(uname -s 2>/dev/null || echo Windows)" in
        Linux*)     PLATFORM="linux";;
        Darwin*)    PLATFORM="mac";;
        MINGW*|CYGWIN*|MSYS*)  PLATFORM="windows";;
        *)          PLATFORM="unknown";;
    esac

    echo "${GREEN}Environment:${RESET} $SHELL_TYPE on $PLATFORM"
}

# Set proper encoding
set_encoding() {
    if [ "$PLATFORM" = "windows" ]; then
        export LANG=de_DE.UTF-8
        export LC_ALL=de_DE.UTF-8
    else
        export LANG=de_DE.UTF-8
        export LC_ALL=de_DE.UTF-8
    fi
    echo "${GREEN}Encoding:${RESET} UTF-8 (German)"
}

# Create directory with proper quoting
create_directory() {
    local dir_name="$1"

    if [ -z "$dir_name" ]; then
        echo "${RED}Error:${RESET} Directory name required"
        return 1
    fi

    mkdir -p "$dir_name"
    echo "${GREEN}Created directory:${RESET} $dir_name"
}

# Set permissions (cross-platform)
set_permissions() {
    local file="$1"
    local mode="$2"

    if [ ! -e "$file" ]; then
        echo "${RED}Error:${RESET} File not found: $file"
        return 1
    fi

    if command -v chmod &> /dev/null; then
        chmod "$mode" "$file"
        echo "${GREEN}Permissions set:${RESET} $mode on $file"
    elif command -v icacls &> /dev/null; then
        echo "${YELLOW}Windows detected:${RESET} Using icacls"
        icacls "$file" /grant:r "%USERNAME%:(RX)"
    else
        echo "${RED}Error:${RESET} Cannot set permissions"
        return 1
    fi
}

# Main execution
main() {
    detect_environment
    set_encoding

    # Create test directory with German characters
    test_dir="Test Übersicht"
    create_directory "$test_dir"

    # Create test files
    cd "$test_dir" || exit 1

    test_files=(
        "Größentabelle.txt"
        "Übersicht Äpfel.txt"
        "Maße und Gewichte.txt"
    )

    for file in "${test_files[@]}"; do
        touch "$file"
        echo "Test content with Umlauts: äöüß ÄÖÜ" > "$file"
        echo "${GREEN}Created:${RESET} $file"
    done

    # Create a script and make it executable
    script_name="test_script.sh"
    cat > "$script_name" << 'EOF'
#!/usr/bin/env bash
echo "Script executed successfully!"
EOF

    set_permissions "$script_name" "755"

    # List all files
    echo ""
    echo "${GREEN}Files created:${RESET}"
    ls -la

    cd ..
}

# Run
main
```

## Example 6: Handling Quotes in File Names

**Scenario:** Work with file names containing quotes, apostrophes, and special characters.

```bash
#!/usr/bin/env bash

# Files with challenging names
files=(
    "File with 'single quotes'.txt"
    'File with "double quotes".txt'
    "File with both 'single' and \"double\" quotes.txt"
    "Größe 10cm - Version 1.0.txt"
)

# Create files safely
for file in "${files[@]}"; do
    # Touch uses the variable safely
    touch "$file"
    echo "Created: $file"
done

# Read files safely
for file in "${files[@]}"; do
    if [ -f "$file" ]; then
        echo "Reading: $file"
        # cat "$file"
    fi
done

# Remove files safely
for file in "${files[@]}"; do
    rm -f "$file"
    echo "Removed: $file"
done
```

## Example 7: Nested Quoting - The Most Common Pain Point

**Scenario:** Complex nested command scenarios that cause repeated iterations and failures.

### PowerShell from Bash (Git Bash on Windows)

```bash
#!/usr/bin/env bash
# Running on Git Bash, calling PowerShell

# ❌ WRONG - Bash interprets $_ before PowerShell sees it
powershell.exe -Command "Get-Process | Where-Object {$_.Name -eq 'chrome'}"

# ✅ CORRECT - Use single quotes to protect from Bash
powershell.exe -Command 'Get-Process | Where-Object {$_.Name -eq "chrome"}'

# ✅ CORRECT - Alternative: escape the dollar sign
powershell.exe -Command "Get-Process | Where-Object {\$_.Name -eq 'chrome'}"

# ✅ CORRECT - Complex example with paths
powershell.exe -Command 'Get-Content "C:\Program Files\My App\config.txt" | Select-String "pattern"'
```

### Bash from PowerShell

```powershell
# Running in PowerShell, calling Bash (WSL or Git Bash)

# ❌ WRONG - PowerShell escaping incorrect
bash -c "echo 'Hello World'"

# ✅ CORRECT - Use single quotes in PowerShell
bash -c 'echo "Hello World"'

# ✅ CORRECT - Or use backtick escaping
bash -c "echo \`"Hello World\`""

# ✅ CORRECT - Complex example with grep and awk
bash -c 'grep "error" /var/log/app.log | awk ''{print $1}'''

# ✅ BEST - Use here-string for complex commands
$bashCmd = @'
grep "error" /var/log/app.log | awk '{print $1}' | sort | uniq
'@
bash -c $bashCmd
```

### Git Commit with Complex Messages

```bash
#!/usr/bin/env bash

# ❌ WRONG - Quotes break the commit message
git commit -m "Fix user's "profile" page bug"

# ❌ WRONG - Escaping becomes messy
git commit -m "Fix user's \"profile\" page bug with O'Brien's account"

# ✅ BEST PRACTICE - Use heredoc
git commit -m "$(cat <<'EOF'
Fix user's "profile" page bug

- Resolved issue with O'Brien's account
- Updated "Edit Profile" functionality
- Added validation for names with apostrophes (e.g., O'Sullivan)
EOF
)"
```

**PowerShell version:**
```powershell
# ✅ BEST PRACTICE - Use here-string
$commitMessage = @"
Fix user's "profile" page bug

- Resolved issue with O'Brien's account
- Updated "Edit Profile" functionality
- Added validation for names with apostrophes (e.g., O'Sullivan)
"@

git commit -m $commitMessage
```

### Docker Exec Nested Commands

```bash
#!/usr/bin/env bash

# ❌ WRONG - Quote nesting broken
docker exec mycontainer bash -c "mysql -e "SELECT * FROM users""

# ✅ CORRECT - Alternate quotes at each level
docker exec mycontainer bash -c 'mysql -e "SELECT * FROM users WHERE name=\"John\""'

# ✅ CORRECT - Complex example with grep
docker exec mycontainer bash -c 'grep "ERROR" /var/log/app.log | tail -n 20'

# ✅ CORRECT - With file paths containing spaces
docker exec mycontainer bash -c 'cat "/app/My Files/Größenübersicht.txt"'

# ✅ BEST - For very complex cases, use heredoc
docker exec mycontainer bash -c "$(cat <<'EOF'
cd /app
mysql -e "SELECT * FROM users WHERE name='O\"Brien'"
grep "error" "log files/app.log"
EOF
)"
```

**From PowerShell:**
```powershell
# ✅ CORRECT - Use single quotes in PowerShell for bash commands
docker exec mycontainer bash -c 'mysql -e "SELECT * FROM users"'

# ✅ CORRECT - Double single quotes for literal single quotes in SQL
docker exec mycontainer bash -c 'mysql -e "SELECT * FROM users WHERE name=''John''"'
```

### SSH with Nested Commands

```bash
#!/usr/bin/env bash

# ❌ WRONG - Quotes interpreted locally instead of remotely
ssh user@remote "grep 'error' /var/log/syslog"

# ✅ CORRECT - Use single quotes to preserve for remote execution
ssh user@remote 'grep "error" /var/log/syslog'

# ✅ CORRECT - Multiple nested levels (SSH -> Docker -> Bash)
ssh user@remote 'docker exec container bash -c "grep \"error\" /var/log/app.log"'

# ✅ CORRECT - With local variable expansion
LOG_PATTERN="error"
ssh user@remote "grep '$LOG_PATTERN' /var/log/syslog"

# ✅ BEST - Very complex: use intermediate script
cat > /tmp/remote_script.sh << 'EOF'
#!/bin/bash
docker exec mycontainer bash -c 'mysql -e "SELECT * FROM users WHERE name=\"admin\""'
EOF

scp /tmp/remote_script.sh user@remote:/tmp/
ssh user@remote 'bash /tmp/remote_script.sh'
```

### JSON in API Calls

```bash
#!/usr/bin/env bash

# ❌ WRONG - Unescaped JSON breaks
curl -X POST -d {"name": "John", "role": "admin"} http://api.example.com

# ❌ WRONG - Escaping becomes nightmare
curl -X POST -d "{\"name\": \"John\", \"role\": \"admin\"}" http://api.example.com

# ✅ CORRECT - Use single quotes for JSON
curl -X POST -H "Content-Type: application/json" \
  -d '{"name": "John", "role": "admin"}' \
  http://api.example.com

# ✅ CORRECT - Complex nested JSON
curl -X POST -H "Content-Type: application/json" \
  -d '{"user": {"name": "John", "meta": {"role": "admin"}}}' \
  http://api.example.com

# ✅ BEST - Use heredoc for complex JSON
JSON_DATA=$(cat <<'EOF'
{
  "user": {
    "name": "O'Brien",
    "path": "/home/user/My Files",
    "metadata": {
      "role": "admin",
      "status": "active"
    }
  }
}
EOF
)

curl -X POST -H "Content-Type: application/json" \
  -d "$JSON_DATA" \
  http://api.example.com
```

**PowerShell version:**
```powershell
# ✅ BEST - Use here-string for JSON
$jsonData = @'
{
  "user": {
    "name": "O'Brien",
    "path": "C:\\Users\\user\\My Files",
    "metadata": {
      "role": "admin",
      "status": "active"
    }
  }
}
'@

Invoke-RestMethod -Method Post -Uri 'http://api.example.com' `
  -Body $jsonData -ContentType 'application/json'

# ✅ EVEN BETTER - Use PowerShell objects
$data = @{
  user = @{
    name = "O'Brien"
    path = "C:\Users\user\My Files"
    metadata = @{
      role = "admin"
      status = "active"
    }
  }
}

$jsonData = $data | ConvertTo-Json -Depth 10
Invoke-RestMethod -Method Post -Uri 'http://api.example.com' `
  -Body $jsonData -ContentType 'application/json'
```

### Multiple Nesting Levels - Real World Example

**Scenario:** Deploy via SSH to a remote host that runs Docker containers with MySQL inside.

```bash
#!/usr/bin/env bash

# ❌ WRONG - Total quote chaos
ssh deploy@prod "docker exec mysql bash -c "mysql -e "CREATE DATABASE app"""

# ✅ WORKS - But hard to read
ssh deploy@prod 'docker exec mysql bash -c "mysql -e \"CREATE DATABASE app\""'

# ✅ BEST PRACTICE - Use intermediate script
cat > /tmp/deploy.sh << 'EOF'
#!/bin/bash
# This runs on the remote host

docker exec mysql bash -c 'mysql -e "CREATE DATABASE app"'
docker exec mysql bash -c 'mysql app -e "CREATE TABLE users (id INT, name VARCHAR(100))"'
docker exec mysql bash -c 'mysql app -e "INSERT INTO users VALUES (1, \"O'"'"'Brien\")"'

echo "Database setup complete"
EOF

# Copy script to remote host
scp /tmp/deploy.sh deploy@prod:/tmp/

# Execute remotely
ssh deploy@prod 'bash /tmp/deploy.sh'

# Cleanup
ssh deploy@prod 'rm /tmp/deploy.sh'
```

**PowerShell version:**
```powershell
# ✅ BEST PRACTICE - Use here-string for complex script
$deployScript = @'
#!/bin/bash
docker exec mysql bash -c 'mysql -e "CREATE DATABASE app"'
docker exec mysql bash -c 'mysql app -e "CREATE TABLE users (id INT, name VARCHAR(100))"'
docker exec mysql bash -c 'mysql app -e "INSERT INTO users VALUES (1, \"O'Brien\")"'
echo "Database setup complete"
'@

# Save locally
$deployScript | Out-File -FilePath /tmp/deploy.sh -Encoding UTF8

# Copy to remote (using scp or pscp)
scp /tmp/deploy.sh deploy@prod:/tmp/

# Execute remotely
ssh deploy@prod 'bash /tmp/deploy.sh'

# Cleanup
ssh deploy@prod 'rm /tmp/deploy.sh'
```

### Decision Tree for Nested Quoting

```bash
#!/usr/bin/env bash

# Level 0: Simple command (no nesting)
echo "Hello World"

# Level 1: One level of nesting - Use opposite quotes
bash -c 'echo "Hello World"'
powershell.exe -Command 'Write-Host "Hello World"'

# Level 2: Two levels of nesting - Alternate and escape
ssh user@host 'bash -c "echo \"Hello World\""'
ssh user@host 'docker exec container bash -c "echo \"Hello\""'

# Level 3+: Multiple levels - Use heredoc/scripts
cat > /tmp/script.sh << 'EOF'
docker exec container bash -c 'mysql -e "SELECT * FROM users WHERE name=\"admin\""'
EOF
ssh user@host 'bash /tmp/script.sh'
```

### Anti-Pattern Examples (What NOT to Do)

```bash
#!/usr/bin/env bash

# ❌ ANTI-PATTERN 1: Same quotes at multiple levels
powershell.exe -Command "Get-Content "file.txt""  # BROKEN

# ❌ ANTI-PATTERN 2: Forgetting to escape $ in Bash
powershell.exe -Command "Get-Process | Where {$_.Name -eq 'foo'}"  # Bash expands $_

# ❌ ANTI-PATTERN 3: Complex inline JSON
curl -d "{\"user\":{\"name\":\"O'Brien\",\"path\":\"C:\\\\Files\"}}" api  # NIGHTMARE

# ❌ ANTI-PATTERN 4: Excessive escaping
echo "He said: \"She's from the \\\"Big Apple\\\"\"" # HARD TO READ

# ❌ ANTI-PATTERN 5: Not testing each nesting level separately
ssh host "docker exec c bash -c "mysql -e "SELECT * FROM t"""  # IMPOSSIBLE TO DEBUG
```

### Best Practice Summary

```bash
#!/usr/bin/env bash

# ✅ Rule 1: Use heredoc for 3+ lines or 2+ nesting levels
COMPLEX_CMD=$(cat <<'EOF'
Multi-line command with "quotes" and 'apostrophes'
Can include anything without escaping
Perfect for JSON, SQL, or complex scripts
EOF
)

# ✅ Rule 2: Alternate quote types at each nesting level
# Outer: single, Inner: double (or vice versa)
docker exec c bash -c 'echo "Hello"'

# ✅ Rule 3: Test each level separately before combining
# Test innermost first:
echo "Hello"
# Then wrap in bash -c:
bash -c 'echo "Hello"'
# Then wrap in docker exec:
docker exec c bash -c 'echo "Hello"'
# Finally wrap in ssh:
ssh user@host 'docker exec c bash -c "echo \"Hello\""'

# ✅ Rule 4: Use intermediate scripts for very complex cases
# Instead of: ssh host "docker exec c bash -c \"complex command\""
# Do this:
cat > script.sh << 'EOF'
docker exec c bash -c 'complex command'
EOF
ssh host 'bash ./script.sh'

# ✅ Rule 5: Document nesting levels with comments
# Level 1: SSH to remote host
# Level 2: Docker exec into container
# Level 3: Bash command inside container
# Level 4: MySQL query with quotes
ssh host 'docker exec c bash -c "mysql -e \"SELECT * FROM t\""'
```

These examples demonstrate the key concepts from the shell environment detector skill in practical scenarios.
