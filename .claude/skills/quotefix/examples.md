# Quotefix - Examples

This file contains practical examples for using the quotefix skill.

## Example 1: Simple Shell Detection and Adaptation

**Scenario:** Create a file that works on both Windows and Unix systems.

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

# Create file with spaces in name (properly quoted)
filename="Project Overview 2024.txt"
touch "$filename"
echo "File created: $filename"

# Write content
cat > "$filename" << 'EOF'
Project Overview
===============

Items: 10 pieces
Size: 5-7 cm
Price: $2.50/kg
EOF

echo "Content written to: $filename"
```

**PowerShell equivalent:**

```powershell
# Detect PowerShell version
Write-Host "Running in PowerShell $($PSVersionTable.PSVersion)"

# Create file with spaces in name
$filename = "Project Overview 2024.txt"
New-Item -Path $filename -ItemType File -Force | Out-Null
Write-Host "File created: $filename"

# Write content
@"
Project Overview
===============

Items: 10 pieces
Size: 5-7 cm
Price: $2.50/kg
"@ | Out-File -FilePath $filename -Encoding UTF8

Write-Host "Content written to: $filename"
```

## Example 2: Cross-Platform Path Handling

**Scenario:** Handle file paths with spaces and special characters across platforms.

```bash
#!/usr/bin/env bash

# Detect platform
case "$(uname -s)" in
    Linux*)
        PLATFORM="linux"
        BASE_PATH="/home/user/Documents"
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

# Create directory with spaces in name
dir_name="My Projects 2024"
full_path="$BASE_PATH/$dir_name"

# Create directory (properly quoted)
mkdir -p "$full_path"
echo "Created: $full_path"

# Create files inside
cd "$full_path" || exit 1

files=(
    "Exercise 1 - Introduction.txt"
    "Exercise 2 - Sizes.txt"
    "Exercise 3 - Measurements.txt"
)

for file in "${files[@]}"; do
    touch "$file"
    echo "Created: $file"
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

# Create directory with spaces in name
$dirName = "My Projects 2024"
$fullPath = Join-Path $BasePath $dirName

# Create directory
New-Item -Path $fullPath -ItemType Directory -Force | Out-Null
Write-Host "Created: $fullPath"

# Create files inside
Set-Location $fullPath

$files = @(
    "Exercise 1 - Introduction.txt",
    "Exercise 2 - Sizes.txt",
    "Exercise 3 - Measurements.txt"
)

foreach ($file in $files) {
    New-Item -Path $file -ItemType File -Force | Out-Null
    Write-Host "Created: $file"
}

# List files
Get-ChildItem
```

## Example 3: Escaping Special Characters

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

## Example 4: PowerShell Here-Strings

**Scenario:** Using PowerShell here-strings for literal and expandable multi-line content.

```powershell
# ========================================
# PowerShell Here-Strings
# ========================================

# Expandable here-string (double quotes)
# Variables are expanded, special characters are interpreted
$username = "JohnDoe"
$path = "C:\Users\$username"

$expandableString = @"
User: $username
Path: $path
Date: $(Get-Date -Format 'yyyy-MM-dd')
Price: `$100.00
"@

Write-Host $expandableString

# Literal here-string (single quotes)
# Everything is literal, no expansion
$literalString = @'
User: $username
Path: $path
Special chars: \ $ @ " ' `
No escaping needed!
"Quotes" and 'apostrophes' work fine.
Even backticks ` don't escape anything here.
'@

Write-Host $literalString

# Common use case: SQL queries
$sqlQuery = @"
SELECT * FROM Users
WHERE Name = 'O''Brien'
  AND Path = 'C:\Program Files\App'
  AND Status = 'Active'
"@

# Invoke-SqlCmd -Query $sqlQuery

# Common use case: JSON data
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

$object = $jsonData | ConvertFrom-Json

# Common use case: XML/HTML
$htmlContent = @"
<!DOCTYPE html>
<html>
<head>
    <title>$username's Page</title>
</head>
<body>
    <h1>Welcome $username!</h1>
    <p>Path: $path</p>
</body>
</html>
"@

# Common use case: Multi-line regex
$regexPattern = @'
(?x)                    # Enable verbose mode
^                       # Start of line
\d{3}                   # Three digits
[-\s]?                  # Optional dash or space
\d{3}                   # Three digits
[-\s]?                  # Optional dash or space
\d{4}                   # Four digits
$                       # End of line
'@

if ("555-123-4567" -match $regexPattern) {
    Write-Host "Phone number is valid"
}
```

