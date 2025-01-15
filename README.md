# PowerShell script for listing user permissions
Here is a PowerShell script to list all Distribution Lists (DLs), Dynamic Distribution Lists (DDLs), and Microsoft 365 groups for which a user has 'Send As' or 'Send on Behalf' permissions:

```powershell
# Connect to Exchange Online
Connect-ExchangeOnline -UserPrincipalName <yourUser@domain.com> -ShowProgress $true

# Function to get Send As permissions
function Get-SendAsPermissions {
    param (
        [string]$User
    )
    Get-RecipientPermission -Trustee $User | Where-Object { $_.AccessRights -contains "SendAs" }
}

# Function to get Send on Behalf permissions
function Get-SendOnBehalfPermissions {
    param (
        [string]$User
    )
    Get-Mailbox -ResultSize Unlimited | ForEach-Object {
        $mailbox = $_
        if ($mailbox.DelegateList -contains $User) {
            [PSCustomObject]@{
                Identity = $mailbox.Identity
                Delegate = $User
                AccessRights = "SendOnBehalf"
            }
        }
    }
}

# Define the user for whom you want to check permissions
$user = "<user@domain.com>"

# Get Distribution Lists (DLs) and Dynamic Distribution Lists (DDLs)
$dlAndDdlPermissions = Get-SendAsPermissions -User $user

# Get Microsoft 365 Groups (Unified Groups)
$m365Groups = Get-UnifiedGroup -ResultSize Unlimited
$m365GroupPermissions = @()
foreach ($group in $m365Groups) {
    $groupPermissions = Get-SendAsPermissions -User $user -Recipient $group.Identity
    if ($groupPermissions) {
        $m365GroupPermissions += $groupPermissions
    }
}

# Get Send on Behalf permissions for DLs, DDLs, and M365 Groups
$sendOnBehalfPermissions = Get-SendOnBehalfPermissions -User $user

# Combine results
$allPermissions = $dlAndDdlPermissions + $m365GroupPermissions + $sendOnBehalfPermissions

# Output results
$allPermissions | Format-Table -Property Identity, Delegate, AccessRights -AutoSize

# Disconnect from Exchange Online
Disconnect-ExchangeOnline -Confirm:$false
```

### Usage
- Replace `<yourUser@domain.com>` and `<user@domain.com>` with the appropriate user email addresses.
- Run the script in a PowerShell session with the necessary permissions to connect to Exchange Online.

Make sure to test this script in a controlled environment to ensure it meets your requirements. If you need any adjustments or further assistance, feel free to ask!
