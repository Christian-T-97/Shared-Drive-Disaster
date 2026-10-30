# Investigation Notes — Shared Drive Disaster

## 1. Initial Setup

Created a Windows 11 lab environment with local accounts for three departments:

- HR
- IT
- Employees

Created corresponding local security groups and assigned users to their intended departmental groups.

Created a company directory structure containing general, confidential, restricted, and shared directories. Added fictional text documents for access-control testing.

## 2. Initial ACL Investigation

Inspected the ACL on the HR confidential directory using PowerShell:

```powershell
(Get-Acl 'C:\Users\Admin1\Company\HR\Confidential').Access |
    Format-Table IdentityReference, FileSystemRights, AccessControlType, IsInherited
```

### Initial Observation

The ACL output included inherited `FullControl` entries for:

- `NT AUTHORITY\SYSTEM`
- `BUILTIN\Administrators`
- The local `Admin1` account

No explicit HR group permission was present at that point.

The inherited entries indicated that permissions were being inherited from a parent directory. Additional testing was required to determine their effect on other users.

## 3. Configuring HR Access

Added an explicit `Modify` permission for the HR security group on the confidential HR directory.

```powershell
$path = "C:\Users\Admin1\Company\HR\Confidential"
$acl = Get-Acl $path

$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "HR",
    "Modify",
    "ContainerInherit,ObjectInherit",
    "None",
    "Allow"
)

$acl.AddAccessRule($rule)
Set-Acl -Path $path -AclObject $acl
```

### Verification

Inspected the ACL again.

The output showed an entry for the local HR group with:

- **Permission:** Modify, Synchronize
- **Access control type:** Allow
- **IsInherited:** False

This confirmed that the explicit HR permission was present on the target directory.

The inherited SYSTEM, Administrators, and Admin1 entries remained in place.

## 4. Account and Sign-In Troubleshooting

Attempted to sign in as an HR user through the Windows login screen. Only the Admin1 account was displayed, and an obvious "Other user" option was unavailable.

Verified that the HR account existed and was enabled:

```powershell
Get-LocalUser -Name "Gervonta" |
    Format-List Name, Enabled, PasswordRequired, PrincipalSource
```

The account was enabled. A password had also been set.

Checked HR group membership:

```powershell
Get-LocalGroupMember -Group "HR"
```

The expected HR usernames appeared in the group membership output.

Tried launching a process under alternate credentials. After entering the credentials, the `whoami` command in the resulting process identified the account as `Gervonta`.

The PowerShell prompt continued to display a working directory under `C:\Users\Admin1`. The prompt path alone does not establish the identity of the process.

## 5. Preliminary HR File Test

While investigating HR access, opened a fictional text file in the confidential HR directory, entered the following text, and saved it:

`Confidential HR test document. Authorized HR access verified.`

The save operation succeeded.

### Interpretation

The file was successfully edited and saved during the investigation. However, the desktop session remained signed in as Admin1, and the file operation was not independently verified under Gervonta's security context.

Therefore, this is a preliminary result, not yet conclusive evidence that the HR ACL correctly grants access to the intended user.

## 6. Next Investigation Steps

1. Confirm the security identity used for the HR file-access test. = VERIFIED
2. Verify that the HR user can read and modify the confidential HR file. = VERIFIED
3. Test the same file using an ordinary Employees account.
4. Record whether access is allowed or denied.
5. Investigate unexpected access, including inherited permissions and group memberships.
6. Apply any necessary remediation and repeat the tests.
7. Capture screenshots of the relevant ACLs and test results.

## Current Status

**Configuration:** An explicit HR `Modify` permission has been added and verified in the ACL output.

**Testing:** A file was edited and saved, but the account context for that operation still requires confirmation.

**Remediation:** Not yet completed. Unauthorized employee access has not yet been tested.

----

### HR Confidential Access Test

**Test account:** Gervonta (member of the HR group)

**Target directory:** `C:\Users\Admin1\Company\HR\Confidential`

**Assigned permission:** Modify

**Test procedure:**
1. Launched a PowerShell session using Gervonta's credentials.
2. Used `whoami` to verify the authenticated identity.
3. Listed the contents of the HR Confidential directory.
4. Used `Add-Content` to append text to an existing file.
5. Used `Get-Content` to verify the updated file contents.

**Result:** Successful. Gervonta was able to access the directory, modify an existing file, and read the updated contents.

**Conclusion:** The HR group's assigned Modify permission is functioning as expected for this test account.
