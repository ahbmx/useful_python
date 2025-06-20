Here's an enhanced batch script that synchronizes **multiple folder pairs** (including network shares with drive letters). It first checks if each drive is mounted before syncing:

### **Multi-Folder Sync Script**
```batch
@echo off
setlocal enabledelayedexpansion

:: Configuration - Define your folder pairs here
:: Format: "SourcePath", "DestinationPath", "DriveLetterToCheck" (optional)
set FOLDER_PAIRS=(
    "Z:\Projects", "D:\Backup\Projects", "Z:"
    "\\Server\Data", "E:\Archive\Data", ""
    "X:\Media", "C:\User\Media", "X:"
)

:: Log file
set LOG_FILE="C:\Logs\Sync_%date:~-4,4%%date:~-10,2%%date:~-7,2%.log"

:: Create log header
echo ===== Folder Sync Started at %date% %time% ===== > %LOG_FILE%
echo. >> %LOG_FILE%

:: Loop through each folder pair
for /f "tokens=1-3 delims=," %%A in ('echo %FOLDER_PAIRS% ^| find /v ""') do (
    set "SOURCE=%%~A"
    set "DEST=%%~B"
    set "DRIVE=%%~C"

    echo Processing: !SOURCE! → !DEST!

    :: Check drive mount (if specified)
    if not "!DRIVE!"=="" (
        echo Checking drive !DRIVE!...
        if not exist "!DRIVE!\*" (
            echo ERROR: Drive !DRIVE! not mounted. Skipping. >> %LOG_FILE%
            echo ERROR: Drive !DRIVE! not mounted. Skipping.
            echo. >> %LOG_FILE%
            goto :NEXT_PAIR
        )
    )

    :: Create destination folder if missing
    if not exist "!DEST!\" (
        echo Creating folder: !DEST!
        mkdir "!DEST!"
    )

    :: Sync with Robocopy
    echo Syncing: !SOURCE! → !DEST!... | tee -a %LOG_FILE%
    robocopy "!SOURCE!" "!DEST!" /MIR /ZB /R:3 /W:5 /V /TEE /LOG+:%LOG_FILE% /NP

    :: Check Robocopy result
    if !errorlevel! LEQ 7 (
        echo SUCCESS: Sync completed. | tee -a %LOG_FILE%
    ) else (
        echo WARNING: Sync had issues (Error: !errorlevel!). | tee -a %LOG_FILE%
    )
    echo. >> %LOG_FILE%

    :NEXT_PAIR
)

echo ===== Sync Finished at %date% %time% ===== >> %LOG_FILE%
echo All operations logged to %LOG_FILE%
pause
```

---

### **Key Features**
1. **Multiple Folder Pairs**  
   - Edit `FOLDER_PAIRS` to add your source/destination paths.
   - Optional: Specify a drive letter to check (e.g., `"Z:"` for network drives).

2. **Drive Mount Check**  
   - Skips sync if a required drive is not mounted.

3. **Auto-Create Missing Folders**  
   - Creates destination folders if they don’t exist.

4. **Robocopy with Logging**  
   - Uses `/MIR` (mirror mode) to sync deletions.
   - Logs output to a dated file (`Sync_YYYYMMDD.log`).

5. **Error Handling**  
   - Checks `robocopy` exit codes (0-7 = success).

---

### **How to Use**
1. **Edit `FOLDER_PAIRS`**  
   Add your source/destination paths in the format:  
   `"Source", "Destination", "DriveToCheck"`  
   (Leave `DriveToCheck` blank for local folders.)

2. **Run as Administrator**  
   Required for network drive access.

3. **Check the Log**  
   Output is saved to `C:\Logs\Sync_YYYYMMDD.log`.

---

### **Example Output**
```
===== Folder Sync Started at 06/20/2025 14:30:00 =====

Processing: Z:\Projects → D:\Backup\Projects
Checking drive Z:...
Syncing: Z:\Projects → D:\Backup\Projects...
SUCCESS: Sync completed.

Processing: \\Server\Data → E:\Archive\Data
Syncing: \\Server\Data → E:\Archive\Data...
SUCCESS: Sync completed.

===== Sync Finished at 06/20/2025 14:35:12 =====
```

---

### **Notes**
- **Robocopy Flags:** Adjust `/MIR`, `/ZB`, etc., as needed.
- **Run in Task Scheduler:** Remove `pause` and schedule for automated syncs.
- **For UNC Paths:** Use `net use` if authentication is needed.

Let me know if you'd like modifications! 🚀
