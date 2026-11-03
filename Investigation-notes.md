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

---

## Key Technical Concepts Encountered

- **Group membership:** Determines which security groups a user belongs to.
- **NTFS permissions:** Define what users and groups can do with files and folders.
- **ACL (Access Control List):** Contains the permission entries associated with a file or folder.
- **Inheritance:** Allows permissions to flow from parent folders to child folders and files.
- **Explicit permissions:** Permissions assigned directly to a particular file or folder.
- **Implicit deny:** Access is denied when no applicable Allow permission grants the requested access.
- **Least privilege:** Users should receive only the permissions necessary to perform their jobs.