**Bash heredoc equivalent:**

```bash
#!/usr/bin/env bash

# Expandable heredoc (quotes can be omitted or use double quotes)
username="JohnDoe"
path="/home/$username"

cat << EOF
User: $username
Path: $path
Date: $(date +%Y-%m-%d)
Price: \$100.00
EOF

# Literal heredoc (single quotes prevent expansion)
cat << 'EOF'
User: $username
Path: $path
Special chars: \ $ @ " '
No escaping needed!
"Quotes" and 'apostrophes' work fine.
EOF

# Store in variable
sql_query=$(cat << 'EOF'
SELECT * FROM Users
WHERE Name = 'O\'Brien'
  AND Path = '/usr/local/app'
  AND Status = 'Active'
EOF
)
```

## Example 5: PowerShell Stop-Parsing Token (--%​)

**Scenario:** Using the stop-parsing token to pass arguments literally to external programs.

```powershell
# ========================================
# PowerShell Stop-Parsing Token (--%)
# ========================================

# Problem: PowerShell interprets arguments before passing to external program
# This often breaks when calling CMD, Git, or other external tools

# ❌ WRONG - PowerShell interprets the quotes and variables
git commit -m "Fix user's "profile" bug"  # BROKEN

# ❌ WRONG - Even escaping can be problematic
git commit -m "Fix user's `"profile`" bug"  # Still tricky

# ✅ CORRECT - Use stop-parsing token
git commit --% -m "Fix user's "profile" bug"

# The --% tells PowerShell to stop interpreting and pass everything literally

# Example: Calling CMD commands
cmd --% /c dir "C:\Program Files" /s

# Example: Calling icacls with complex arguments
icacls --% "C:\My Files\test.txt" /grant:r "Domain\User:(RX)"

# Example: Git with complex commit messages
git commit --% -m "Feature: Support O'Brien's "special" characters"

# Example: Calling curl with complex arguments
curl --% -X POST -H "Content-Type: application/json" -d "{\"name\":\"John\"}" http://api.example.com

# Example: Docker with quotes
docker run --% --name "my-container" -v "C:\My Files:/data" ubuntu bash

# ⚠️ LIMITATION: Cannot mix variables with --%
# This does NOT work:
# $user = "JohnDoe"
# icacls --% $file /grant:r "$user:(RX)"  # $file and $user won't expand

# Workaround: Build the command string first
$user = "JohnDoe"
$file = "C:\My Files\test.txt"
$icaclsArgs = "$file /grant:r `"$user:(RX)`""
icacls --% $icaclsArgs  # Still problematic

# Better workaround: Use Start-Process with ArgumentList
$icaclsArgs = @(
    $file,
    "/grant:r",
    "`"$user:(RX)`""
)
Start-Process icacls -ArgumentList $icaclsArgs -NoNewWindow -Wait

# Or use cmd /c as intermediary
cmd /c "icacls `"$file`" /grant:r `"$user:(RX)`""
```

## Example 6: PowerShell Comparison Operators

**Scenario:** Understanding PowerShell comparison operators and their quoting requirements.

```powershell
# ========================================
# PowerShell Comparison Operators
# ========================================

# PowerShell uses different operators than Bash
# -eq (equal), -ne (not equal), -gt (greater than), -lt (less than), etc.

# String comparison
$name = "John"

if ($name -eq "John") {
    Write-Host "Name is John"
}

if ($name -ne "Jane") {
    Write-Host "Name is not Jane"
}

# Case-sensitive comparison
if ($name -ceq "john") {
    Write-Host "Case-sensitive match"  # Won't match
}

# Numeric comparison
$count = 10

if ($count -gt 5) {
    Write-Host "Count is greater than 5"
}

if ($count -le 20) {
    Write-Host "Count is less than or equal to 20"
}

# String contains
if ($name -like "*oh*") {
    Write-Host "Name contains 'oh'"
}

# Regex match
if ($name -match "^J") {
    Write-Host "Name starts with J"
}

# Common mistake when calling from Bash
# ❌ WRONG - Bash doesn't understand -eq
# bash -c "if [ $count -eq 10 ]; then echo 'yes'; fi"

# ❌ WRONG from Bash - single = is assignment, not comparison
# bash -c "if [ $count = 10 ]; then echo 'yes'; fi"

# ✅ CORRECT - Bash uses different syntax
bash -c 'if [ 10 -eq 10 ]; then echo "yes"; fi'

