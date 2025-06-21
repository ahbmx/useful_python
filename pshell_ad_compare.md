# Enhanced User Comparison with CSV Export in Constrained Mode

Here's the complete solution that compares multiple user pairs and exports results to a CSV file with the current date as the filename:

```powershell
# Define the user pairs to compare
$userPairs = @(
    @("jdoe", "asmith"),
    @("bobh", "jend"),
    @("user3", "user4")  # Add more pairs here
)

$rootDSE = [ADSI]"LDAP://RootDSE"
$domainPath = "LDAP://$($rootDSE.defaultNamingContext)"  # Update with your domain
$outputFile = "UserComparison_$(Get-Date -Format 'yyyyMMdd').csv"  # CSV filename with date

# Initialize CSV output
$csvData = @()

# Function to get user properties
function Get-UserProperties {
    param($username)
    
    $userProps = @{}
    try {
        $domain = New-Object System.DirectoryServices.DirectoryEntry($domainPath)
        $searcher = New-Object System.DirectoryServices.DirectorySearcher($domain)
        $searcher.Filter = "(&(objectCategory=user)(sAMAccountName=$username))"
        $user = $searcher.FindOne()

        if ($user) {
            $userProps["Name"] = $user.Properties["name"][0]
            $userProps["Login"] = $user.Properties["samaccountname"][0]
            $userProps["Email"] = $user.Properties["mail"][0]
            $userProps["Title"] = $user.Properties["title"][0]
            $userProps["Department"] = $user.Properties["department"][0]
            $userProps["Enabled"] = if ($user.Properties["useraccountcontrol"][0] -band 0x2) { "Disabled" } else { "Enabled" }
            
            # Get group memberships
            $groupSearcher = New-Object System.DirectoryServices.DirectorySearcher($domain)
            $groupSearcher.Filter = "(&(objectCategory=group)(member=$($user.Properties["distinguishedname"][0])))"
            $groups = $groupSearcher.FindAll()
            $userProps["Groups"] = $groups | ForEach-Object { $_.Properties["name"][0] } | Sort-Object
        } else {
            $userProps["Error"] = "User not found"
        }
    } catch {
        $userProps["Error"] = "Error: $_"
    }
    
    return $userProps
}

# Process each pair
foreach ($pair in $userPairs) {
    $user1 = $pair[0]
    $user2 = $pair[1]
    
    # Get properties for both users
    $user1Props = Get-UserProperties $user1
    $user2Props = Get-UserProperties $user2

    # Display comparison header (console output)
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
        # Add error info to CSV
        $csvData += [ordered]@{
            "Comparison" = "$user1 vs $user2"
            "User1" = $user1
            "User2" = $user2
            "Status" = "Error occurred - see console for details"
            "User1_Groups" = ""
            "User2_Groups" = ""
            "Shared_Groups" = ""
            "Unique_to_User1" = ""
            "Unique_to_User2" = ""
        }
        continue
    }

    # Display basic properties comparison (console output)
    Write-Host ("{0,-20} {1,-30} {2,-30}" -f "PROPERTY", $user1.ToUpper(), $user2.ToUpper())
    Write-Host ("{0,-20} {1,-30} {2,-30}" -f "--------", "------------------------------", "------------------------------")

    "Name", "Login", "Email", "Title", "Department", "Enabled" | ForEach-Object {
        $prop = $_
        Write-Host ("{0,-20} {1,-30} {2,-30}" -f $prop, $user1Props[$prop], $user2Props[$prop])
    }

    # Compare groups
    $user1Only = $user1Props["Groups"] | Where-Object { $_ -notin $user2Props["Groups"] }
    $user2Only = $user2Props["Groups"] | Where-Object { $_ -notin $user1Props["Groups"] }
    $commonGroups = $user1Props["Groups"] | Where-Object { $_ -in $user2Props["Groups"] }

    # Console output for groups
    Write-Host ""
    Write-Host "GROUP MEMBERSHIP:"
    Write-Host ("{0,-40} {1,-40}" -f "ONLY IN $($user1.ToUpper())", "ONLY IN $($user2.ToUpper())")
    Write-Host ("{0,-40} {1,-40}" -f "----------------------------------------", "----------------------------------------")

    if ($user1Only.Count -gt $user2Only.Count) {
        $maxLines = $user1Only.Count
    } else {
        $maxLines = $user2Only.Count
    }
    for ($i = 0; $i -lt $maxLines; $i++) {
        $left = if ($i -lt $user1Only.Count) { $user1Only[$i] } else { "" }
        $right = if ($i -lt $user2Only.Count) { $user2Only[$i] } else { "" }
        Write-Host ("{0,-40} {1,-40}" -f $left, $right)
    }

    Write-Host ""
    Write-Host "SHARED GROUPS (both users):"
    if ($commonGroups) {
        $commonGroups | ForEach-Object { Write-Host $_ }
    } else {
        Write-Host "None" -ForegroundColor Yellow
    }

    # Prepare data for CSV export
    $csvEntry = [ordered]@{
        "Comparison" = "$user1 vs $user2"
        "User1" = $user1
        "User2" = $user2
        "User1_Name" = $user1Props["Name"]
        "User2_Name" = $user2Props["Name"]
        "User1_Email" = $user1Props["Email"]
        "User2_Email" = $user2Props["Email"]
        "User1_Title" = $user1Props["Title"]
        "User2_Title" = $user2Props["Title"]
        "User1_Department" = $user1Props["Department"]
        "User2_Department" = $user2Props["Department"]
        "User1_Status" = $user1Props["Enabled"]
        "User2_Status" = $user2Props["Enabled"]
        "User1_Groups" = ($user1Props["Groups"] -join "; ")
        "User2_Groups" = ($user2Props["Groups"] -join "; ")
        "Shared_Groups" = ($commonGroups -join "; ")
        "Unique_to_User1" = ($user1Only -join "; ")
        "Unique_to_User2" = ($user2Only -join "; ")
    }
    $csvData += $csvEntry
}

# Export to CSV
try {
    $csvData | Export-Csv -Path $outputFile -NoTypeInformation -Encoding UTF8
    Write-Host "`nComparison results saved to: $outputFile" -ForegroundColor Green
} catch {
    Write-Host "Error exporting to CSV: $_" -ForegroundColor Red
}

# Display final message
Write-Host "`nComparison process completed. Check both console output and CSV file for results."
```

## Key Features:

1. **CSV Export with Date-Based Filename**:
   - Automatically creates filename like `UserComparison_20230621.csv`
   - Uses current date in YYYYMMDD format

2. **Comprehensive CSV Contents**:
   - All user properties (name, email, title, etc.)
   - Complete group membership information
   - Both shared and unique groups
   - User status (Enabled/Disabled)

3. **Dual Output**:
   - Detailed console output for immediate viewing
   - Structured CSV file for analysis and reporting

4. **Error Handling**:
   - Skips invalid user pairs but records them in CSV
   - Handles CSV export errors gracefully

5. **Constrained Mode Compatible**:
   - Uses only basic PowerShell features
   - No admin rights required

## How to Use:

1. Add your user pairs to the `$userPairs` array
2. Update the `$domainPath` with your domain information
3. Run the script
4. Results will be:
   - Displayed in the console
   - Saved to a CSV file with the current date

The CSV file will be created in your current working directory with all comparison results in a structured format.
