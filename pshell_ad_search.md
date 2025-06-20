If you don't have the `Get-ADUser` cmdlet available (which is part of the Active Directory module), you can still search Active Directory using alternative methods in PowerShell. Here are some approaches:

---

### **Method 1: Using `[ADSISearcher]` (Native .NET, No Modules Required)**
This works without installing any additional modules.

#### **Search by Name (Wildcard Supported)**
```powershell
$searcher = [ADSISearcher]"(|(name=*John*)(displayName=*John*)(givenName=*John*)(sn=*John*))"
$searcher.FindAll() | ForEach-Object {
    [PSCustomObject]@{
        Name = $_.Properties["name"][0]
        SamAccountName = $_.Properties["samaccountname"][0]
        UserPrincipalName = $_.Properties["userprincipalname"][0]
        Email = $_.Properties["mail"][0]
        Enabled = !$_.Properties["useraccountcontrol"][0].Contains("2")  # 2 = Disabled flag
    }
}
```

#### **Search by `sAMAccountName` (Primary Login)**
```powershell
$username = "jdoe"  # Replace with the username you're searching for
$searcher = [ADSISearcher]"(&(objectCategory=user)(samaccountname=$username))"
$result = $searcher.FindOne()

if ($result) {
    [PSCustomObject]@{
        Name = $result.Properties["name"][0]
        SamAccountName = $result.Properties["samaccountname"][0]
        UserPrincipalName = $result.Properties["userprincipalname"][0]
        Email = $result.Properties["mail"][0]
        Enabled = !($result.Properties["useraccountcontrol"][0] -band 0x2)  # Check if disabled
    }
} else {
    Write-Host "User not found."
}
```

---

### **Method 2: Using `System.DirectoryServices.DirectorySearcher` (More Flexible)**
This gives you more control over the search.

```powershell
$searchTerm = "John"  # Name or username to search for
$domain = "LDAP://DC=yourdomain,DC=com"  # Replace with your domain

$searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$domain)
$searcher.Filter = "(&(objectCategory=user)(|(name=*$searchTerm*)(samaccountname=*$searchTerm*)))"

$results = $searcher.FindAll()
foreach ($result in $results) {
    $user = $result.GetDirectoryEntry()
    [PSCustomObject]@{
        Name = $user.Name
        Username = $user.sAMAccountName
        Email = $user.mail
        Enabled = !($user.userAccountControl -band 0x2)  # Check if disabled
    }
}
```

---

### **Method 3: Using `dsquery` (Legacy Command)**
If you're on an older system, you can use `dsquery` (available by default on Windows Server).

```powershell
# Search by name
dsquery user -name "*John*"

# Search by username (sAMAccountName)
dsquery user -samid "jdoe"
```

---

### **Common Issues & Fixes**
1. **"ADSISearcher not found"** → Ensure you're running PowerShell as Administrator.
2. **No results?** → Check if your `LDAP` path is correct (e.g., `LDAP://DC=corp,DC=local`).
3. **Need more properties?** → Modify the search filter to include `mail`, `department`, etc.

Would you like to refine the search further (e.g., by OU, disabled users only, etc.)?