# Or use double brackets with =
bash -c 'if [[ "10" == "10" ]]; then echo "yes"; fi'

# When calling PowerShell from Bash with comparison operators
# ✅ CORRECT - Protect from Bash interpretation
powershell.exe -Command 'Get-Process | Where-Object {$_.CPU -gt 100}'

# ❌ WRONG - Bash might interpret > as redirect
# powershell.exe -Command "Get-Process | Where-Object {\$_.CPU > 100}"

# Comparison in Where-Object
Get-Process | Where-Object {$_.Name -eq "chrome"}
Get-Process | Where-Object {$_.CPU -gt 50}
Get-Service | Where-Object {$_.Status -ne "Running"}

# Multiple conditions
Get-Process | Where-Object {$_.Name -eq "chrome" -and $_.CPU -gt 50}
Get-Service | Where-Object {$_.Status -eq "Running" -or $_.Status -eq "Starting"}

# When these operators appear in nested commands
# ✅ CORRECT - From Git Bash
bash -c 'powershell.exe -Command "Get-Process | Where-Object {\$_.Name -eq \"chrome\"}"'

# ✅ BETTER - Use single quotes to avoid Bash expansion
bash -c "powershell.exe -Command 'Get-Process | Where-Object {\$_.Name -eq \"chrome\"}'"
```

**Bash comparison equivalent:**

```bash
#!/usr/bin/env bash

# Bash uses different comparison operators

# String comparison
name="John"

if [ "$name" = "John" ]; then
    echo "Name is John"
fi

if [ "$name" != "Jane" ]; then
    echo "Name is not Jane"
fi

# Or use double brackets (more modern)
if [[ "$name" == "John" ]]; then
    echo "Name is John"
fi

# Numeric comparison (uses -eq, -ne, -gt, -lt)
count=10

if [ $count -eq 10 ]; then
    echo "Count equals 10"
fi

if [ $count -gt 5 ]; then
    echo "Count is greater than 5"
fi

# String pattern matching
if [[ "$name" == *oh* ]]; then
    echo "Name contains 'oh'"
fi

# Regex match
if [[ "$name" =~ ^J ]]; then
    echo "Name starts with J"
fi
```

## Example 7: Calling External Programs from PowerShell

**Scenario:** Properly quoting when calling external programs from PowerShell.

```powershell
# ========================================
# Calling External Programs from PowerShell
# ========================================

# PowerShell's argument parsing can be tricky when calling external programs
# (programs not written in PowerShell, like .exe, .bat, .cmd files)

# Problem: PowerShell tries to parse arguments before passing them

# ❌ WRONG - PowerShell removes quotes before passing to program
git.exe commit -m "Fix bug"  # Git receives: Fix bug (no quotes)

# ✅ CORRECT - Escape inner quotes
git.exe commit -m "`"Fix bug`""

# ✅ BETTER - Use single quotes (literal)
git commit -m 'Fix bug'

# ✅ BEST - For complex messages, use here-string
$message = @'
Fix user's "profile" bug

- Resolved issue with O'Brien's account
- Updated "Edit Profile" functionality
'@
git commit -m $message

# Calling CMD commands
# ❌ WRONG - Quotes get mangled
cmd /c dir "C:\Program Files"

# ✅ CORRECT - Use backtick escaping
cmd /c dir `"C:\Program Files`"

# ✅ CORRECT - Or use --% stop-parsing token
cmd --% /c dir "C:\Program Files"

# Calling Python scripts
# ❌ WRONG - Argument with spaces breaks
python script.py C:\My Files\data.txt

# ✅ CORRECT - Quote the argument
python script.py "C:\My Files\data.txt"

# ✅ CORRECT - Or use backtick escaping in expandable string
$path = "C:\My Files\data.txt"
python script.py "`"$path`""

# Calling Node.js
# ❌ WRONG - JSON argument breaks
node app.js {"name":"John"}

# ✅ CORRECT - Use single quotes for JSON
node app.js '{\"name\":\"John\"}'

# ✅ BETTER - Use here-string
$jsonConfig = @'
{"name":"John","role":"admin"}
'@
node app.js $jsonConfig

# Calling curl
# ❌ WRONG - Quotes break
curl -X POST -d {"name":"John"} http://api.example.com

# ✅ CORRECT - Use single quotes for JSON
curl -X POST -H 'Content-Type: application/json' -d '{\"name\":\"John\"}' http://api.example.com

# ✅ BEST - Use here-string
$jsonData = @'
{"name":"John","role":"admin"}
'@
curl -X POST -H 'Content-Type: application/json' -d $jsonData http://api.example.com

# Calling Docker
# ❌ WRONG - Path with spaces breaks
docker run -v C:\My Files:/data ubuntu

# ✅ CORRECT - Quote the path
docker run -v "C:\My Files:/data" ubuntu

# ✅ CORRECT - With expanded variable
$hostPath = "C:\My Files"
docker run -v "`"${hostPath}:/data`"" ubuntu

