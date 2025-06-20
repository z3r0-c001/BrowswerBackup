# Multi-Browser Bookmark Backup for WSL

![Python](https://img.shields.io/badge/python-3.8+-blue.svg)
![Platform](https://img.shields.io/badge/platform-WSL-green.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

A Python script to automatically backup browser bookmarks from WSL (Windows Subsystem for Linux) to a configurable location with scheduling support.

**Supported Browsers:** Chrome, Edge, Brave, and custom browser paths

## 🚀 Features

- **Multi-browser support** - Chrome, Edge, Brave, and custom browser paths
- **Cross-platform bookmark detection** - Automatically finds browser bookmarks on Windows from WSL
- **Automatic first-run setup** - Prompts for browser, user and backup directory on first use
- **Smart configuration** - Suggests OneDrive folder for automatic cloud backup
- **Multiple scheduling options** - Works around WSL cron limitations
- **Persistent configuration** - Saves settings for future runs
- **Backup rotation** - Keeps only the most recent N backups
- **Multiple profile support** - Backs up all browser profiles
- **Permission handling** - Gracefully handles WSL permission issues
- **Comprehensive diagnostics** - Built-in diagnostic tool for troubleshooting

## 📋 Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Scheduling Options](#scheduling-options)
- [Usage Examples](#usage-examples)
- [Troubleshooting](#troubleshooting)
- [File Structure](#file-structure)
- [Best Practices](#best-practices)
- [Contributing](#contributing)
- [License](#license)
- [Support](#support)

## Installation

### Prerequisites

- Windows 10/11 with WSL enabled
- Python 3.8 or higher
- Supported browser installed on Windows (Chrome, Edge, Brave, or custom Chromium-based browser)
- Access to Windows user directories from WSL

### Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/z3r0-c001/BrowserBackup.git
   cd BrowserBackup
   ```

   **Or download individual files:**
   ```bash
   # Download main backup script
   wget https://raw.githubusercontent.com/z3r0-c001/BrowserBackup/main/bookmarkBackup.py
   
   # Download diagnostic script (optional)
   wget https://raw.githubusercontent.com/z3r0-c001/BrowserBackup/main/browser_diagnostic.py
   ```

2. **Make executable:**
   ```bash
   chmod +x bookmarkBackup.py
   ```

3. **First run (auto-setup):**
   ```bash
   python3 bookmarkBackup.py -l
   ```
   *The script will automatically prompt for configuration on first use.*

## Quick Start

### First Time Setup (Automatic)

The script automatically prompts for configuration on first run:

```bash
# Simply run any command - setup happens automatically
python3 bookmarkBackup.py -l
```

**The script will prompt you for:**
1. **Browser Selection** - Choose Chrome, Edge, Brave, or specify custom path
2. **Backup Directory** - Suggests OneDrive location, allows custom path
3. **Windows User** - Shows available users, prompts for selection
4. **Backup Retention** - Choose how many backups to keep (1, 5, 10, 30, or custom)

**Example first run:**
```bash
$ python3 bookmarkBackup.py -l

=== First Time Setup: Backup Directory ===
Please specify where to store your bookmark backups.
Suggested location: /mnt/c/Users/user1/OneDrive/bookmarks

Use suggested location? (y/n): y
✓ Backup directory saved: /mnt/c/Users/user1/OneDrive/bookmarks

=== Backup Retention Setup ===
How many backup files should be kept?
  1. Keep 1 backup (only most recent)
  2. Keep 5 backups
  3. Keep 10 backups
  4. Keep 30 backups (recommended)
  5. Custom number

Select retention option (1-5): 4
✓ Backup retention set to: 30 backups

=== Browser Selection ===
Select the browser you want to backup:
  1. Google Chrome
  2. Microsoft Edge
  3. Brave Browser
  4. Custom browser (specify path)

Select browser (1-4): 1
✓ Browser 'Google Chrome' configured for backups.

=== Windows User Selection ===
Available Windows user accounts:
  1. user1
  2. user2
  3. WsiAccount

Select user account (1-3): 1
✓ User 'user1' saved for future backups.
```

### Basic Usage (After Setup)

```bash
# Perform backup with verbose output
python3 bookmarkBackup.py -v

# Test run (dry run)
python3 bookmarkBackup.py -t

# List available browser profiles
python3 bookmarkBackup.py -l
```

## Configuration

### Automatic Configuration

The script automatically prompts for configuration on first run:
- **Browser Selection** - Choose from Chrome, Edge, Brave, or custom path
- **Backup Directory** - Suggests `/mnt/c/Users/username/OneDrive/bookmarks`
- **Windows User** - Lists accessible Windows users for selection
- **Backup Retention** - Choose how many backups to keep (1, 5, 10, 30, or custom)

### Manual Configuration Commands

| Command | Description |
|---------|-------------|
| `--setup` | Force complete setup (optional - happens automatically) |
| `--show-config` | Display current configuration |
| `--configure-browser` | Change browser selection |
| `--configure-user` | Change Windows user selection |
| `--configure-backup` | Change backup directory |
| `--configure-retention` | Change backup retention count |
| `--configure` | Reconfigure browser, user, backup directory and retention |

### Configuration File

Settings are stored in `~/.browser_backup_config.json`:

```json
{
  "browser": "chrome",
  "windows_user": "user1",
  "backup_path": "/mnt/c/Users/user1/OneDrive/bookmarks",
  "max_backups": 30
}
```

**For custom browsers:**
```json
{
  "browser": {"custom": "/path/to/browser/data"},
  "windows_user": "user1",
  "backup_path": "/mnt/c/Users/user1/OneDrive/bookmarks",
  "max_backups": 10
}
```

### Command Line Options

```bash
python3 bookmarkBackup.py [options] [backup_path]

Options:
  -v, --verbose             Verbose output
  -l, --list               List available profiles
  -t, --test               Test mode (dry run)
  -m, --max-backups N      Maximum backups to keep (overrides config)
  --setup                  Run initial setup
  --show-config            Show current configuration
  --configure-browser      Configure browser selection only
  --configure-user         Configure Windows user only
  --configure-backup       Configure backup directory only
  --configure-retention    Configure backup retention only
  --configure              Configure browser, user, backup directory and retention
```

## Scheduling Options

Since cron doesn't work reliably in WSL, here are several scheduling alternatives:

### Option 1: Windows Task Scheduler (⭐ Recommended)

1. **Create batch file (`brave_backup.bat`):**
   ```batch
   @echo off
   wsl -d Ubuntu python3 /mnt/c/path/to/bookmarkBackup.py -v >> C:\logs\brave_backup.log 2>&1
   ```

2. **Setup Task Scheduler:**
   - Open Task Scheduler (`Win + R` → `taskschd.msc`)
   - Create Basic Task → "Brave Bookmark Backup"
   - Set trigger (Daily, Weekly, etc.)
   - Action: Start program → Point to your `.bat` file
   - Enable "Run whether user is logged on or not"

### Option 2: PowerShell Scheduled Job

**Create PowerShell script (`BraveBackup.ps1`):**
```powershell
$logFile = "C:\logs\brave_backup.log"
$timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"

try {
    $result = wsl python3 /mnt/c/path/to/bookmarkBackup.py -v 2>&1
    Add-Content -Path $logFile -Value "[$timestamp] SUCCESS: $result"
    
    # Optional: Email notification on success
    # Send-MailMessage -To "you@email.com" -Subject "Backup Success" -Body $result
} catch {
    Add-Content -Path $logFile -Value "[$timestamp] ERROR: $($_.Exception.Message)"
    
    # Optional: Email notification on failure
    # Send-MailMessage -To "you@email.com" -Subject "Backup Failed" -Body $_.Exception.Message
}
```

**Register scheduled task:**
```powershell
$trigger = New-ScheduledTaskTrigger -Daily -At 2:00AM
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-File C:\Scripts\BraveBackup.ps1"
$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries
Register-ScheduledTask -TaskName "BraveBookmarkBackup" -Trigger $trigger -Action $action -Settings $settings
```

### Option 3: systemd (WSL2 with systemd enabled)

**Service file (`~/.config/systemd/user/brave-backup.service`):**
```ini
[Unit]
Description=Brave Bookmark Backup
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/bin/python3 /home/username/scripts/bookmarkBackup.py -v
User=%i
```

**Timer file (`~/.config/systemd/user/brave-backup.timer`):**
```ini
[Unit]
Description=Run Brave Bookmark Backup Daily
Requires=brave-backup.service

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

**Enable timer:**
```bash
systemctl --user daemon-reload
systemctl --user enable brave-backup.timer
systemctl --user start brave-backup.timer

# Check status
systemctl --user list-timers
```

### Option 4: Startup Script

**For backup on login (`brave_backup_startup.bat`):**
```batch
@echo off
timeout /t 30 /nobreak >nul
wsl python3 /mnt/c/path/to/bookmarkBackup.py -v
```

Place in Windows startup folder: `Win + R` → `shell:startup`

## Usage Examples

### Basic Operations

```bash
# First run - automatic setup prompts will appear
python3 bookmarkBackup.py -l

# Show current configuration
python3 bookmarkBackup.py --show-config

# Test backup (no files created)
python3 bookmarkBackup.py -t

# Perform backup with detailed output
python3 bookmarkBackup.py -v

# Run comprehensive system diagnostic
python3 browser_diagnostic.py

# Backup to specific location (overrides saved config)
python3 bookmarkBackup.py -v /mnt/c/MyBackups/browsers

# Keep only 10 most recent backups
python3 bookmarkBackup.py -v -m 10
```

### Configuration Changes

```bash
# Change browser
python3 bookmarkBackup.py --configure-browser

# Change Windows user
python3 bookmarkBackup.py --configure-user

# Change backup directory
python3 bookmarkBackup.py --configure-backup

# Change backup retention
python3 bookmarkBackup.py --configure-retention

# Force complete reconfiguration (optional)
python3 bookmarkBackup.py --setup
```

### Monitoring and Verification

```bash
# List recent backups (files now include browser name)
ls -la /mnt/c/Users/user1/OneDrive/bookmarks/*_bookmarks_*.json

# Examples of backup files:
# chrome_bookmarks_default_20241215_143022.json
# edge_bookmarks_profile_1_20241215_143025.json
# brave_bookmarks_default_20241215_143028.json

# Check backup file integrity
python3 -c "
import json, sys
with open(sys.argv[1], 'r') as f:
    data = json.load(f)
    print(f'Backup contains {len(data.get(\"roots\", {}).get(\"bookmark_bar\", {}).get(\"children\", []))} bookmarks')
" /path/to/backup/file.json

# Verify OneDrive sync status (if using OneDrive)
```

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| **Permission denied** | Ensure WSL has access to Windows user directories |
| **No setup prompts appear** | Delete `~/.browser_backup_config.json` and run again |
| **No browser profiles found** | Run `--configure-browser` to select correct browser |
| **Browser not detected** | Try `--configure-browser` and select custom path |
| **No Windows user found** | Run `--configure-user` to select correct Windows user |
| **Backup directory not accessible** | Run `--configure-backup` to set valid path |
| **Task Scheduler not running** | Check Task Scheduler history and enable logging |
| **WSL not starting** | Restart WSL: `wsl --shutdown` then `wsl` |
| **Setup prompts skipped** | Configuration exists, use `--setup` to force reconfiguration |
| **General issues** | Run `python3 browser_diagnostic.py` for comprehensive diagnosis |

### Debug Commands

```bash
# Test WSL access to Windows
wsl whoami
wsl ls /mnt/c/Users

# Test Python script
python3 bookmarkBackup.py --show-config

# Run comprehensive diagnostic
python3 browser_diagnostic.py

# Test Windows Task Scheduler
schtasks /run /tn "BraveBookmarkBackup"

# Check WSL version
wsl --list --verbose
```

### Log Analysis

```bash
# View recent backup logs (if using batch file logging)
tail -f /mnt/c/logs/browser_backup.log

# Check Windows Event Logs (PowerShell as Administrator)
Get-EventLog -LogName Application -Source "BrowserBackup" -Newest 10
```

## File Structure

```
BrowserBackup/
├── bookmarkBackup.py          # Main backup script
├── browser_diagnostic.py      # Multi-browser diagnostic tool  
├── README.md                  # This documentation
├── LICENSE                    # MIT License file
├── examples/
│   ├── brave_backup.bat       # Windows batch file example
│   ├── BraveBackup.ps1        # PowerShell script example
│   └── brave-backup.service   # systemd service example
└── logs/
    └── brave_backup.log        # Default log file location
```

## Best Practices

### Recommended Setup

For most users in WSL, I recommend:
1. **Use OneDrive backup location** for automatic cloud sync
2. **Schedule daily backups** at 2:00 AM when computer is likely idle  
3. **Keep 30 days of backups** (default) for recovery options
4. **Enable logging** to monitor backup success
5. **Test scheduling** before relying on automation

For detailed setup instructions, see the [Scheduling Options](#scheduling-options) section.

### Security Considerations

- **File permissions**: Ensure backup directory has appropriate permissions
- **Network paths**: Use encrypted connections for network backups
- **Sensitive bookmarks**: Consider encrypting backup files if they contain sensitive data
- **Access logs**: Monitor backup logs for unauthorized access

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

### Development Setup

```bash
# Clone repository
git clone https://github.com/z3r0-c001/BrowserBackup.git
cd BrowserBackup

# Test installation
python3 bookmarkBackup.py --setup

# Run diagnostic
python3 browser_diagnostic.py
```

### Testing

Before submitting changes:

1. Test on WSL with different Windows users
2. Verify configuration prompts work correctly
3. Test all scheduling options
4. Ensure error handling works properly
5. Update documentation as needed

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Google, Microsoft, and Brave Software for their excellent browsers
- Microsoft for WSL integration
- Python community for excellent libraries
- Chromium project for consistent bookmark format across browsers

---

**⭐ If this project helped you, please consider giving it a star!**

## Support

If you encounter any issues:

1. Check the [Troubleshooting](#troubleshooting) section
2. Run the diagnostic script: `python3 browser_diagnostic.py`
3. Open an issue with detailed error information
4. Include your WSL version, Python version, and browser type

---

*Made with ❤️ for the WSL and browser community*