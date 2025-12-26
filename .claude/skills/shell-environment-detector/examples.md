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

These examples demonstrate the key concepts from the shell environment detector skill in practical scenarios.