# Using Start-Process for better control
# ✅ BEST PRACTICE - Use ArgumentList for complex arguments
$arguments = @(
    '-v',
    'C:\My Files:/data',
    'ubuntu',
    'bash'
)
Start-Process docker -ArgumentList $arguments -NoNewWindow -Wait

# Calling Az CLI
# Complex quoting scenario
az vm create --name "MyVM" --resource-group "MyRG" --image "UbuntuLTS"

# With JSON parameters
$tags = @'
{"Environment":"Production","Owner":"John"}
'@
az vm create --name "MyVM" --resource-group "MyRG" --tags $tags

# Call operator & for programs with spaces in path
& "C:\Program Files\My App\app.exe" --arg1 value1

# With arguments containing quotes
& "C:\Program Files\My App\app.exe" --message "`"Hello World`""
```

## Example 8: Azure CLI Quoting Differences

**Scenario:** Azure CLI behaves differently on Windows vs Unix due to shell differences.

```powershell
# ========================================
# Azure CLI Quoting - Windows PowerShell
# ========================================

# Azure CLI (az) has different quoting requirements on Windows vs Linux/macOS

# Simple command (no difference)
az group list --output table

# JSON parameters - Windows PowerShell
# ✅ CORRECT - Use single quotes for outer, double quotes for JSON
az vm create `
  --name "MyVM" `
  --resource-group "MyRG" `
  --image "UbuntuLTS" `
  --tags '{"Environment":"Production","Owner":"John"}'

# ✅ CORRECT - Or escape double quotes
az vm create `
  --name "MyVM" `
  --resource-group "MyRG" `
  --image "UbuntuLTS" `
  --tags "{`"Environment`":`"Production`",`"Owner`":`"John`"}"

# ✅ BEST - Use here-string
$tags = @'
{"Environment":"Production","Owner":"John"}
'@
az vm create --name "MyVM" --resource-group "MyRG" --tags $tags

# Complex query with JMESPath
# ❌ WRONG - Quotes break the query
az vm list --query "[?location=='eastus']"

# ✅ CORRECT - Use backtick escaping
az vm list --query "[?location==``'eastus``']"

# ✅ BETTER - Use single quotes (literal)
az vm list --query '[?location==`"eastus`"]'

# ✅ BEST - Use here-string for complex queries
$query = @'
[?location=='eastus' && powerState=='Running'].{Name:name,RG:resourceGroup}
'@
az vm list --query $query --output table

# Creating resources with complex JSON
$vmConfig = @'
{
  "location": "eastus",
  "properties": {
    "hardwareProfile": {
      "vmSize": "Standard_DS1_v2"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "Canonical",
        "offer": "UbuntuServer",
        "sku": "18.04-LTS"
      }
    },
    "osProfile": {
      "computerName": "myVM",
      "adminUsername": "azureuser"
    }
  }
}
'@

# Save to file and use @file.json syntax
$vmConfig | Out-File -FilePath vm.json -Encoding UTF8
az vm create --name "MyVM" --resource-group "MyRG" --parameters '@vm.json'

# Or pass inline (not recommended for complex JSON)
az deployment group create --resource-group "MyRG" --template-file template.json --parameters $vmConfig
```

```bash
#!/usr/bin/env bash

# ========================================
# Azure CLI Quoting - Linux/macOS/Git Bash
# ========================================

# Simple command (no difference)
az group list --output table

# JSON parameters - Unix shells
# ✅ CORRECT - Use single quotes for JSON (prevents shell expansion)
az vm create \
  --name "MyVM" \
  --resource-group "MyRG" \
  --image "UbuntuLTS" \
  --tags '{"Environment":"Production","Owner":"John"}'

# Complex query with JMESPath
# ✅ CORRECT - Single quotes protect from shell
az vm list --query '[?location==`eastus`]' --output table

# ✅ CORRECT - For complex queries
az vm list --query '[?location==`eastus` && powerState==`Running`].{Name:name,RG:resourceGroup}' --output table

