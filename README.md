# AD-Enumeration
Used for CSC125 

# ==========================================
# Active Directory Enumeration Script
# Assignment 8.3
# ==========================================

# Import Active Directory module
Import-Module ActiveDirectory

# Create output directory
$OutputDir = "C:\AD_Enumeration_Results"
if (!(Test-Path $OutputDir)) {
    New-Item -ItemType Directory -Path $OutputDir
}

Write-Host "Starting Active Directory Enumeration..." -ForegroundColor Green

# ==========================================
# 1. Enumerate User Accounts
# ==========================================
Write-Host "Collecting user accounts..." -ForegroundColor Yellow

$Users = Get-ADUser -Filter * -Property Name, SamAccountName, Enabled, LastLogonDate

$Users | Select-Object Name, SamAccountName, Enabled, LastLogonDate |
    Export-Csv "$OutputDir\Users.csv" -NoTypeInformation

# ==========================================
# 2. Enumerate Group Memberships
# ==========================================
Write-Host "Collecting group memberships..." -ForegroundColor Yellow

$Groups = Get-ADGroup -Filter *

$GroupMemberships = foreach ($group in $Groups) {
    Get-ADGroupMember -Identity $group.Name | Select-Object @{
        Name="GroupName"; Expression = {$group.Name}
    }, Name, objectClass
}

$GroupMemberships |

    Export-Csv "$OutputDir\GroupMemberships.csv" -NoTypeInformation

# ==========================================
# 3. Enumerate GPOs (Group Policy Objects)
# ==========================================
Write-Host "Collecting GPOs..." -ForegroundColor Yellow

Import-Module GroupPolicy

$GPOs = Get-GPO -All

$GPOs | Select-Object DisplayName, Id, CreationTime, ModificationTime |
    Export-Csv "$OutputDir\GPOs.csv" -NoTypeInformation

# ==========================================
# 4. SID Lookup Example
# ==========================================
Write-Host "Performing SID lookup..." -ForegroundColor Yellow

# Example: Lookup SID for Administrator account
$User = Get-ADUser -Identity "Administrator"

$SID = $User.SID.Value

# Convert SID back to name
$SIDObject = New-Object System.Security.Principal.SecurityIdentifier($SID)
$Account = $SIDObject.Translate([System.Security.Principal.NTAccount])

$SIDResult = [PSCustomObject]@{
    Username = $User.SamAccountName
    SID      = $SID
    ResolvedName = $Account.Value
}

$SIDResult | Export-Csv "$OutputDir\SID_Lookup.csv" -NoTypeInformation

# ==========================================
# Summary Output
# ==========================================
Write-Host "Enumeration Complete!" -ForegroundColor Green
Write-Host "Results saved to: $OutputDir" -ForegroundColor Cyan
