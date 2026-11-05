# Investigation Notes — Shared Drive Disaster

## 1. Initial Environment Setup

Created a simulated Windows company environment using Windows 11.

**User accounts:**
- Two HR users
- Two IT users
- Two Employees users

**Local security groups:**
- HR
- IT
- Employees

Each user was assigned to their corresponding department group.

**Directory structure:**

```text
Company/
├── HR/
│   ├── General/
│   └── Confidential/
├── IT/
│   ├── General/
│   └── Restricted/
└── Employees/
    └── Shared/
```

Created fictional company documents in the directories to simulate a working business environment.

## 2. Initial Permission Investigation

Inspected the existing NTFS permissions using PowerShell:

```powershell
(Get-Acl $path).Access |
    Format-Table IdentityReference, FileSystemRights, AccessControlType, IsInherited
```

**Initial findings:**

The directories contained three inherited permission entries:

- NT AUTHORITY\SYSTEM — FullControl
- BUILTIN\Administrators — FullControl
- Admin1 — FullControl

No explicit department-group permissions had been configured at this stage.

This established the initial permission state before implementing the access-control policy.

## 3. HR Confidential Permission Configuration

Assigned the HR group Modify permission on:

```text
C:\Users\Admin1\Company\HR\Confidential
```

Used PowerShell to create and apply the permission rule.

**Permission details:**
- Group: HR
- Permission: Modify
- Access type: Allow
- Applies to: This folder, subfolders and files
- Inherited: False

Verified the permission using `Get-Acl`.

The HR group now had an explicit Modify permission while the existing inherited SYSTEM, Administrators and Admin1 entries remained intact.

## 4. HR Authorized Access Test

**Test account:** Gervonta (member of the HR group)

**Target directory:** HR\Confidential

**Assigned permission:** Modify

**Test procedure:**
1. Launched a PowerShell session using Gervonta's credentials.
2. Verified the authenticated identity using `whoami`.
3. Listed the contents of the HR Confidential directory.
4. Used `Add-Content` to append text to an existing file.
5. Used `Get-Content` to verify the updated file contents.

**Result:** Successful. Gervonta was able to access the directory, modify an existing file, and read the updated contents.

**Conclusion:** The HR group's assigned Modify permission was functioning as expected for this test account.

## 5. Unauthorized Employee Access Test — Initial Baseline

**Test account:** Mary (member of the Employees group)

**Target directory:** HR\Confidential

**Expected permission:** No access

**Test procedure:**
1. Launched a PowerShell session using Mary's credentials.
2. Verified the authenticated identity using `whoami`.
3. Attempted to list the contents of the HR Confidential directory using `Get-ChildItem`.

**Result:** Access denied. Windows returned an `UnauthorizedAccessException` when Mary attempted to enumerate the directory.

**Conclusion:** The Employee account could not access HR Confidential under the initial permission configuration.

## 6. Employee Shared Folder Permission Configuration

Inspected the existing NTFS permissions on:

```text
C:\Users\Admin1\Company\Employees\Shared
```

The initial ACL contained inherited FullControl entries for:
- NT AUTHORITY\SYSTEM
- BUILTIN\Administrators
- Admin1

No explicit Employees group permission was present.

Assigned the Employees group Modify permission.

**Permission details:**
- Group: Employees
- Permission: Modify
- Access type: Allow
- Applies to: This folder, subfolders and files
- Inherited: False

Verified the new permission using `Get-Acl`.

## 7. Employee Authorized Access Test

**Test account:** Mary (member of the Employees group)

**Target directory:** Employees\Shared

**Assigned permission:** Modify

**Test procedure:**
1. Launched a PowerShell session using Mary's credentials.
2. Verified the authenticated identity using `whoami`.
3. Listed the contents of the Employees Shared directory.
4. Used `Add-Content` to append text to an existing file.
5. Used `Get-Content` to verify the updated file contents.

**Result:** Successful. Mary was able to access and modify a file in the Employees Shared directory.

**Conclusion:** The Employees group's assigned Modify permission was functioning as expected for this test account.

----

## 8. Unauthorized Employee Access Test — Deliberate Misconfiguration

### Objective
Demonstrate how excessive NTFS permissions can expose confidential company information to unauthorized employees.

### Misconfiguration
An explicit Read permission was intentionally assigned to the Employees group on the `HR\Confidential` directory.

This permission was added using PowerShell and applied to the directory, its subdirectories, and files through inheritance flags.

### Test Account
- User: Mary
- Group: Employees
- Target directory: `HR\Confidential`

### Test Procedure
Authenticated as Mary and executed the following commands:

```powershell
Get-ChildItem "C:\Users\Admin1\Company\HR\Confidential"
```

```powershell
Get-Content "C:\Users\Admin1\Company\HR\Confidential\payroll.txt"
```

### Results
Both commands executed successfully.

Mary was able to enumerate the contents of the confidential HR directory and read `payroll.txt`, despite not being an authorized HR employee.

### Security Finding
The Employees group had excessive permissions on a directory containing confidential HR information.

**Impact:** Unauthorized employees could view sensitive company documents, creating a confidentiality risk.

**Root Cause:** An overly permissive NTFS access control entry (ACE) granted the Employees group Read access to the HR confidential directory.

---

## 9. Remediation — Removal of Excessive Permission

### Objective
Remove the unauthorized Read permission assigned to the Employees group while preserving legitimate HR access and existing inherited permissions.

### Remediation Procedure

Recreated the original `FileSystemAccessRule` object to identify the specific permission that needed to be removed.

```powershell
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "Employees",
    "Read",
    "ContainerInherit,ObjectInherit",
    "None",
    "Allow"
)
```

Retrieved the directory's ACL and removed the matching access rule:

```powershell
$acl = Get-Acl $path
$acl.RemoveAccessRule($rule)
```

The command returned `True`, indicating that a matching rule was removed from the ACL object in memory.

Applied the updated ACL to the directory:

```powershell
Set-Acl -Path $path -AclObject $acl
```

### ACL Verification

Inspected the updated permissions:

```powershell
(Get-Acl $path).Access |
    Format-Table IdentityReference, FileSystemRights, AccessControlType, IsInherited
```

### Results
- The Employees group's explicit Read permission was no longer present.
- The HR group's explicit Modify permission remained.
- Existing inherited permissions for SYSTEM, Administrators, and Admin1 remained intact.

**Conclusion:** The excessive permission was successfully removed without removing the legitimate HR permission.

---

## 10. Final Verification — Authorized and Unauthorized Access

### Objective
Verify that the remediation successfully restored the intended access boundary without disrupting authorized HR users.

### Test 1: Unauthorized Employee Access

**Account:** Mary (Employees)

Authenticated as Mary and attempted to enumerate the HR confidential directory:

```powershell
Get-ChildItem "C:\Users\Admin1\Company\HR\Confidential"
```

**Result:** Access denied.

**Conclusion:** Mary could no longer access the HR confidential directory after the excessive permission was removed.

### Test 2: Authorized HR Access

**Account:** Gervonta (HR)

Authenticated

---

## 11. IT Department Permission Configuration

### Objective
Configure NTFS permissions to grant the IT group Modify access to its designated directories while maintaining separation from other departments.

### Target Directories
- `IT\General`
- `IT\Restricted`

### Permission Configuration

Inspected the existing ACLs on both directories using PowerShell.

Both directories initially contained inherited administrative permissions for:
- SYSTEM — FullControl
- Administrators — FullControl
- Admin1 — FullControl

No explicit department-group permissions were present.

Created and applied an explicit Modify permission for the IT group on each directory using `FileSystemAccessRule`.

The permissions were configured with:
- Identity: IT
- Permission: Modify
- Inheritance: ContainerInherit, ObjectInherit
- Access Control Type: Allow

### ACL Verification

Inspected both updated ACLs and confirmed:
- IT — Modify, Allow, IsInherited: False
- Existing inherited administrative permissions remained intact.

**Conclusion:** The IT group was successfully granted Modify access to both designated directories.

---

## 12. Authorized IT Access Test

### Objective
Verify that an authenticated IT user can access and modify files within both IT directories.

### Test Account
- User: Jeff
- Group: IT

### Test Procedure

Launched a separate PowerShell session using the IT account's credentials.

Verified the account identity using:

```powershell
whoami
```

Tested directory enumeration:

```powershell
Get-ChildItem "C:\Users\Admin1\Company\IT\General"
```

```powershell
Get-ChildItem "C:\Users\Admin1\Company\IT\Restricted"
```

Tested file modification using `Add-Content` and verified the changes using `Get-Content`.

### Results
- Successfully listed the contents of IT\General.
- Successfully listed the contents of IT\Restricted.
- Successfully modified files in both directories.
- Verified the changes to the files.

**Conclusion:** The IT group's Modify permissions were functioning as intended on both directories.

---

## 13. Unauthorized Department Access Tests — IT Directories

### Objective
Verify that users outside the IT department cannot access IT's designated directories.

### Test Accounts
- Mary — Employees group
- Gervonta — HR group

### Test Procedure

Authenticated separately as each user and attempted to enumerate both IT directories using:

```powershell
Get-ChildItem "C:\Users\Admin1\Company\IT\General"
```

```powershell
Get-ChildItem "C:\Users\Admin1\Company\IT\Restricted"
```

### Results

Both Mary and Gervonta received Access Denied when attempting to access either directory.

| User | Group | IT\General | IT\Restricted |
|---|---|---|---|
| Jeff | IT | Access allowed | Access allowed |
| Mary | Employees | Access denied | Access denied |
| Gervonta | HR | Access denied | Access denied |

### Security Finding

The IT directories successfully restricted access to members of the IT group.

No explicit Deny permissions were required. Unauthorized users lacked applicable Allow permissions for these directories.

**Conclusion:** Departmental access separation was successfully verified for both IT directories.

---

## 14. HR General Permission Configuration

The HR General directory initially contained only inherited permissions for SYSTEM, Administrators, and Admin1.

An explicit Modify permission was added for the HR group using PowerShell and the Windows NTFS access control list (ACL).

The permission was configured to apply to the directory, its subdirectories, and files through container and object inheritance.

After applying the ACL with `Set-Acl`, the saved permissions were inspected to confirm that:

- HR had explicit Modify permission.
- SYSTEM, Administrators, and Admin1 retained their inherited Full Control permissions.
- No additional department permissions were present.

## 15. HR General Access Verification

Access was tested using separate department accounts.

**Authorized access — HR**

- Authenticated as Gervonta, an HR user.
- Successfully listed the HR General directory.
- Successfully modified an existing file.
- Verified the file contents after modification.

**Unauthorized access — Employees and IT**

- Authenticated as Mary, an Employees user.
- Attempted to list the HR General directory.
- Access was denied.
- Repeated the test using an IT account.
- Access was denied.

The results confirmed that HR users could modify HR General while Employees and IT users could not access it.

## 16. Employees Shared Permission Configuration

The Employees Shared directory already contained an explicit Modify permission for the Employees group.

To support cross-department access, additional explicit Read permissions were configured for the HR and IT groups.

The existing Employees Modify permission was preserved.

After applying the changes, the saved ACL was inspected to confirm:

- Employees — Modify
- HR — Read
- IT — Read
- SYSTEM, Administrators, and Admin1 — inherited Full Control

The HR and IT permissions were configured to apply to the directory, its subdirectories, and files.

## 17. Employees Shared Access Verification

Access was tested using accounts from all three departments.

**Employees — Mary**

- Successfully listed the directory.
- Successfully read existing files.
- Successfully modified an existing file.

**HR — Gervonta**

- Successfully listed the directory.
- Successfully read existing files.
- Modification attempts were denied.

**IT — IT test account**

- Successfully listed the directory.
- Successfully read existing files.
- Modification attempts were denied.

The results confirmed that Employees retained Modify access while HR and IT had Read-only access.

## 18. Final Company Root Directory Review

The ACL on the Company root directory was inspected to identify permissions inherited or explicitly assigned at the parent level.

The inspected ACL contained:

- SYSTEM — Full Control
- Administrators — Full Control
- Admin1 — Full Control

No explicit HR, IT, or Employees group permissions were listed on the Company root directory.

A path-access test was then performed using Mary, an Employees account.

- Mary successfully accessed Employees Shared through the Company directory path.
- Mary was denied access to HR Confidential through the same path.

This confirmed that the parent directory configuration did not prevent access to an authorized department folder or grant access to an unauthorized department folder.

## Project Verification Summary

All five directories were configured and tested against the intended access policy:

| Directory | HR | IT | Employees |
|---|---|---|---|
| HR General | Modify | Denied | Denied |
| HR Confidential | Modify | Denied | Denied |
| IT General | Denied | Modify | Denied |
| IT Restricted | Denied | Modify | Denied |
| Employees Shared | Read | Read | Modify |

A deliberate unauthorized permission was also introduced in HR Confidential, tested successfully, and subsequently removed. Final verification confirmed that the unauthorized access was no longer possible.

**Project outcome:** The simulated department-level NTFS permission policy was configured, tested, and verified.

## Key Technical Concepts Encountered

- **Group membership:** Determines which security groups a user belongs to.
- **NTFS permissions:** Define what users and groups can do with files and folders.
- **ACL (Access Control List):** Contains the permission entries associated with a file or folder.
- **Inheritance:** Allows permissions to flow from parent folders to child folders and files.
- **Explicit permissions:** Permissions assigned directly to a particular file or folder.
- **Implicit deny:** Access is denied when no applicable Allow permission grants the requested access.
- **Least privilege:** Users should receive only the permissions necessary to perform their jobs.