# Creating resources with complex JSON
vm_config=$(cat <<'EOF'
{
  "location": "eastus",
  "properties": {
    "hardwareProfile": {
      "vmSize": "Standard_DS1_v2"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "Canonical",
        "offer": "UbuntuServer",
        "sku": "18.04-LTS"
      }
    },
    "osProfile": {
      "computerName": "myVM",
      "adminUsername": "azureuser"
    }
  }
}
EOF
)

# Save to file
echo "$vm_config" > vm.json
az vm create --name "MyVM" --resource-group "MyRG" --parameters '@vm.json'

# Key difference: Windows often needs more escaping due to PowerShell interpretation
# Unix shells with single quotes keep everything literal

# Calling Az CLI from PowerShell in WSL/Git Bash
# ✅ CORRECT
bash -c 'az vm list --query "[?location==\`eastus\`]" --output table'
```

## Example 9: Complete Cross-Platform Utility Script

**Scenario:** A utility script that detects environment and adapts accordingly.

```bash
#!/usr/bin/env bash

# ============================================
# Cross-Platform File Manager
# Handles paths with spaces and special characters
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

# Process files with spaces in names
process_files() {
    local search_dir="${1:-.}"

    echo "${GREEN}Processing files in:${RESET} $search_dir"

    # Use find with -print0 and read with -d '' for files with special characters
    while IFS= read -r -d '' file; do
        echo "  Processing: $file"
        # Add processing logic here
    done < <(find "$search_dir" -type f -name "*.txt" -print0)
}

# Main execution
main() {
    detect_environment

    # Create test directory with spaces
    test_dir="Test Projects 2024"
    create_directory "$test_dir"

    # Create test files
    cd "$test_dir" || exit 1

    test_files=(
        "Size Chart.txt"
        "Overview Report.txt"
        "Measurements and Weights.txt"
        "File with 'quotes'.txt"
    )

    for file in "${test_files[@]}"; do
        touch "$file"
        echo "Test content for: $file" > "$file"
        echo "${GREEN}Created:${RESET} $file"
    done

    # List all files
    echo ""
    echo "${GREEN}Files created:${RESET}"
    ls -la

    # Process files
    cd ..
    process_files "$test_dir"
}

# Run
main
```

## Example 10: Handling Quotes in File Names

**Scenario:** Work with file names containing quotes, apostrophes, and special characters.

```bash
#!/usr/bin/env bash

