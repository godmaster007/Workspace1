# Define the combined list of winget applications

# Check for Administrative privileges

if (-not ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Error "This script must be run as Administrator."
    return
}

$packages = @(
    # Original List
    "VideoLAN.VLC",
    "Vysor.Vysor",
    "Microsoft.VisualStudioCode",
    "Microsoft.WindowsTerminal",
    "GitHub.GitHubDesktop",
    "AutoHotkey.AutoHotkey",
    "appcs.Ditto",
    "Dropbox.Dropbox",
    "7zip.7zip",
    
    # Second Batch
    "Microsoft.PowerShell",
    "Python.Python.3",
    "Git.Git",
    "LogMeIn.LastPass",
    "Microsoft.PowerToys",
    "DevToys-app.DevToys",
    "Piriform.CCleaner",
    "TeamViewer.TeamViewer",
    "Oracle.VirtualBox",
    "OBSProject.OBSStudio",
    "Webull.WebullDesktop",
    "Obsidian.Obsidian",
    "GIMP.GIMP",
    "XnSoft.XnViewMP",
    "Zoom.Zoom",
    "Google.GoogleDrive",
    "TransmissionProject.Transmission",
    "Google.Chrome",
    "Mozilla.Firefox",
    "Wieldraaijer.WinDirStat",
    "HardcodedSoftware.dupeGuru",
    
    # Third Batch Additions
    "Google.Antigravity",
    
    # Required for Gemini CLI
    "OpenJS.NodeJS.LTS" 
)

# Set up the log file on the desktop

$logPath = "$env:USERPROFILE\Desktop\Winget_Install_Log.txt"
"--- Winget Installation Log ($(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')) ---" | Out-File $logPath

# ==========================================

# PHASE 1: Winget Packages

# ==========================================

"--------------------------------------------------" | Out-File $logPath -Append
"Phase 1: Winget Applications" | Out-File $logPath -Append
"--------------------------------------------------" | Out-File $logPath -Append

foreach ($pkg in $packages) {
    Write-Host "Installing $pkg..." -ForegroundColor Cyan
    
    $pathBefore = [Environment]::GetEnvironmentVariable("Path", "Machine") + [Environment]::GetEnvironmentVariable("Path", "User")
    $output = winget install -e --id $pkg --accept-package-agreements --accept-source-agreements --silent --force 2>&1
    $exitCode = $LASTEXITCODE
    $pathAfter = [Environment]::GetEnvironmentVariable("Path", "Machine") + [Environment]::GetEnvironmentVariable("Path", "User")

    $pathAdded = ""
    if ($pathBefore -ne $pathAfter) {
        $pathAdded = " [NOTE: Installer added executable to System/User PATH]"
    }

    if ($exitCode -eq 0) {
        "✅ SUCCESS: $pkg$pathAdded" | Out-File $logPath -Append
        Write-Host "  Success!" -ForegroundColor Green
    } else {
        "❌ FAILED:  $pkg (Exit Code: $exitCode)" | Out-File $logPath -Append
        $cleanOutput = ($output | Out-String).Trim() -replace "`n", " " -replace "`r", ""
        "         Reason: $cleanOutput" | Out-File $logPath -Append
        Write-Host "  Failed (Logged to desktop)" -ForegroundColor Red
    }
}

# ==========================================

# PHASE 2: Extensions & Terminal Tools

# ==========================================

"--------------------------------------------------" | Out-File $logPath -Append
"Phase 2: IDE Extensions & NPM Packages" | Out-File $logPath -Append
"--------------------------------------------------" | Out-File $logPath -Append

# Force PowerShell to reload the PATH variables mid-script so it recognizes 'code' and 'npm'

$env:Path = [System.Environment]::GetEnvironmentVariable("Path", "Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path", "User")

# Install VS Code Extension: Cloud Code (Includes Gemini Code Assist)

Write-Host "`nInstalling VS Code Extension: Cloud Code (Gemini Code Assist)..." -ForegroundColor Cyan

if (Get-Command code -ErrorAction SilentlyContinue) {
    $vscodeOutput = code --install-extension GoogleCloudTools.cloudcode 2>&1
    if ($LASTEXITCODE -eq 0) {
        "✅ SUCCESS: VS Code Extension - Cloud Code (Gemini Code Assist)" | Out-File $logPath -Append
        Write-Host "  Success!" -ForegroundColor Green
    } else {
        "❌ FAILED:  VS Code Extension - Cloud Code" | Out-File $logPath -Append
        $cleanVSOutput = ($vscodeOutput | Out-String).Trim() -replace "`n", " "
        "         Reason: $cleanVSOutput" | Out-File $logPath -Append
        Write-Host "  Failed (Logged to desktop)" -ForegroundColor Red
    }
} else {
    "⚠️ SKIPPED: Cloud Code Extension (VS Code 'code' command not found in PATH)" | Out-File $logPath -Append
    Write-Host "  Skipped: VS Code CLI not available." -ForegroundColor Yellow
}

# Install Gemini CLI via NPM

Write-Host "`nInstalling terminal tool: Gemini CLI..." -ForegroundColor Cyan
if (Get-Command npm -ErrorAction SilentlyContinue) {
    try {
        $npmOutput = npm install -g @google/gemini-cli 2>&1
        if ($LASTEXITCODE -eq 0) {
            "✅ SUCCESS: NPM Package - @google/gemini-cli" | Out-File $logPath -Append
            Write-Host "  Success!" -ForegroundColor Green
        } else {
            throw $npmOutput
        }
    } catch {
        "❌ FAILED:  NPM Package - @google/gemini-cli" | Out-File $logPath -Append
        $cleanNpmOutput = ($_.Exception.Message | Out-String).Trim() -replace "`n", " "
        "         Reason: $cleanNpmOutput" | Out-File $logPath -Append
        Write-Host "  Failed (Logged to desktop)" -ForegroundColor Red
    }
} else {
    "⚠️ SKIPPED: Gemini CLI (npm command not found. Node.js installation may require a shell restart)" | Out-File $logPath -Append
    Write-Host "  Skipped: NPM not found. Node.js installation may have failed." -ForegroundColor Yellow
}

"--------------------------------------------------" | Out-File $logPath -Append
"Installation sequence complete." | Out-File $logPath -Append
Write-Host "`nAll done! Check Winget_Install_Log.txt on your Desktop for the full report." -ForegroundColor Yellow