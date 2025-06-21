# Final Fixed Script Without Null-Coalescing Operator

The error occurs because the `??` null-coalescing operator isn't available in older PowerShell versions or constrained mode. Here's the complete fixed script using compatible syntax:

```powershell
# User Comparison Script for Constrained Mode
# Compares multiple AD user pairs and exports to CSV

# Define user pairs to compare
$userPairs = @(
    @("jdoe", "asmith"),
    @("bobh", "jend"),
    @("user3", "user4")  # Add more pairs as needed
)

# Get domain information (works in constrained mode)
try {
    $domainInfo = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
    $domainPath = "LDAP://$($domainInfo.Name)"
} catch {
    # Fallback to environment variable method
    $domainName = $env:USERDNSDOMAIN
    $domainPath = "LDAP://$domainName"
}

# Initialize data collection
$csvData = @()

# Function to get user properties (constrained mode compatible)
function Get-UserProperties {
    param($username)
    
    $userProps = @{}
    try {
        $domain = New-Object System.DirectoryServices.DirectoryEntry($domainPath)
        $searcher = New-Object System.DirectoryServices.DirectorySearcher($domain)
        $searcher.Filter = "(&(objectCategory=user)(sAMAccountName=$username))"
        $user = $searcher.FindOne()

        if ($user) {
            $userProps["Name"] = if ($user.Properties["name"][0]) { $user.Properties["name"][0] } else { "" }
            $userProps["Login"] = if ($user.Properties["samaccountname"][0]) { $user.Properties["samaccountname"][0] } else { "" }
            $userProps["Email"] = if ($user.Properties["mail"][0]) { $user.Properties["mail"][0] } else { "" }
            $userProps["Title"] = if ($user.Properties["title"][0]) { $user.Properties["title"][0] } else { "" }
            $userProps["Department"] = if ($user.Properties["department"][0]) { $user.Properties["department"][0] } else { "" }
            $userProps["Enabled"] = if ($user.Properties["useraccountcontrol"][0] -band 0x2) { "Disabled" } else { "Enabled" }
            
            # Get group memberships
            $groupSearcher = New-Object System.DirectoryServices.DirectorySearcher($domain)
            $groupSearcher.Filter = "(&(objectCategory=group)(member=$($user.Properties["distinguishedname"][0])))"
            $groups = $groupSearcher.FindAll()
            $userProps["Groups"] = $groups | ForEach-Object { if ($_.Properties["name"][0]) { $_.Properties["name"][0] } else { "" } } | Sort-Object
        } else {
            $userProps["Error"] = "User not found"
        }
    } catch {
        $userProps["Error"] = "Error: $_"
    }
    
    return $userProps
}

# Process each user pair
foreach ($pair in $userPairs) {
    $user1 = $pair[0]
    $user2 = $pair[1]
    
    # Get properties for both users
    $user1Props = Get-UserProperties $user1
    $user2Props = Get-UserProperties $user2

    # Display comparison header
    Write-Host ""
    Write-Host ("=" * 80)
    Write-Host ("COMPARISON: {0} vs {1}" -f $user1.ToUpper(), $user2.ToUpper())
    Write-Host ("=" * 80)
    Write-Host ""

    # Check for errors
    $errorOccurred = $false
    if ($user1Props["Error"]) {
        Write-Host ("Error with {0}: {1}" -f $user1, $user1Props["Error"]) -ForegroundColor Red
        $errorOccurred = $true
    }
    if ($user2Props["Error"]) {
        Write-Host ("Error with {0}: {1}" -f $user2, $user2Props["Error"]) -ForegroundColor Red
        $errorOccurred = $true
    }
    if ($errorOccurred) {
        # Add error info to CSV data
        $csvData += @{
            "Comparison" = "$user1 vs $user2"
            "User1" = $user1
            "User2" = $user2
            "Status" = "Error occurred"
            "User1_Name" = ""
            "User2_Name" = ""
            "User1_Email" = ""
            "User2_Email" = ""
            "User1_Title" = ""
            "User2_Title" = ""
            "User1_Department" = ""
            "User2_Department" = ""
            "User1_Status" = ""
            "User2_Status" = ""
            "Shared_Groups" = ""
            "Unique_to_User1" = ""
            "Unique_to_User2" = ""
        }
        continue
    }

    # Display basic properties comparison
    Write-Host ("{0,-20} {1,-30} {2,-30}" -f "PROPERTY", $user1.ToUpper(), $user2.ToUpper())
    Write-Host ("{0,-20} {1,-30} {2,-30}" -f "--------", "------------------------------", "------------------------------")

    "Name", "Login", "Email", "Title", "Department", "Enabled" | ForEach-Object {
        $prop = $_
        $value1 = if ($user1Props[$prop]) { $user1Props[$prop] } else { "" }
        $value2 = if ($user2Props[$prop]) { $user2Props[$prop] } else { "" }
        Write-Host ("{0,-20} {1,-30} {2,-30}" -f $prop, $value1, $value2)
    }

    # Compare groups
    $user1Only = if ($user1Props["Groups"]) { $user1Props["Groups"] | Where-Object { $_ -notin $user2Props["Groups"] } } else { @() }
    $user2Only = if ($user2Props["Groups"]) { $user2Props["Groups"] | Where-Object { $_ -notin $user1Props["Groups"] } } else { @() }
    $commonGroups = if ($user1Props["Groups"] -and $user2Props["Groups"]) { $user1Props["Groups"] | Where-Object { $_ -in $user2Props["Groups"] } } else { @() }

    # Display group comparison
    Write-Host ""
    Write-Host "GROUP MEMBERSHIP:"
    Write-Host ("{0,-40} {1,-40}" -f "ONLY IN $($user1.ToUpper())", "ONLY IN $($user2.ToUpper())")
    Write-Host ("{0,-40} {1,-40}" -f "----------------------------------------", "----------------------------------------")

    # Determine max lines without [Math]::Max
    $maxLines = if ($user1Only.Count -gt $user2Only.Count) { $user1Only.Count } else { $user2Only.Count }

    # Display side-by-side
    for ($i = 0; $i -lt $maxLines; $i++) {
        $left = if ($i -lt $user1Only.Count) { $user1Only[$i] } else { "" }
        $right = if ($i -lt $user2Only.Count) { $user2Only[$i] } else { "" }
        Write-Host ("{0,-40} {1,-40}" -f $left, $right)
    }

    # Show common groups
    Write-Host ""
    Write-Host "SHARED GROUPS (both users):"
    if ($commonGroups) {
        $commonGroups | ForEach-Object { Write-Host $_ }
    } else {
        Write-Host "None" -ForegroundColor Yellow
    }

    # Add to CSV data
    $csvData += @{
        "Comparison" = "$user1 vs $user2"
        "User1" = $user1
        "User2" = $user2
        "User1_Name" = if ($user1Props["Name"]) { $user1Props["Name"] } else { "" }
        "User2_Name" = if ($user2Props["Name"]) { $user2Props["Name"] } else { "" }
        "User1_Email" = if ($user1Props["Email"]) { $user1Props["Email"] } else { "" }
        "User2_Email" = if ($user2Props["Email"]) { $user2Props["Email"] } else { "" }
        "User1_Title" = if ($user1Props["Title"]) { $user1Props["Title"] } else { "" }
        "User2_Title" = if ($user2Props["Title"]) { $user2Props["Title"] } else { "" }
        "User1_Department" = if ($user1Props["Department"]) { $user1Props["Department"] } else { "" }
        "User2_Department" = if ($user2Props["Department"]) { $user2Props["Department"] } else { "" }
        "User1_Status" = if ($user1Props["Enabled"]) { $user1Props["Enabled"] } else { "" }
        "User2_Status" = if ($user2Props["Enabled"]) { $user2Props["Enabled"] } else { "" }
        "Shared_Groups" = if ($commonGroups) { ($commonGroups -join "; ") } else { "" }
        "Unique_to_User1" = if ($user1Only) { ($user1Only -join "; ") } else { "" }
        "Unique_to_User2" = if ($user2Only) { ($user2Only -join "; ") } else { "" }
    }
}

# ===== CSV EXPORT =====
$outputFile = "UserComparison_$(Get-Date -Format 'yyyyMMdd').csv"

# Build the header line
$headerLine = "Comparison,User1,User2,User1_Name,User2_Name,User1_Email,User2_Email,User1_Title,User2_Title,User1_Department,User2_Department,User1_Status,User2_Status,Shared_Groups,User1_Unique_Groups,User2_Unique_Groups"

# Initialize the CSV content with header
$csvContent = @()
$csvContent += $headerLine

# Process each comparison entry
foreach ($entry in $csvData) {
    # Escape any quotes in the data and handle nulls
    $comparison       = if ($entry["Comparison"]) { $entry["Comparison"].ToString().Replace('"', '""') } else { "" }
    $user1            = if ($entry["User1"]) { $entry["User1"].ToString().Replace('"', '""') } else { "" }
    $user2            = if ($entry["User2"]) { $entry["User2"].ToString().Replace('"', '""') } else { "" }
    $user1Name        = if ($entry["User1_Name"]) { $entry["User1_Name"].ToString().Replace('"', '""') } else { "" }
    $user2Name        = if ($entry["User2_Name"]) { $entry["User2_Name"].ToString().Replace('"', '""') } else { "" }
    $user1Email       = if ($entry["User1_Email"]) { $entry["User1_Email"].ToString().Replace('"', '""') } else { "" }
    $user2Email       = if ($entry["User2_Email"]) { $entry["User2_Email"].ToString().Replace('"', '""') } else { "" }
    $user1Title       = if ($entry["User1_Title"]) { $entry["User1_Title"].ToString().Replace('"', '""') } else { "" }
    $user2Title       = if ($entry["User2_Title"]) { $entry["User2_Title"].ToString().Replace('"', '""') } else { "" }
    $user1Department  = if ($entry["User1_Department"]) { $entry["User1_Department"].ToString().Replace('"', '""') } else { "" }
    $user2Department  = if ($entry["User2_Department"]) { $entry["User2_Department"].ToString().Replace('"', '""') } else { "" }
    $user1Status      = if ($entry["User1_Status"]) { $entry["User1_Status"].ToString().Replace('"', '""') } else { "" }
    $user2Status      = if ($entry["User2_Status"]) { $entry["User2_Status"].ToString().Replace('"', '""') } else { "" }
    $sharedGroups     = if ($entry["Shared_Groups"]) { $entry["Shared_Groups"].ToString().Replace('"', '""') } else { "" }
    $user1Unique      = if ($entry["Unique_to_User1"]) { $entry["Unique_to_User1"].ToString().Replace('"', '""') } else { "" }
    $user2Unique      = if ($entry["Unique_to_User2"]) { $entry["Unique_to_User2"].ToString().Replace('"', '""') } else { "" }
    
    # Build the data line with proper CSV formatting
    $dataLine = """$comparison"",""$user1"",""$user2"",""$user1Name"",""$user2Name"",""$user1Email"",""$user2Email"",""$user1Title"",""$user2Title"",""$user1Department"",""$user2Department"",""$user1Status"",""$user2Status"",""$sharedGroups"",""$user1Unique"",""$user2Unique"""
    
    $csvContent += $dataLine
}

# Join all lines with newlines and save to file
try {
    $csvContent -join "`r`n" | Out-File -FilePath $outputFile -Encoding UTF8
    Write-Host "`nCSV export successful: $outputFile" -ForegroundColor Green
    
    # Verify output
    Write-Host "First 3 lines of CSV file:"
    Get-Content $outputFile | Select-Object -First 3
} catch {
    Write-Host "CSV export failed: $_" -ForegroundColor Red
}

Write-Host "`nComparison process completed." -ForegroundColor Cyan
```

Key Changes Made:
1. Replaced all `??` operators with traditional `if` statements
2. Added proper null checking throughout the script
3. Ensured all values are converted to strings before CSV export
4. Maintained all functionality while being compatible with constrained mode
5. Added more robust error handling for null values

The script will now:
1. Work in constrained language mode
2. Handle null values properly
3. Export complete CSV data with headers
4. Show comparison results in console
5. Create dated output files