# Files with challenging names
files=(
    "File with 'single quotes'.txt"
    'File with "double quotes".txt'
    "File with both 'single' and \"double\" quotes.txt"
    "O'Brien's File - Version 1.0.txt"
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

## Example 11: Nested Quoting - The Most Common Pain Point

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

# ✅ CORRECT - With comparison operators
powershell.exe -Command 'Get-Process | Where-Object {$_.CPU -gt 100}'

# ✅ CORRECT - Multiple conditions
powershell.exe -Command 'Get-Service | Where-Object {$_.Status -eq "Running" -and $_.Name -like "win*"}'
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

### Calling PowerShell from CMD

```cmd
REM Running in CMD, calling PowerShell

REM ❌ WRONG - Quotes break
powershell.exe -Command "Get-Process | Where-Object {$_.Name -eq 'chrome'}"

REM ✅ CORRECT - Use escaped quotes
powershell.exe -Command "Get-Process | Where-Object {$_.Name -eq 'chrome'}"

REM ✅ BETTER - Use single quotes
powershell.exe -Command 'Get-Process | Where-Object {$_.Name -eq "chrome"}'

REM ✅ BEST - Use -File parameter with script file
echo Get-Process | Where-Object {$_.Name -eq "chrome"} > temp.ps1
powershell.exe -File temp.ps1
del temp.ps1
```

### Calling CMD from PowerShell

```powershell
# Running in PowerShell, calling CMD

# ❌ WRONG - Quotes get mangled
cmd /c dir "C:\Program Files"

# ✅ CORRECT - Use --% stop-parsing token
cmd --% /c dir "C:\Program Files"

# ✅ CORRECT - Use backtick escaping
cmd /c dir `"C:\Program Files`"

# ✅ CORRECT - Calling CMD batch file with arguments
cmd /c mybatch.bat `"C:\My Files`" `"Output Dir`"

# ✅ CORRECT - Complex CMD command
cmd --% /c for /f "tokens=*" %i in ('dir /b *.txt') do echo %i

# ✅ BEST - For very complex cases, create temporary .cmd file
$cmdScript = @'
@echo off
for /f "tokens=*" %%i in ('dir /b *.txt') do (
    echo Processing: %%i
)
'@
$cmdScript | Out-File -FilePath temp.cmd -Encoding ASCII
cmd /c temp.cmd
Remove-Item temp.cmd
```

### Windows Paths with Spaces in Commands

```bash
#!/usr/bin/env bash
# Running on Git Bash on Windows

# ❌ WRONG - Path with spaces breaks
cd C:\Program Files\My App
/c/Program Files/My App/app.exe

# ✅ CORRECT - Quote the path
cd "/c/Program Files/My App"
"/c/Program Files/My App/app.exe"

# ✅ CORRECT - Calling PowerShell with Windows path with spaces
powershell.exe -Command 'Get-Content "C:\Program Files\My App\config.txt"'

# ✅ CORRECT - Passing path to Windows program
powershell.exe -Command "& 'C:\Program Files\My App\app.exe' --arg1 value1"

# ✅ CORRECT - Complex: Bash -> PowerShell -> Windows program with spaces
powershell.exe -Command '& "C:\Program Files\My App\app.exe" --config "C:\My Files\config.txt"'
```

```powershell
# Running in PowerShell

# ❌ WRONG - Path with spaces breaks
cd C:\Program Files\My App
C:\Program Files\My App\app.exe

# ✅ CORRECT - Quote the path
cd "C:\Program Files\My App"
& "C:\Program Files\My App\app.exe"

# ✅ CORRECT - With arguments
& "C:\Program Files\My App\app.exe" --config "C:\My Files\config.txt"

# ✅ CORRECT - Calling from CMD
cmd /c "`"C:\Program Files\My App\app.exe`" --arg1 value1"

# ✅ BEST - Use Start-Process for complex scenarios
Start-Process "C:\Program Files\My App\app.exe" -ArgumentList @(
    '--config',
    'C:\My Files\config.txt',
    '--output',
    'C:\My Files\Output'
) -NoNewWindow -Wait
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
docker exec mycontainer bash -c 'cat "/app/My Files/data.txt"'

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

# ✅ BEST - Use here-string for complex commands
$dockerCmd = @'
cd /app
mysql -e "SELECT * FROM users WHERE name='O\"Brien'"
grep "error" "log files/app.log"
'@
docker exec mycontainer bash -c $dockerCmd
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

**From PowerShell:**
```powershell
# ✅ CORRECT - Use single quotes for remote commands
ssh user@remote 'grep "error" /var/log/syslog'

# ✅ CORRECT - Multiple nested levels
ssh user@remote 'docker exec container bash -c "grep \\"error\\" /var/log/app.log"'

# ✅ BEST - Use here-string for complex commands
$remoteScript = @'
#!/bin/bash
docker exec mycontainer bash -c 'mysql -e "SELECT * FROM users WHERE name=\"admin\""'
'@

$remoteScript | Out-File -FilePath /tmp/remote_script.sh -Encoding UTF8
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

# ❌ ANTI-PATTERN 6: Using wrong comparison operators across shells
bash -c "if [ \$count -eq 10 ]; then echo 'yes'; fi"  # Correct for Bash
powershell.exe -Command "if (\$count == 10) { Write-Host 'yes' }"  # WRONG - use -eq

# ❌ ANTI-PATTERN 7: Not using stop-parsing token in PowerShell
git commit -m "Fix O'Brien's "profile" bug"  # BROKEN in PowerShell

# ❌ ANTI-PATTERN 8: Mixing --% with variables
powershell.exe -Command "icacls --% $file /grant:r 'User:(RX)'"  # $file won't expand
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

# ✅ Rule 6: Use here-strings in PowerShell for multi-line content
# See PowerShell examples above

# ✅ Rule 7: Use --% in PowerShell when calling external programs with complex args
# See PowerShell stop-parsing token examples above

# ✅ Rule 8: Know your comparison operators
# Bash: -eq, -ne, -gt, -lt for numbers; =, != for strings
# PowerShell: -eq, -ne, -gt, -lt for everything
# Don't mix them across shells

# ✅ Rule 9: Use proper escaping for paths with spaces
# Always quote: "C:\Program Files\App"
# In PowerShell call operator: & "C:\Program Files\App\app.exe"

# ✅ Rule 10: For Azure CLI, use here-strings for complex JSON/queries
# Avoids quoting hell with JMESPath queries
```

These examples demonstrate the key concepts from the quotefix skill in practical scenarios, with special emphasis on Windows-specific challenges and PowerShell quoting complexities.
